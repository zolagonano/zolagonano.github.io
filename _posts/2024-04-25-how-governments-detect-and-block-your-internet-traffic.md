---
title: How Governments Detect and Block your Internet traffic 
layout: post
series: Censorship
---

If you have ever lived in a country with advanced internet censorship, such as China or Iran, you would know how challenging it is to bypass these restrictions. In this post, I want to discuss the methods by which these firewalls block and detect your traffic, as well as the circumvention tools and methods available for each of them.

An advanced censorship system can employ a combination of these methods based on multiple factors, such as the geolocation of the destination server or its data center, as well as packet fingerprinting and throttling of suspicious traffic, among others.

## IP Blocking

[Ip blocking](https://en.wikipedia.org/wiki/IP_address_blocking) is the simplest and easiest method for firewalls to implement. It involves checking the destination address of traffic, and if it matches a certain IP or IP range, the firewall will either block the traffic or route it to another IP.

Because of its simplicity, it's quite easy to bypass as well. All you have to do is route the traffic through another server whose IP is not on a block list using a [proxy server](https://en.wikipedia.org/wiki/Proxy_server).

## DNS Spoofing

Most of the time, an IP can be used for multiple services. For example, YouTube and other Google services might share the same IP addresses. This would make IP blocking quite expensive for a government because they might inadvertently block some services they didn't intend to block.

In this situation, the easiest thing that can be done to block specific websites and services would be [spoofing the user's DNS traffic](https://en.wikipedia.org/wiki/DNS_spoofing) (as DNS lacks encryption and can be read along the way). If the destination domain matches a specific domain, it would return an invalid address or not respond to the user at all.

The fix for this would be relatively straightforward as well. All we have to do is encrypt the DNS traffic using protocols such as [DNS-Over-TLS](https://www.cloudflare.com/learning/dns/dns-over-tls/) or [DNS-Over-HTTPS](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-tls/), [DNSSEC](https://www.cloudflare.com/dns/dnssec/how-dnssec-works/), or [DNSCrypt](https://www.dnscrypt.org/). Because the firewall wouldn't see what domain name we're asking for, it can't determine whether we're trying to access a blocked website or not.

## Protocol Blocking

In some extreme cases, a firewall might fully block a protocol, mostly those used for VPNs such as PPTP and L2TP.

This method is usually applied to protocols that wouldn't incur significant costs. For example, if they were to block HTTPS or TLS, it would disrupt most websites regardless of whether they are blocked or not, and that can be quite expensive.

To overcome this type of blocking, some protocols have been created to make proxied traffic appear as normal web or TLS traffic as much as possible. Protocols such as [REALITY](https://github.com/XTLS/REALITY) and [Trojan](https://trojan-gfw.github.io/trojan/) are designed for this purpose.

## Packet Filtering

A firewall might filter packets and block or throttle them based on factors such as their protocol (whether they're TCP or UDP), size and length, destination port, and headers.

For example, in extreme cases, they might fully block UDP traffic and only allow TCP connections to certain ports, such as 443 and 22, which are common ports for TLS and SSH.

Bypassing this type of restriction is not difficult, but it will come with speed and bandwidth sacrifices, as you might be limited to using TCP connections, which are typically slower than UDP ones, especially for streaming.

## Deep Packet Inseption (DPI)

[DPI (Deep Packet Inspection)](https://en.wikipedia.org/wiki/Deep_packet_inspection) is a method used to inspect and filter traffic in real-time, meaning that the firewall will open your packets, analyze their content, and decide whether to block them or allow them to pass through.

DPI can even detect and block fully encrypted traffic through methods like fingerprinting, allowing it to determine whether the traffic is VPN or proxy traffic or normal traffic.

Bypassing DPI is actually challenging, as DPI systems are constantly evolving and becoming more advanced at detecting encrypted traffic. However, so are the protocols for bypassing them.

The most promising and well-tested protocol for bypassing DPI is obfuscating protocols, such as [obfs4](https://gitlab.com/yawning/obfs4), which are designed to make the traffic appear as noise or normal traffic.

## Active Probing

[Active probing](https://ensa.fi/active-probing/), unlike other methods of censorship, isn't passive; it involves automated servers sending requests to other servers to check if they're being used for bypassing censorship or not.

These servers might act as obfs4 clients and send requests to your server to examine its response.

Preventing these probes is a challenging task, but there are measures that can be taken, such as only allowing connections from specific IP addresses or blocking any suspicious attempts on the server."

---

My blog and all of its content are available under the `CC by SA 4.0` License on my [GitHub](https://github.com/zolagonano/zolagonano.github.io). If you notice any problems or have any improvements for the blog or its content, you're always welcome to open a pull request.

