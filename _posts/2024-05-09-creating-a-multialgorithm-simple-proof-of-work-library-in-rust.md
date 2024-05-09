---
title: "Creating a Muti-Algorithm simple Proof-Of-Work library in Rust"
layout: post
author: Zola Gonano
---



I had an idea for a project that required Proof of Work as a part of it, but I couldn't find any Rust libraries that would have Argon2id or Scrypt algorithms and were meant to provide proof of work functionality without being tied to a specific blockchain. So, I decided to develop my own. When I was doing so, I wanted to show how Rust's data types can make such things easy and clean.

Okay, but what is PoW? Proof of Work is like solving a puzzle – an expensive task for a computer. It's proof that your computer has done something difficult. It's mostly used in cryptocurrency blockchains during mining, proving that the miner has completed the puzzle (equivalent to going to a mine and digging with a pickaxe for gold and silver). However, it's not necessarily meant only for mining. Another common example of its use is for preventing spam on a network, which was the reason I needed to use it. For example, Bitmessage, which was an old (now defunct) P2P email network, used it to prevent spam on the network.

The idea is pretty straightforward. I wanted a library that is expandable, meaning multiple algorithms can be added to it without breaking everything. Secondly, I needed it to be general-purpose. I didn't want it to be meant for one specific blockchain; I wanted it to handle any data and verify any data using it.

To start, I created a `PoWAlgorithm` enum, which would be expanded to include algorithms that the library is going to support. For now, I've used SHA2_256 for testing.

```rust
pub enum PoWAlgorithm {
    Sha2_256,
}
```

Next, I implemented calculate functions for it, which compute the hash of given data:

```rust
impl PoWAlgorithm {
    pub fn calculate_sha2_256(data: &[u8], nonce: usize) -> Vec<u8> {
        let mut hasher = Sha256::new();
        hasher.update(data);

        hasher.update(nonce.to_le_bytes());

        let final_hash = hasher.finalize();

        final_hash.to_vec()
    }

    pub fn calculate(&self, data: &[u8], nonce: usize) -> Vec<u8> {
        match self {
            Self::Sha2_256 => Self::calculate_sha2_256(data, nonce),
        }
    }
}
```

This kind of structure also allows using a trait for algorithms later, reducing some of the code and making the project cleaner as it grows.

Then I created the `PoW` library, meant to calculate and verify the proof of work based on the given data, difficulty, and the algorithm.

```rust
pub struct PoW {
    data: Vec<u8>,
    difficulty: usize,
    algorithm: PoWAlgorithm,
}
```

For the data, I decided to accept all types that implement serde's Serialize trait and convert the data to JSON. This way, anyone could provide their own structs and data types to the PoW.

```rust
impl PoW {
    pub fn new(
        data: impl Serialize,
        difficulty: usize,
        algorithm: PoWAlgorithm,
    ) -> Result<Self, String> {
        Ok(PoW {
            data: serde_json::to_vec(&data).unwrap(),
            difficulty,
            algorithm,
        })
    }

    pub fn calculate_pow(&self, target: &[u8]) -> (Vec<u8>, usize) {
        let mut nonce = 0;

        loop {
            let hash = self.algorithm.calculate(&self.data, nonce);

            if &hash[..target.len()] == target {
                return (hash, nonce);
            }
            nonce += 1;
        }
    }

    pub fn verify_pow(&self, target: &[u8], pow_result: (Vec<u8>, usize)) -> bool {
        let (hash, nonce) = pow_result;

        let calculated_hash = self.algorithm.calculate(&self.data, nonce);

        if &calculated_hash[..target.len()] == target && calculated_hash == hash {
            return true;
        }
        false
    }
}
```

For the actual proof of work calculation, I followed what most cryptocurrencies do. I defined target bytes, which are the bytes that the hash should match with at the beginning. The nonce is then incremented until the hash meets this target.

---

My blog and all of its content are available under the `CC by SA 4.0` License on my [GitHub](https://github.com/zolagonano/zolagonano.github.io). If you notice any problems or have any improvements for the blog or its content, you're always welcome to open a pull request.