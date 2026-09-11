---
title: "Making of SARE: Master Seeds in Hybrid Post-Quantum Encryption"
author: Zola Gonano
layout: post
series: Making of SARE
---

> This post is written by a human being (me).

A few years ago, I started working on a project called the Sare Project. Sare stands for Safe At Rest Encryption. I wanted to make a hybrid post-quantum encryption library and tool to be a post-quantum replacement for GPG.

I have been away from this project for almost a year, and when I opened the codebase I didn’t know what I had done. So I thought as I read my own old code, I could explain why I did certain things and what ideas led me down those paths.

The first issue was that I wanted something hybrid, meaning it would be the best of both worlds. If the post-quantum algorithms failed, we’d still have our normal algorithms like elliptic curves.

The issue was that I needed one post-quantum KEM (Key Encapsulation Mechanism) for encapsulation. Encapsulation in cryptography is a method to **securely establish a shared secret key** between two parties over an insecure channel, using **asymmetric (public-key) cryptography**.

The way that these KEMs work is that they generate a public key and a secret key. Then, using the recipient’s public key, a **ciphertext** (called an "encapsulation") and a **shared secret** are produced. On decapsulation, the recipient uses their secret key to recover the shared secret from the ciphertext.

The idea I had was to combine a post-quantum KEM with an elliptic curve DH (Diffie-Hellman) key exchange. The way that DH key exchanges work is that two parties (Alice and Bob for example) each generate a private/public key pair and exchange public keys to compute a shared secret using properties of modular arithmetic.

The difference with KEMs is that instead of exchanging keys via exponentiation, the sender **encapsulates** a secret into a ciphertext using the recipient’s public key. The recipient **decapsulates** using their private key to recover the secret.

```rust
// /sare-core/src/hybrid_kem/mod.rs

pub struct DHKeyPair {
    pub public_key: Vec<u8>,
    pub secret_key: SecretVec<u8>,
    pub algorithm: DHAlgorithm,
}

pub struct KEMKeyPair {
    pub public_key: Vec<u8>,
    pub secret_key: SecretVec<u8>,
    pub algorithm: KEMAlgorithm,
}

pub struct HybridKEM {
    pub dh_keypair: DHKeyPair,
    pub kem_keypair: KEMKeyPair,
}

impl HybridKEM {
    pub fn new(dh_keypair: DHKeyPair, kem_keypair: KEMKeyPair) -> Self {
        HybridKEM {
            dh_keypair,
            kem_keypair,
        }
    }

    pub fn calculate_raw_shared_key(
        &self,
        kem_cipher_text: &[u8],
        dh_sender_public_key: &[u8],
    ) -> Result<(SecretVec<u8>, SecretVec<u8>), HybridKEMError> {
        let binding = dh_sender_public_key.to_vec();
        let diffie_hellman = DiffieHellman::new(&self.dh_keypair, &binding);
        let kem_decapsulation =
            Decapsulation::new(&self.kem_keypair.secret_key, &self.kem_keypair.algorithm);

        let dh_shared_secret = diffie_hellman.calculate_shared_key()?;
        let kem_shared_secret = kem_decapsulation.decapsulate(kem_cipher_text)?;

        Ok((dh_shared_secret, kem_shared_secret.shared_secret))
    }
}
```

This meant that only for exchanging keys, I’d need two secret keys to be generated.

And that wasn’t it. Another issue was that these exchange algorithms couldn’t be used to sign messages. So once again I had to generate two different keys: one for post-quantum signing and one for signing with elliptic curves. I would include both signatures in the signed outputs so the recipient could verify both: once with the post-quantum algorithm and once with the classical elliptic curve algorithm.

```rust
// /sare/blob/main/sare-core/src/hybrid_sign/mod.rs

pub struct ECKeyPair {
    pub public_key: Vec<u8>,
    pub secret_key: SecretVec<u8>,
    pub algorithm: ECAlgorithm,
}

pub struct ECSignature<'a> {
    pub keypair: &'a ECKeyPair,
}

pub struct PQKeyPair {
    pub public_key: Vec<u8>,
    pub secret_key: SecretVec<u8>,
    pub algorithm: PQAlgorithm,
}

pub struct PQSignature<'a> {
    pub keypair: &'a PQKeyPair,
}
```

I wanted the system to be modular, so we could have multiple options for each component’s algorithm. It wouldn't be a fixed algorithm, and it could be expanded as new algorithms become available.

For example, in `HybridKEM`:

```rust
// /sare/blob/main/sare-core/src/hybrid_kem/mod.rs

pub enum DHAlgorithm {
    X25519,
}

pub enum KEMAlgorithm {
    Kyber768,
}
```

Now the issue was that I had to store four different private keys and four different public keys. What I did for public keys relates to how I decided to format keys, which I will explain in the next article about the making of SARE.

For private keys, I decided to use what most crypto wallets use: a master seed from which all other keys are derived. This way, the user only needs to store a single key, and all private keys are derived from that. The master seed was defined as follows:

```rust
// /sare/blob/main/sare-core/src/seed/mod.rs

pub struct Seed {
    raw_seed: SecretVec<u8>,
}
```

I went with a 128-byte seed, just in case aliens come to Earth with their supercomputers. Even they shouldn’t have the power to brute-force it.

For comparison, to brute-force AES-256 you need to guess
`115,792,089,237,316,195,423,570,985,008,687,907,853,269,984,665,640,564,039,457,584,007,913,129,639,936` possibilities.

To brute-force SARE’s 128-byte master seed, you’d need to guess
`179,769,313,486,231,590,772,930,519,078,902,473,361,797,697,894,230,657,273,430,081,157,732,675,805,500,963,132,708,477,322,407,536,021,120,113,879,871,393,357,658,789,768,814,416,622,492,847,430,639,474,124,377,767,893,424,865,485,276,302,219,601,246,094,119,453,082,952,085,005,768,838,150,682,342,462,881,473,913,110,540,827,237,163,350,510,684,586,298,239,947,245,938,479,716,304,835,356,329,624,224,137,216` possibilities. That’s `2^768` times more than AES-256.

It might seem like a large master seed to remember, but when using Bitcoin’s mnemonic phrases, this turns into 96 words, which is reasonable. You can even write it down on paper if you want.

```rust
// /sare/blob/main/sare-core/src/seed/mod.rs

impl Seed {
    pub fn new(raw_seed: SecretVec<u8>) -> Self {
        Seed { raw_seed }
    }

    pub fn generate() -> Self {
        let mut raw_seed_buffer = vec![0u8; 128];

        OsRng.fill_bytes(&mut raw_seed_buffer);

        Seed {
            raw_seed: SecretVec::from(raw_seed_buffer),
        }
    }

    pub fn to_mnemonic(&self) -> SecretString {
        let seed_chunks: Vec<&[u8]> = self.raw_seed.expose_secret().chunks_exact(32).collect();

        let mut mnemonic_phrase: String = String::new();

        for chunk in seed_chunks {
            let mnemonic = Mnemonic::from_entropy(chunk, Language::English).unwrap();
            mnemonic_phrase.push_str(mnemonic.phrase());
            mnemonic_phrase.push(' ');
        }

        SecretString::from(mnemonic_phrase.trim_end().to_string())
    }
}
```

Using this solution, users don’t store multiple keys. They store a single master seed, from which all of their keys are derived. If the algorithms change, the master seed stays the same, providing built-in backward compatibility.

To store this master seed, users have two options:

* Store it symmetrically encrypted using AES-KW with a passphrase (which is recommended)
* Store it unencrypted (not recommended)

```rust
pub struct KeyWrap {
    input_key: SecretVec<u8>,
}

impl KeyWrap {
    pub fn new(input_key: SecretVec<u8>) -> Result<Self, EncryptionError> {
        if input_key.expose_secret().len() != 32 {
            return Err(EncryptionError::InvalidKeyLength);
        }

        Ok(KeyWrap { input_key })
    }

    pub fn wrap(&self, data: &SecretVec<u8>) -> Result<Vec<u8>, EncryptionError> {
        let mut output: Vec<u8> = vec![0; data.expose_secret().len() + 8];

        let input_key = <[u8; 32]>::try_from(self.input_key.expose_secret().as_slice()).unwrap();

        let kek = KekAes256::from(input_key);

        kek.wrap_with_padding(data.expose_secret(), &mut output)?;

        Ok(output)
    }
}
```

To derive unique keys for each algorithm from the master seed, I used magic bytes. Each algorithm gets a different identifier:

```rust
const ED25519_MAGIC_BYTES: [u8; 4] = [25, 85, 210, 14]; // 0xED25519 in little endian
const DILITHIUM3_MAGIC_BYTES: [u8; 4] = [211, 12, 0, 0]; // 0xCD3 in little endian

const X25519_MAGIC_BYTES: [u8; 4] = [25, 85, 2, 0];      // 0x25519 in little endian
const KYBER768_MAGIC_BYTES: [u8; 4] = [104, 7, 0, 0];    // 0x768 in little endian
```

After that, the derivation process was fairly simple. I used HKDF-SHA256 for keys that needed to be 32 bytes and HKDF-SHA512 for keys that needed to be 64 bytes.

```rust
// /sare/blob/main/sare-core/src/kdf/mod.rs

impl<'a> HKDF<'a> {
    pub fn new(input_data: &'a SecretVec<u8>, salt: &'a [u8], algorithm: HKDFAlgorithm) -> Self {
        HKDF {
            input_data,
            salt,
            algorithm,
        }
    }

    fn expand_sha256(
        &self,
        additional_context: Option<&[u8]>,
        okm: &mut [u8],
    ) -> Result<(), KDFError> {
        let hkdf = Hkdf::<Sha256>::new(Some(self.salt), self.input_data.expose_secret());
        hkdf.expand(additional_context.unwrap_or(&[0]), okm)?;

        Ok(())
    }

    fn expand_sha512(
        &self,
        additional_context: Option<&[u8]>,
        okm: &mut [u8],
    ) -> Result<(), KDFError> {
        let hkdf = Hkdf::<Sha512>::new(Some(self.salt), self.input_data.expose_secret());
        hkdf.expand(additional_context.unwrap_or(&[0]), okm)?;

        Ok(())
    }
}
```

---

In the next article in this series, I’ll try to explain more of it. A lot of things are too basic to explain in a blog post, but if I find anything interesting I’ll try to explain the reasoning behind it and how I implemented it.

SARE is available at: [https://sareproject.github.io](https://sareproject.github.io) and [https://github.com/SareProject](https://github.com/SareProject)
Giving the project a star would be good motivation for me to get back to this old project and hopefully make something usable out of it. Right now, it’s just a library.

