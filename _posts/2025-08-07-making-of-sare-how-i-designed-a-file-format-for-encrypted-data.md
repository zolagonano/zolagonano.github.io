---
title: "Making of SARE: How I Designed a File Format for Encrypted Data"
author: Zola Gonano
series: Making of SARE
layout: post
---


**Disclaimer**: This might sound like a stupid idea, but believe me, I had good reasons for it.

In the previous post on the making of **SARE**, I went over how and why I integrated master seeds into the architecture. That laid the groundwork for consistent key derivation across different cryptographic operations. If you haven’t read that post, you can find it [here](/blog/posts/making-of-sare-master-seeds-in-hybrid-post-quantum-encryption) - that’ll help this one make more sense.

At this point, I had all the core building blocks in place.

- I had **HybridKEM** working to exchange shared secrets between parties. 

- I had **HybridSign** to sign data and prove its authenticity.

- And I had encryption ready using AEAD algorithms.
    

But I still needed a way to store the encrypted data. Not just dump ciphertext into a file, but structure it in a way that made sense for machines, future-proofing, and minimal complexity. So this post is about how I designed the binary format that sits at the start of every SARE-encrypted file, and how it helps keep everything structured, verifiable, and extensible.

## Why Bother With a Custom Format

You might ask, "Why not just use an existing container format?" Like PGP, PKCS #7, or even ZIP with metadata extensions?

And yeah, I could have done that. But those formats bring a ton of complexity, assumptions, and legacy cruft I didn’t need. My use case was specific: I wanted a simple, self-contained format that supports hybrid crypto, optional signatures, forward compatibility, and human-readable metadata. Something compact, easy to parse, and still flexible enough to evolve.

So I built my own.

## The Header Layout

Here's the layout I came up with for the file header:

- First 9 bytes are the string `"SARECRYPT"` - this acts as the magic bytes, so we can verify that the file is using the right format.
    
- Next, 8 bytes (little-endian) indicating the total length of the header section. This tells the parser how much of the file to read before hitting the encrypted payload.
    
- Then 4 bytes for the file format version. This allows backward compatibility.
    
- After that, 8 bytes (little-endian) indicating the length of the metadata section.
    
- And finally, another 8 bytes (little-endian) giving the length of the signature section.
    

That’s the fixed structure. What comes next are two variable-length sections:

1. Metadata - encoded as BSON.
    
2. Signature - also encoded as BSON. May be empty if not used.
    

By explicitly length-prefixing each section, the format becomes fully self-framed. No guesswork, no delimiters, no relying on external schemas.

## Why BSON

You might wonder why I went with **BSON** instead of something more efficient like CBOR or even plain binary structs.

There were a few practical reasons:

- Good **Serde support** in Rust. BSON integrates cleanly and lets me serialize complex structs with minimal boilerplate.
    
- **Extensibility**. BSON supports optional fields and nested structures, so the format can grow without breaking old readers.
    
- **Self-describing**. Each field includes a key, which helps debugging and forward compatibility.
    
- **Readable with tooling**. BSON can be dumped and inspected if needed. Helps during development and debugging.
    

I didn't want to define and maintain a custom binary schema. Encoding Rust structs to BSON with Serde got me everything I needed.

## What Goes Into Metadata

The metadata holds all the crypto parameters needed to decrypt and verify the file. Things like:

- Which encryption algorithm was used (e.g., `AES256GCM`)
    
- What key derivation function (`scrypt`, `argon2`, etc.) and its parameters
    
- Salt
    
- Nonce
    
- Optional comment
    
- Optional KEM and signature metadata
    

All of this is captured in this struct:

```rust
#[derive(Serialize, Deserialize)]
pub struct HeaderMetadataFormat {
    #[serde(skip_serializing_if = "Option::is_none", flatten)]
    kem_metadata: Option<KEMMetadataFormat>,

    #[serde(skip_serializing_if = "Option::is_none", flatten)]
    signature_metadata: Option<SignatureMetadataFormat>,

    #[serde(flatten)]
    encryption_metadata: EncryptionMetadataFormat,

    #[serde(skip_serializing_if = "Option::is_none")]
    comment: Option<String>,
}
```

Everything is optional where it makes sense. If there's no signature, it’s omitted. If there's no KEM data, same thing. And because BSON supports this kind of structure naturally, the result stays clean.

This makes the format flexible enough to support both password-based encryption and hybrid public key schemes, depending on the use case.

## How Encoding Works

Header encoding is pretty mechanical. Build a buffer in the right order, write out the fixed parts first, and then append the variable sections.

```rust
pub fn encode(&self) -> Vec<u8> {
    let mut header: Vec<u8> = Vec::new();
    header.extend(MAGIC_BYTES); // "SARECRYPT"

    let mut header_buffer: Vec<u8> = Vec::new();

    header_buffer.extend(&self.version.to_le_bytes());

    let metadata_bson = self.metadata.encode();
    header_buffer.extend(&(metadata_bson.len() as u64).to_le_bytes());
    header_buffer.extend(metadata_bson);

    if let Some(signature) = &self.signature {
        let signature_bson = signature.encode_bson();
        header_buffer.extend(&(signature_bson.len() as u64).to_le_bytes());
        header_buffer.extend(signature_bson);
    } else {
        header_buffer.extend(&0u64.to_le_bytes());
    }

    header.extend(&(header_buffer.len() as u64).to_le_bytes());
    header.extend(header_buffer);

    header
}
```

The result is a binary blob with this structure:

```
MAGIC_BYTES || HEADER_LEN || VERSION || METADATA_LEN || METADATA || SIGNATURE_LEN || SIGNATURE?
```

No complexity, no dependencies, and everything is length-prefixed. I like simplicity.

## How Decoding Works

Decoding is just the reverse of encoding. You use a cursor to step through the byte stream, read the lengths, and slice each section accordingly.

```rust
pub fn decode(header: &[u8]) -> Result<Self, FormatError> {
    let mut cursor = 0;

    if !Self::verify_magic_bytes(header, &mut cursor)? {
        return Err(FormatError::FailedToDecode(ErrSection::HEADER));
    }

    let header_length = Self::read_u64(header, &mut cursor)?;
    let version = Self::read_u32(header, &mut cursor)?;

    let metadata_length = Self::read_u64(header, &mut cursor)?;
    let metadata_bson = &header[cursor..cursor + metadata_length as usize];
    let metadata = HeaderMetadataFormat::decode(metadata_bson)?;
    cursor += metadata_length as usize;

    let signature_length = Self::read_u64(header, &mut cursor)?;
    let signature = if signature_length > 0 {
        let signature_bson = &header[cursor..cursor + signature_length as usize];
        cursor += signature_length as usize;
        Some(SignatureFormat::decode_bson(signature_bson)?)
    } else {
        None
    };

    Ok(HeaderFormat {
        version,
        metadata,
        signature,
    })
}
```

Each failure returns a typed `FormatError`, so issues can be logged or surfaced in a user-friendly way. Without those ugly panics.

## Testing and Roundtrip Validation

To make sure encoding and decoding are consistent, I wrote tests that decode a sample header, re-encode it, and compare the result to a fixed base64 snapshot.

```rust
#[test]
fn header_format_decode() {
    let decoded_header =
        HeaderFormat::decode(&BASE64_STANDARD.decode(ENCODED_HEADER).unwrap()).unwrap();

    assert_eq!(expected_header.encode(), decoded_header.encode());
}
```

This helps catch regressions and ensures the binary layout stays deterministic and compatible. I don’t care about minor reordering of fields in BSON, but the full re-encoded header needs to match.

After that comes the encrypted body - usually raw ciphertext. The metadata tells you how to decrypt it, what keys to derive, what nonce to use, etc.

Because the header is self-contained, the decryption pipeline can be fully stateless. You load the header, parse it, and then stream the rest.

---

In the next post, I’ll break down how the actual encryption pipeline works and how all of this ties together with key management.

SARE is available at: [https://sareproject.github.io](https://sareproject.github.io) and [https://github.com/SareProject](https://github.com/SareProject) Giving the project a star would be good motivation for me to get back to this old project and hopefully make something usable out of it.
