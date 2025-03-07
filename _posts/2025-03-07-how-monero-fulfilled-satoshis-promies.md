---
title: "How Monero Fulfilled Satoshi's Promise"
layout: post
series: Crypto
has_math: true
---

Since Trump's election and even before that, news kept coming out about how crypto was going to change the world, how the US was going to become the crypto capital of the world, and how they wanted to build strategic reserves of Bitcoin, Ethereum, Solana, Ripple, and others. At the same time, however, they de-listed Monero from major exchanges like Binance and even criminalized this cryptocurrency.

But if we take a closer look at Monero, we can see that it was the true crypto—it was what Satoshi wanted to build.

## But why is it hated so much by the authorities?

Governments first thought Bitcoin and early cryptocurrencies were anonymous too, as we did. But soon it became obvious that this anonymity shatters as soon as you spend your money in the real world.

Imagine you get some Bitcoin somehow, and you try to buy a car with it. Everyone can trace that transaction, get to the person who sold you the car, and identify you. With the help of AI, this can be further simplified and made easier to track.

But Monero is different. Monero is untraceable, meaning they don’t know who spends money. They cannot collect taxes, and they hate it when they cannot figure out what everyone in society is doing.

But despite all the efforts to criminalize Monero and delist it from exchanges, it has kept its value, continuing to increase over time because, like cash, it is actually used in transactions.

## What is cash and what was Satoshi's promise?

Satoshi’s whitepaper introduced Bitcoin as **"Bitcoin: A Peer-to-Peer Electronic Cash System"**, but Bitcoin never became electronic cash. Not even Bitcoin Cash managed to be electronic cash.

But what is cash? Cash is a form of money that allows for direct **peer-to-peer transactions** without intermediaries. When you pay for something with cash, you don’t know where the cash originally came from. You just know you got it, and you will give it to someone else in exchange. No matter how many times these green papers circulate among people, they won't be linked to anyone. When you hand over cash, the transaction is **irreversible** and requires no third-party verification.

Cash is **Permissionless**, meaning no approval is needed to use it. Bitcoin and other public ledger cryptos are that to some extent.

Cash is **Censorship-Resistant**, meaning it cannot be blocked by banks or governments. Bitcoin is that as well to some extent, but there can be censorship when exchanging on centralized exchanges (CEX).

Cash is **Untraceable**, meaning transactions are not publicly recorded. Bitcoin and most cryptos are not that; they're exactly the opposite.

Cash has **No Counterparty Risk**, meaning unlike digital bank balances, which depend on a central authority, cash does not. Neither Bitcoin nor most cryptos do.

Self-Custodial: When you have cash, it’s in your wallet. You’re holding it. Bitcoin and other cryptos are like that as well.

Based on all these points, Bitcoin and other public ledger cryptocurrencies have most of the characteristics of cash except one thing: **Untraceability**.

## Monero found the missing piece of the puzzle

Monero introduced a new way that fixed the only missing piece of the "Electronic Cash" system that Satoshi wanted to build. Monero made a **untraceable cryptocurrency** while keeping all of cash's characteristics.

To achieve this, Monero introduced a few new technologies to the crypto space:

### **1. Ring Signatures: Sender Anonymity**

Monero transactions do not directly reveal the sender. Instead, they use **ring signatures**, which mix a real transaction input with decoys.

A **ring signature** allows a signer to create a signature that proves ownership of a private key **without revealing which key in the set was used**.

In a ring signature, the user is given a **set of public keys** $P1, P2, …, Pn$. The signer has the private key $sk$ corresponding to one of these.

The signature is constructed so that an observer cannot determine which private key was used, only that one of the keys in the ring signed the message.

Monero used to use **Linkable Spontaneous Anonymous Group (LSAG) Signatures** for ring signatures, in which:

Each **participant's public key** contributes to a **key image** $I$, which is derived as: I=xH(P)I=xH(P) where $x$ is the private key, and $H(P)$ is a cryptographic hash of the public key.

Then, the signer constructs a ring $(P1, P2, …, Pn)$ such that: $σ=(I,c1,r1,…,rn)$

where $c1$ is a random challenge, and $r_i$ are random values that make the signature verifiable without revealing the real signer.

A verifier can check that the **key image $I$** hasn’t been used before, preventing double spending.

In 2020, Monero introduced **[Concise Linkable Spontaneous Anonymous Group Signatures (CLSAG)](https://eprint.iacr.org/2019/654.pdf)**, which does this much more efficiently.

> With CLSAG, users see a 20% improvement in signature verification, and at least a 10% overall improvement for typical transactions. For example, a typical Monero transaction (2 inputs and 2 outputs) that usually weighs 2.5kB now takes only 1.9kB of blockchain space with CLSAG, a ~25% improvement.

**2. RingCT: Amount Privacy**

Bitcoin transactions openly show input and output amounts. Monero hides these with **RingCT** (Ring Confidential Transactions).

RingCT is based on **Pedersen Commitments**, which allow a sender to commit to a value without revealing it.

A commitment to a value $v$ is:

$C=vG+rH$

where:

- $G, H$ are generator points on an elliptic curve.
- $r$ is a random blinding factor.

**Each input and output amount is committed** using Pedersen Commitments. **Bulletproofs** are used to prove that the committed values **sum to zero** (to ensure no coins are created out of thin air) without revealing them. **Decoy amounts** are included to make it impossible to determine the real transaction amounts.

Additionally, a Bulletproofs+ upgrade in 2022 **reduced transaction size** by 5-7x compared to original Bulletproofs, making Monero even faster.

Later on, cryptos like ZCash came along that use Zero-Knowledge Proofs (ZKPs) like ZK-Snarks to do this exact thing, and they are superior. But the problem with ZCash is that it is not the default; in Monero, all transactions are private, whereas in ZCash, some transactions can be private.

In ZKPs, someone can prove that they possess information (in the case of crypto, the amount of money they want to transfer) without revealing anything about the information itself.

### **3. Stealth Addresses: Receiver Privacy**

Bitcoin addresses are static, meaning you can see all transactions going to a given address. Monero uses **one-time stealth addresses** that unlink the receiver from the transaction.

Each Monero wallet has a **public view key $v$ and a public spend key $s$**. When sending a transaction, the sender generates a **one-time public key $P$** as follows:

$P=H(rV)G+S$

where:

- $r$ is a random scalar chosen by the sender.
- $V$ is the recipient’s **public view key**.
- $S$ is the recipient’s **public spend key**.
- $G$ is the elliptic curve generator.

The recipient can compute the corresponding **private key** to spend the funds: 

$P′=H(rv)G+s$

This ensures that **only the recipient can identify and spend the transaction**, but an outsider cannot link it to a wallet.

### **4. Dandelion++: Network-Level Privacy**

Even if transactions are private on-chain, Bitcoin and other cryptocurrencies **leak metadata** through the P2P network when broadcasting transactions. Monero mitigates this with **Dandelion++**, which obfuscates transaction propagation.

In Dandelion++, a node first **selects a "stem" path** (a small subset of nodes) to pass the transaction privately. Then, at a random step, the transaction **"fluffs"** (broadcasts) to the entire network. This makes it difficult to determine the **original source** of the transaction.

## Monero makes a more decentralized cryptocurrency than Bitcoin

As mining becomes more competitive, larger entities with access to specialized hardware (ASICs) and cheap electricity can dominate the mining process. This leads to centralization, where a few large mining pools control the majority of the network's hash power, potentially threatening decentralization and security.

Monero introduces a new PoW algorithm called RandomX to be ASIC-resistant by making the PoW memory-hard, meaning it requires RAM to compute hashes, which cannot be specialized like SHA hashes. RAM is expensive to obtain.

Also, unlike Bitcoin’s **1 MB limit**, Monero has an **adaptive block size** to prevent congestion. And unlike Ethereum and Solana, Monero has **no pre-mined coins, no corporate backing, and no central leadership**.

---

With all that being said, we all love Bitcoin, Ethereum, and Solana. They're great technologies, and even the worst of them is still superior to the best of banking systems.
