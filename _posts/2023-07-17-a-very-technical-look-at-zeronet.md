---
title: A Very Technical Look at ZeroNet
layout: post
series: zeronet
---



[ZeroNet](https://en.wikipedia.org/wiki/ZeroNet) has always been a project that I'm very passionate about, and I enjoy contributing to it. It is a Peer-to-Peer Web-Like Network that cannot be censored or taken down, thanks to its decentralized nature. When I first started exploring ZeroNet, I struggled to find comprehensive documents or blog posts that provided a clear understanding of the network. Therefore, I decided to write this blog post to make it easier for newcomers to learn more about the network and contribute to it.

# Introduction

[ZeroNet](https://github.com/HelloZeroNet/ZeroNet) was created in 2015 by Tamas Kocsis as a decentralized and peer-to-peer network. It utilizes technologies such as Bitcoin's cryptography, [BitTorrent](https://en.wikipedia.org/wiki/BitTorrent) trackers, and [NameCoin](https://en.wikipedia.org/wiki/Namecoin)'s domain name system. These technologies enable ZeroNet to function as a fully dynamic web-like network, in contrast to [IPFS](https://ipfs.tech) which is limited to serving static files and websites. One of the key strengths of ZeroNet is its high resistance to censorship, as it operates without relying on a single central server. It also provides anonymity for users by enabling them to use the [TOR](https://torproject.org) network and hide their IP address while using ZeroNet.
In ZeroNet, every site has an address that is represented by a Bitcoin public key. Peers for the sites are discovered through various methods, including the use of BitTorrent trackers. However, it's worth noting that this is the only instance where a central server comes into play. To eliminate the reliance on a central server, ZeroNet provides an internal plugin called BootStrapper, which allows peers to function as trackers.

# How exactly do we find other peers?

As mentioned before the main method for finding the peers that are hosting the site on the ZeroNet is BitTorrent Trackers, BitTorrent trackers are an essential part of the BitTorrent protocol as they hold the information that what peer holds what piece of data, and ZeroNet uses them for the same purpose. When your ZeroNet client tries to access a site it asks the trackers for the peers that are hosting the website and when your client gets the peers list it starts downloading the site's content and it becomes a peer itself and the next time someone comes to that site they might download a piece of it from you.

![How BitTorrent Trackers work illustration](/assets/pics/bittorrent_tracker.png)

But BitTorrent trackers are not the only option for discovering peers of a site. ZeroNet offers a plugin called `AnnounceZero` that utilizes the ZeroNet Protocol to find peers. This plugin is particularly useful for locating peers on the TOR network, as BitTorrent trackers are not capable of performing this function.

The BitTorrent Tracker works through a plugin named `AnnounceBitTorrent` which operates as follows:
It collects all the trackers that start with `http://`, `https://`, and `udp://`, and sends them a request with the following parameters:

```python
{
  "info_hash": "SHA1_HASH_OF_SITE_ADDRESS",
  "peer_id": "YOUR_PEER_ID",
  "port": "YOUR_FILESERVER_PORT",
  "uploaded": 0,
  "downloaded": 0,
  "left": 431102370,
  "compact": 1,
  "numwant": NUMBER_OF_PEERS_NEEDED,
  "event": "started"
}
```
If the tracker has any peers associated with that `info_hash`, it will return them in a BenCode, which is a binary serialization format commonly used in BitTorrent. Here's a Python code snippet that demonstrates the process behind the scenes:

```python
import hashlib, bencode, requests, urllib

tracker_url = "https://some_tracker:443/announce"
peer_id = "-UT3530-some_id"
peer_port = 15441
number_wanted = 2
site_sha1 = hashlib.sha1(b"1HELLoE3sFD9569CLCbHEAVqvqV7U2Ri9d").digest()

params = {
 "info_hash": site_sha1,
 "peer_id": peer_id,
 "port": peer_port,
 "uploaded": 0, "downloaded": 0, "left": 431102370,
 "compact": 1,
 "numwant": number_wanted,
 "event": "started"
 }

response = requests.get(tracker_url+"?"+urllib.parse.urlencode(params)).content
print(bencode.decode(response))
```

Now, if we execute this code, we would obtain our peers. However, please note that the peer information would be in binary encoding and would  need to be decoded in order to establish connections with them. In this  blog post, we won't cover the decoding process.

```python
{
 b'complete': 0,
 b'incomplete': 15,
 b'interval': 1800,
 b'min interval': 300,
 b'peers': b'\xbc\xe2D\xf3\x00\x01PuW\xb3\x00\x01'
}
```

The `AnnounceZero` plugin functions in a similar manner, but the request it sends to trackers starting with `zero://` has slightly different parameters. Here is an example of the request:

```python
{
  "hashes": ["SITE_ADDRESS_HASHES"],
  "onions": ["YOUR_ONION_ADDRESSES"],
  "port": YOUR_FILESERVER_PORT,
  "need_types": ["PEERS_TYPES"],
  "need_num": NUMBER_OF_PEERS_NEEDED,
  "add": ["TYPE_OF_ADDRESSES_TO_REACH_YOU"]
}
```

In this request, the `need_types` parameter specifies the type of peers you want to receive, which can be either `onion` or `ipv4`. The `onions` field contains your own onion addresses for communication, and the `add` field indicates the type of addresses (ipv4 or onion) through which others can reach you. If you have set Tor mode to `Always`, the value for `add` will always be `onion`.

There are several other plugins designed for the same purpose. For example, the `AnnounceLocal` plugin enables peer discovery on a Local Area Network (LAN) through UDP broadcasting. The `AnnounceShare` plugin allows peers to share discovered trackers with each other.
Additionally, there is a plugin called `Bootstrapper` that is disabled by default to prioritize user privacy. The `Bootstrapper` plugin functions as a BitTorrent Tracker server running on the user's client, facilitating peer discovery within the ZeroNet network.

# How do we validate the content?

In ZeroNet, the data and content of a site are hosted by its visitors. However, to prevent corruption of the data, ZeroNet utilizes [hash functions](https://en.wikipedia.org/wiki/Hash_function), [checksums](https://en.wikipedia.org/wiki/Checksum), and [cryptographic signatures](https://en.wikipedia.org/wiki/Digital_signature). As mentioned earlier, ZeroNet site addresses are Bitcoin public keys, and the user creating the site possesses the private key.

![Illustration of how checksums work](/assets/pics/checksum.png)

When downloading a ZeroNet site, the first file retrieved is the `content.json` file. This file contains the SHA512 checksum for each available file within the site. Additionally, it includes a `sign` field that ensures the integrity of the content.json file itself. By verifying the signature, we can confirm that the content.json file is valid and has been signed by the owner of the site.

Here is the `content.json` file for an empty site that I just created:

```json
{
 "address": "17u6wJX7fCd9BZwLX8JyhVKTwxb1uEhzcH",
 "address_index": 5655359,
 "background-color": "#FFF",
 "clone_root": "template-new",
 "cloned_from": "1HELLoE3sFD9569CLCbHEAVqvqV7U2Ri9d",
 "description": "",
 "files": {
  "index.html": {
   "sha512": "542f7724432a22ceb8821b4241af4d36cfd81e101b72d425c6c59e148856537e",
   "size": 1114
  },
  "js/ZeroFrame.js": {
   "sha512": "76a24d167e8a4c45f7d1b315efe31e4b4ccb19efef011c080994c94045ff4c93",
   "size": 5088
  }
 },
 "ignore": "",
 "inner_path": "content.json",
 "modified": 1689603718,
 "postmessage_nonce_security": true,
 "signers_sign": "HMGXIah7OTimruksGGqAhzC821bWemxkmS9MG6DlrIJDGpHuIb2MYGO0UpAzxd0nfUzF4x0xxSal1rfbBD0sok4=",
 "signs": {"17u6wJX7fCd9BZwLX8JyhVKTwxb1uEhzcH": "G3jkxsTv+acg0Bzm7+EJZxBznaXdMhrKD3YKSdlMRGsNYUSCzbZ0cSag+VUIkMuPhH2ApveJOglAZcytJik59Lc="},
 "signs_required": 1,
 "title": "my new site",
 "translate": ["js/all.js"],
 "zeronet_version": "0.7.6-internal 2"
}
```

When the `content.json` file is retrieved, ZeroNet looks for the `files` field and downloads the files listed in it. It verifies the integrity of each file by comparing the hash in the `content.json` file with the hash of the downloaded file. If the hashes match, it  means that the file has not been changed or lost during the downloading  process.

The signatures in ZeroNet are based on the same [cryptographic techniques used in Bitcoin's signatures](https://en.wikipedia.org/wiki/EdDSA), which have undergone audits and have been established as secure over a long period of time.

# How sites are created?

ZeroNet's data and settings are stored in the `data` directory. Within this directory, there is a file called `users.json` that contains the private keys for sites, certificates for ID systems, and a `master_seed` field from which site keys are derived.

Here is an example of the `users.json` file:

```json
{
  "1NupA7xwj4qiQzh58Zu4oKn6c3zQUYGdbt": {
    "certs": {},
    "master_seed": "14cacb9f9321b<CENSORED>70a61a9dda362",
    "settings": {
      "theme": "light",
      "use_system_theme": true
    },
    "sites": {
      "17u6wJX7fCd9BZwLX8JyhVKTwxb1uEhzcH": {
    	"auth_address": "18Vijw97tkMp7sYYiuX5pvoewpKybCfxZ2",
        "auth_privatekey": "5JgiXeSAPJ1tF<CENSORED>cx6QSSdXhuxtS",
        "privatekey": "5KNmLu2S6<CENSORED>T1hu1Lthdp2SRr"
      }
    }
  }
}
```

When you create a site in ZeroNet, a random HD Keypair ([hierarchical deterministic keys](https://www.w3.org/2016/04/blockchain-workshop/interest/robles.html)) will be derived from your [BIP32](http://bip32.org/) encoded `master_seed`, and it will be written to your `users.json` file. This allows you to restore all your site's private keys by having your master seed.

The process involves generating an index, which is a random number between `0` and `29639936`. This index is used to derive the private key from your master seed:

```python
index = (a random index between 0 and 29639936)
private_key = HDPrivateKey(master_seed, index)
```

However, it's also possible to use normal Bitcoin keypairs that are not associated with your master seed. In fact, you can even generate a custom public key using tools like VanityGen.

You might notice that there is another keypair named `auth_address` and `auth_privatekey` in the `users.json` file. These two addresses act as your identity on the site, and their primary use case is for identity systems like ZeroID. These keypairs are separate from the site's private keys and serve to authenticate and identify you on the platform.

# How Identities work in ZeroNet?

Identities in ZeroNet are provided by an ID provider such as [ZeroID](https://github.com/HelloZeroNet/ZeroID), (which acts as a centralized system to ensure uniqueness of the IDs). When a user requests an ID, the provider signs them a Certificate, which is a cryptographic signature of their public address and the username they requested. The validity of this Certificate is later verified by matching it with the provider's public key, ensuring the authenticity and integrity of the identity.

The Cert is generated using the following process:

```python
cert = Base64(BitcoinSign(PROVIDER_PRIVATE_KEY, (USER_AUTH_ADDRESS + "#METHOD_OF_CREATION/") + USERNAME))
```

For example, if a user with the address `18Vijw97tkMp7sYYiuX5pvoewpKybCfxZ2` wants the username `zeronet_user` and uses the web interface, the message that would be signed is constructed as follows:

```python
cert = Base64(BitcoinSign(PROVIDER_PRIVATE_KEY, "18Vijw97tkMp7sYYiuX5pvoewpKybCfxZ2#web/zeronet_user"))
```

Here's an example of accepted ID providers in a site's `data/users/content.json`:
```json
"cert_signers": {
  "cryptoid.bit": ["18143WPue3rQykNaopx5KJKzYmaYhCjqhv"],
  "zeroid.bit": ["1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz"]
}
```

# Domain Name System in ZeroNet

Domain names are provided using NameCoin's IDs to help users remember the site's addresses. The way it works is that a server transfers all the NameCoin domains that have the `zeronet` key in them to a ZeroNet site named ZeroName.

Example domain registered for ZeroNet in NameCoin's blockchain:
```json
{
  "name": {
    "formatted": "ZeroNet project"
  },
  "bitcoin": {
    "address": "1QDhxQ6PraUZa21ET5fYUCPgdrwBomnFgX"
  },
  "zeronet": {
    "": "1EU1tbG9oC1A8jz2ouVwGZyQ5asrNsE4Vr",
    "blog": "1BLogC9LN4oPDcruNz3qo1ysa133E9AGg8",
    "talk": "1TaLk3zM7ZRskJvrh3ZNCDVGXvkJusPKQ"
  },
  "ns": [
    "ns1.domaincoin.net",
    "ns2.domaincoin.net"
  ]
}
```

And when a user looks for a site with the domain name `zeronetwork.bit`, it looks it up from the ZeroName site and redirects the user to the public key associated with that domain name.

# How Databases work in ZeroNet?

ZeroNet also provides decentralized databases, making it a dynamic network. The structure of the database is defined in a file named `dbschema.json`, where tables, fields, and types are specified. ZeroNet offers APIs for site developers to interact with the database and modify the data. The data is stored in the site's data directory, and when it changes, it is automatically updated for other peers, ensuring synchronization.

Here's a simplified example of a dbschema for a forum-like site:

```json
{
  "database": "ZeroTalk",
  "tables": {
    "topic": {
      "cols": [
        ["topic_id", "INTEGER"],
        ["title", "TEXT"],
        ["body", "TEXT"],
        ["added", "DATETIME"]
      ]
    },
    "comment": {
      "cols": [
        ["comment_id", "INTEGER"],
        ["body", "TEXT"],
        ["added", "DATETIME"],
        ["topic_id", "INTEGER REFERENCES topic (topic_id)"]
      ]
    }
  }
}
```

And the corresponding `data.json` would look like this:

```json
{
  "topics": [
    {
      "topic_id": 1,
      "title": "First Topic",
      "body": "This is the first topic.",
      "added": "2022-01-01 10:00:00"
    },
    {
      "topic_id": 2,
      "title": "Second Topic",
      "body": "This is the second topic.",
      "added": "2022-02-01 12:00:00"
    }
  ],
  "comments": [
    {
      "comment_id": 1,
      "body": "Comment 1",
      "added": "2022-01-01 11:00:00",
      "topic_id": 1
    },
    {
      "comment_id": 2,
      "body": "Comment 2",
      "added": "2022-02-01 13:00:00",
      "topic_id": 2
    }
  ]
}
```

This example showcases a simple database structure for a forum-like site with two tables: `topic` and `comment`. The `topic` table contains  columns for the topic ID, title, body, and timestamp. Similarly, the `comment` table includes columns for the comment ID, body, timestamp,  and a foreign key referencing the topic ID in the `topic` table. The  accompanying `data.json` file presents sample data entries for topics  and comments.

# How to get started with ZeroNet?

While the official development of ZeroNet has not been active since 2020, there are several forks that continue to work on the project. Some of these forks are even reimplementing ZeroNet in other languages such as Rust, which would eventually increase its security and performance. Here are some links to these forks:

- ZeroNetX - [https://github.com/ZeroNetX/ZeroNet](https://github.com/ZeroNetX/ZeroNet)
- ZeroNet-rs (Rust Implementation) - [https://github.com/ZeroNetX/zeronet-rs](https://github.com/ZeroNetX/zeronet-rs)
- zeronet-conservancy - [https://github.com/zeronet-conservancy/zeronet-conservancy](https://github.com/zeronet-conservancy/zeronet-conservancy)

Additionally, if you're interested in using ZeroNet but don't know where to start, I have created a curated list on my GitHub called "awesome-zeronet". This list provides resources and links to new sites on ZeroNet, and you can also contribute by adding sites that are not yet listed. You can find the list at [https://github.com/zolagonano/awesome-zeronet](https://github.com/zolagonano/awesome-zeronet).

# Final Thoughts on ZeroNet

ZeroNet might be a great approach towards achieving a decentralized web, but it is far from perfect. It lacks encryption for data and still  depends on central servers to operate properly, such as BitTorrent  Trackers and ZeroID. However, it can be improved. If you are interested, you can start contributing to make it better.

---

As always, the entire blog is available on my [GitHub](https://github.com/zolagonano/zolagonano.github.io), and I would greatly appreciate it if you could point out any  inaccuracies, grammatical errors, technical mistakes, or simply give it a star. Your contributions means a lot to me.

Here are some of the resources used in this post:

- [en.wikipedia.org/wiki/ZeroNet](https://en.wikipedia.org/wiki/ZeroNet)
- [zeronet.io/docs/site_development//](https://zeronet.io/docs/site_development/getting_started/)
- [github.com/HelloZeroNet/ZeroID](https://github.com/HelloZeroNet/ZeroID)
- [github.com/HelloZeroNet/ZeroNet](https://github.com/HelloZeroNet/ZeroNet)
