---
title: "Haraka Hash: A Look at AES-based PoW for Blockchains"
author: Zola Gonano
layout: post
---

Haraka Hash is really interesting to me, it's not a typical checksum hash function like SHA, or Blake, nor it is a KDF like Argon2 or scrypt. Instead, Haraka is designed as an **AES-based permutation**, optimized for short inputs and extremely high throughput on modern CPUs with AES-NI instructions. 

But why? Why would anyone use this? Well, for crypto…

## Why Use Haraka in Crypto?

Haraka's core idea is to leverage the hardware acceleration built into almost every modern processor to perform cryptographic mixing much faster than traditional sponge-based constructions. This makes it particularly attractive for **blockchain applications**, where hash functions aren’t just about integrity of the hash function, rather it's about fairness and security of the network (against 51% attacks for example).

In Proof-of-Work (PoW) systems, miners repeatedly compute hashes to find a value below a certain target. Traditional hashes like SHA-256 or Blake2, while very secure cryptographically, don’t exploit modern CPU features fully, which means high-end GPUs or ASICs dominate mining, reducing the fairness of incentives in the network and reduce overall security and stability of the network.

Haraka flips this: by using **AES instructions**, CPUs can achieve high throughput cheaply and efficiently, allowing ordinary users with commodity hardware to compete on a more level playing field. This CPU-friendliness is a **decentralization lever**, reducing the concentration of mining power in the hands of specialized operators (like ASIC farms in case of bitcoin).

### Security Benefits

Because Haraka favors CPUs, mining success depends less on specialized hardware and more on **wide participation**. This helps prevent centralization, which is a common vulnerability in PoW networks.

By leveling the computational playing field, Haraka makes it more costly and difficult for attackers to dominate the network. More participants mean greater **network resilience**, reducing the likelihood of reorgs or double-spend attacks.

## Can It Help Monero?

The 51% attack risk on Monero, which was [recently highlighted by the Qubic](/blog/posts/protect-the-freedom-money) mining pool, shows how a single actor controlling the majority of hash power can threaten network integrity, by reversing transactions, censoring others, or reorganizing recent blocks. 

Switching Monero’s Proof-of-Work to **Haraka Hash** could improve the network in several ways:

1. **CPU-Level Optimization:** Haraka uses AES-NI instructions for extremely fast hashing on ordinary CPUs. This makes it easier for smaller miners to compete, flattening the hashrate distribution and raising the cost of a 51% attack.
    
2. **Decentralization:** More accessible mining encourages broader participation, distributing control across many miners instead of allowing dominance by one pool or a coordinated fleet.
    
3. **Energy Efficiency:** Haraka reduces CPU cycles per hash, lowering electricity costs for small miners and further promoting a diverse mining ecosystem.
    
4. **Using with Hybrid Consensus:** In hybrid PoW/PoS systems, Haraka’s efficiency allows frequent block attempts without overloading hardware, making attacks that rely solely on hash power far less feasible. (Like what VerusCoin does for it's consensus, [read more](/blog/posts/veruscoin-everything-that-ethereum_isnt))
    

In short, Haraka wouldn’t completely eliminate the possibility of a 51% attack, but it would **raise the barrier significantly**, making sudden reorganizations or coordinated attacks much harder and costlier.
