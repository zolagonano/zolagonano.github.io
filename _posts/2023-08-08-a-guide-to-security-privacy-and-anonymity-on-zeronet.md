---
title: A Guide to Security, Privacy, and Anonymity on ZeroNet
layout: post
series: ZeroNet
---

In my previous post, I took [a technical look at ZeroNet](/blog/posts/a-very-technical-look-at-zeronet), explaining how it works and the technologies it uses to create a peer-to-peer web-like network. In this post, I want to discuss how you can maintain privacy and security in this network, explore the potential threats, and provide some techniques to enhance your privacy and security.

I will divide this guide into three main sections: "Security," "Privacy," and "Anonymity." In each section, I will explain what you can do to enhance each aspect.

These sections are interconnected, and as a result, there will be a lot of overlap. For example, you will need security to ensure your privacy, and you will need privacy to stay anonymous.

# Security

This section covers what is needed to protect assets, information, or prevent harm, damage, unauthorized access, or exploitation.

## Ensure Browser Security

In the [ZeroNet](https://zeronet.dev) network, no servers are hosting the sites, which means everything is executed on the client-side. Every site you visit will run JavaScript or [WASM](https://webassembly.org/) code in your browser. This makes browser sandboxing critically important because you cannot simply block every script and expect things to function properly. Therefore, the most important thing to stay secure in such a network is the browser.

### Use a well-known browser

Use a browser that has been around for a while, as it will have had the chance to be audited and tested, and most of its vulnerabilities are likely to be fixed. Browsers like Firefox or Chromium are good options, as they have sandboxing features and vulnerabilities are detected and fixed quickly. However, the best choice would be the [Tor Browser](https://www.torproject.org/download/), which is a modified version of Firefox provided by [the Tor Project](https://torproject.org). It is designed to reduce fingerprinting and enhances security by enabling some sandboxing features of Firefox that are not enabled by default.

### Keep your browser up-to-date

Usually, most attacks come from already known and fixed vulnerabilities, and it is very unlikely to be attacked by a zero-day vulnerability. That's why you should always keep your browser and all your software up-to-date. It prevents attacks through known and fixed vulnerabilities and reduces your attack surface area significantly.

## Encrypt All ZeroNet Traffic

By default, traffic encryption is not enforced for all requests and clients in ZeroNet. However, you can enforce it by adding `force_encryption = True` to the `[global]` section in the `zeronet.conf` file. Alternatively, you can run your ZeroNet with the `--force_encryption` flag, but you may accidentally forget to do so, so it's safer to put it in the config file.

By doing so, all your traffic will be secured using [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security).

### Add an Additional Layer of Encryption

You can also provide an additional layer of encryption by using a [VPN(Virtual Private Network)](https://en.wikipedia.org/wiki/Virtual_private_network) or by tunneling your whole system through TOR. This will prevent your [ISP(Internet Service Provider)](https://en.wikipedia.org/wiki/Internet_service_provider) from knowing you're connecting to ZeroNet nodes.

But note that using a VPN means you're putting your trust in your VPN provider instead of your Internet Service Provider.

## Encrypt Your ZeroNet Data

ZeroNet data, such as site files, even your private keys, etc., are stored in plain text inside a directory named `data`, and ZeroNet itself doesn't provide encryption for those files yet. However, there are several tricks to implement encryption for these files.

### Use ZeroNet inside an encrypted virtual machine

If you want to take it to the next level and increase the security and privacy of your ZeroNet client at the same time, you can install a fully encrypted OS inside a [virtual machine](https://en.wikipedia.org/wiki/Virtual_machine) and run your ZeroNet inside it. This approach provides strong sandboxing and security for your ZeroNet data. An OS recommended for such purposes is [Whonix OS](https://www.whonix.org/), and they have a [guide](https://www.whonix.org/wiki/ZeroNet) on how to use ZeroNet inside Whonix.

### Store your ZeroNet files in an encrypted container

This method is simpler and offers added benefits, such as portability. You can encrypt a USB stick and store your entire ZeroNet bundle there, enabling you to use it on any machine you desire.

For enhanced portability, I highly recommend using [Tails OS](https://tails.net), which routes all traffic through Tor and includes an [Encrypted Persistent Storage](https://tails.net/doc/persistent_storage/index.en.html) feature. This allows you to store files like your ZeroNet bundle securely without having them deleted when you restart the Tails Os.

## Do not log in when using a ZeroNet instance (public proxy)

Everyone who runs a ZeroNet public proxy can see your private key and potentially steal your identity. Therefore, never use your identity or perform actions like creating an ID or site using a public proxy, as the proxy manager will have access to that site and ID just as you do.

# Privacy

This section covers what you can do to control what information about you is shared or accessed when using ZeroNet.

## Always use Tor

Setting Tor mode to `Always` will ensure that your IP address is not exposed when connecting and seeding a site in ZeroNet. Instead of IP addresses, other ZeroNet nodes will communicate with you through your [onion address](https://tb-manual.torproject.org/onion-services/), and you have multiple of them (10 by default) to reduce the fingerprinting of an onion address.

To do so, you need to open a Tor controller port and allow ZeroNet to have access to the cookie authentication file. 

Here are the changes you will need to make in the `torrc` file to achieve this:

1. Uncomment or add this line to open the controller port:

```
ControlPort 9151
```

2. Add these lines to make the cookie authentication file readable for ZeroNet:

```
CookieAuthFile /var/lib/tor/control_auth_cookie
DataDirectoryGroupReadable 1
CacheDirectoryGroupReadable 1
CookieAuthentication 1
```

3. Add your user or the user that runs ZeroNet to the Tor group:

```bash
sudo chown -R your_user:tor /var/lib/tor/control_auth_cookie
```

Once you've made these changes, you can add the following lines to the `[global]` section in your `zeronet.conf` file to enforce ZeroNet to use Tor:

```yaml
tor_proxy = 127.0.0.1:9050
tor_controller = 127.0.0.1:9151
tor = always
```

With these configurations, your ZeroNet instance will route its traffic through Tor, providing increased privacy and anonymity.

## Use Tor Browser or Disconnect Your Browser from the Internet

Even if you have set Tor to 'Always' mode, the sites can still send a request from your browser to the clearnet and expose your real IP address. The best practice here would be to always use ZeroNet inside the Tor browser to ensure that your real IP address never leaks to the clearnet through your browser. Optionally, you can set a non-working proxy or use your browser in offline mode when using ZeroNet, as ZeroNet doesn't require your browser to have access to the internet.

## Protect your Identity

In ZeroNet, all identities are represented by [private keys and certificates](https://en.wikipedia.org/wiki/Public-key_cryptography), which are stored in the `users.json` file. Therefore, it is essential to regularly back up this file if you want to ensure you don't lose your identity on ZeroNet.

### Use ZeroID if possible

ID providers like CryptoID or KaffieID are considered "decentralized," but this decentralization comes at the cost of lacking uniqueness and regulation on IDs. As a result, anyone could generate the same ID that you have and potentially use your identity. On the other hand, ZeroID follows a different approach with a centralized server that ensures usernames and IDs are unique. However, it is worth noting that using ZeroID inside the Tor Browser should help maintain privacy despite the data being sent to the clearnet.

# Anonymity

This section covers what you can do to seperate your real identity from the identity that you have on ZeroNet.

## You need almost everything from the Security and Privacy sections

There's a lot of overlap between security, privacy, and anonymity, and you should have those to achieve anonymity.

## You need good OPSEC

"[OPSEC](https://en.wikipedia.org/wiki/Operations_security)" stands for Operational Security. It involves using specific practices and strategies to protect sensitive information. When you have applied most of the security and privacy practices, it's mostly about OPSEC. You are the one who chooses what to share, where to share, and with whom.

---

Privacy is a human right, and digital privacy is necessary for people living in dictatorships, journalists, or activists to stay safe in the real world. If there's anything else that can be added, I'd be more than happy to know. This entire blog is available on [GitHub](https://github.com/zolagonano/zolagonano.github.io), and you can contribute to it or simply give it a star.
