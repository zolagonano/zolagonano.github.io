---
title: "A Technical Look at Iran’s Internet Shutdowns"
layout: post
author: Zola Gonano
series: Censorship
---

> This post is written by a human being (me).

Every time mass protests erupt in Iran, a familiar pattern follows: the flow of information stops. The internet slows to a crawl or disappears entirely.

But how does a modern country survive cutting itself off from the internet? Wouldn't that break everything?

Not quite, because the Islamic Republic has spent the last decade building an internet within the internet.

## The National Information Network (NIN): Isolation by Design

Iran’s **National Information Network (NIN)** is a state-controlled intranet designed to keep domestic services running even when international connectivity is cut off. Think of it as a national sandbox: websites, banking portals, messaging apps, and e-government services that function entirely within Iran’s borders.

This setup serves two primary functions:

- It **enables selective blackouts**: the state can block international platforms (like WhatsApp, Instagram, or news sites) while keeping local services (like state media, banking apps) fully operational.
    
- It **forces ISPs to route traffic through government-controlled gateways**, making it easier to monitor, filter, or shut down parts of the network on demand.
    

Layered on top of this is the **IRGFW**—the Iranian Great Firewall. Modeled after China’s Great Firewall (GFW), but with stricter enforcement and more centralized control, it filters, blocks, and surveils traffic across the country.
On paper, it seems solid and impenetrable.

But there’s always a hole. Always.

## What Can Be Done in This Situation?

The Iranian internet blockade is aggressive but it's not perfect. Like any large-scale filtering system, it relies on outdated metadata, and static blocklists. And that's where cracks begin to form.

### The Known Flaw: IPs Are Rented, Not Owned

IPv4 addresses are limited and constantly reallocated. Most are rented and passed between hosting providers, resold between datacenters, or migrated across regions. The Iranian filtering system uses **GeoIP databases and BGP information** to decide which IP ranges to trust and which to block. But those records lag behind the changes.

An IP that once belonged to a local Iranian provider may now be assigned to a server in Amsterdam, but unless the IRGFW updates its filters in real time (which it doesn’t), that IP might still be allowed through.

This opens up a small but real opportunity: **scanning IP space** for reachable proxies, VPNs, or relays that haven’t yet been added to blocklists.

Yes, it’s mostly brute-force. But when everything else is down, even a single working IP can be a tunnel into the world.

### Pingtunnel: Slow, but Better Than Nothing

To prevent scanning or circumvention, the IRGFW has tried to block most outbound protocols. But one thing they haven’t (yet) blocked completely is **ICMP** which is the protocol behind `ping`.

ICMP packets are typically used for diagnostics (checking whether a server is alive), and blocking them outright would break a lot of legitimate network functionality. So they’re still allowed, even under heavy shutdown conditions.

That’s where tools like **Pingtunnel** come in.

Pingtunnel allows you to **smuggle data inside ICMP packets**, essentially tunneling TCP traffic over a stream of pings. It’s really slow and It’s prone to packet loss. But it works, especially for text-based communication, command-line access, or sending small files.

It’s not secure against DPI or timing analysis. But in a shutdown scenario, **"slow but working" is better than nothing**. Even a shell session or basic messaging tool can be a critical line to the outside world.

## Starlinks Behind NATs: Iranians Bypass Censorship Despite Criminalization

Despite Iran’s very aggressive criminalization of Starlink receivers, many Iranians still manage to get their hands on one of them. Starlink connections can be securely shared with others by routing traffic through NAT-enabled local routers using encrypted tunnels like WireGuard.

### What Does “Behind NAT” Means?

Starlink provides an independent internet connection via satellite, bypassing local ISP infrastructure entirely. However, because Starlink devices receive **dynamic, often private IPv4/IPv6 addresses** and operate outside traditional ISP infrastructure, they are typically **behind NATs** when connected to local Iranian networks or user devices.

Here’s what this looks like:

- A **Starlink terminal** (user terminal + router) connects to the satellite network and obtains an IP address from Starlink’s global network.
    
- Meanwhile, the user’s devices in Iran are connected to the **local ISP’s network**, which uses its own IP addressing and NAT systems.
    
- The user sets up a **local router or gateway inside Iran** that bridges these two networks, routing local traffic through the Starlink link.
    

### How This Setup Bypasses Censorship

The key is that **Iranian ISPs and firewalls cannot effectively block Starlink traffic without physically confiscating or disabling the terminals**, because Starlink’s satellites communicate directly with user terminals over encrypted, proprietary protocols.

To share this Starlink connection with friends and family inside Iran, users employ **WireGuard VPN tunnels**:

1. **WireGuard Server Setup:**  
    The user sets up a **WireGuard server on a local Iranian ISP router** that acts as the gateway. This router has two interfaces:
    
    - The **WAN interface** connected to the Iranian ISP network.
        
    - The **LAN interface** connected to the Starlink terminal (or vice versa, depending on setup).
        
2. **Traffic Routing:**  
    Incoming VPN connections from trusted friends or family connect to the WireGuard server on the Iranian ISP router (over the local Iranian network). The router then **routes this decrypted traffic outbound via the Starlink interface**, effectively using Starlink as the internet exit point.
    
3. **NAT and IP Translation:**  
    Because the Iranian ISP router is behind the ISP’s NAT and firewall, it **performs NAT translation** on the incoming WireGuard traffic and routes it over the Starlink network, which has its own IP addressing scheme.
    
4. **WireGuard Configuration Sharing:**  
    The WireGuard config files containing public keys, endpoint addresses, allowed IPs, and ports are shared securely with trusted users. This allows them to establish encrypted tunnels into the Starlink-connected router from their own devices within Iran.
    

### Why This Setup Is Resilient

- **Starlink traffic is encrypted and satellite-based**, making it extremely difficult for IRGFW to inspect or block without physical interference.
    
- **WireGuard uses UDP and a small handshake footprint**, making detection and blocking via DPI harder.
    
- **NAT hides the underlying IP structure**, so Iranian firewalls see only standard encrypted traffic to the local router — they can’t easily distinguish if that traffic is then routed over Starlink.
    
- This also **allows multiple users to share a single Starlink terminal**, maximizing scarce and expensive hardware.
    

## How to Survive Inside the NIN?

Sometimes, when the National Information Network (NIN) is up but Iran’s connection to the global internet is completely cut off, you simply _can’t_ reach friends and family outside the country. Even communicating inside Iran becomes a challenge.

So what are your options?

**SMS?**  
SMS in Iran is unencrypted. The government can—and does—intercept, read, and store messages at will. Plus, SMS is slow, costly, and unreliable for anything beyond short texts.

**Phone calls?**  
Calls use GSM networks, which are fully controlled by the state. Voice calls can be intercepted, recorded, and monitored without any user consent. Privacy here is nonexistent.

---

### Exploiting the Local Network: Self-Hosted Encrypted Services

But here’s the catch: the NIN _does_ allow traffic inside Iran’s local network. This means you can run services completely inside Iran, avoiding the censored global internet but still enabling communication.

One powerful option is setting up your own **end-to-end encrypted messaging and calling service**. like a **Matrix Synapse server** which can be fully hosted on a VPS inside an Iranian ISP.

**Why Matrix?**  
Matrix is an open-source protocol that supports secure, E2E-encrypted messaging, voice, and video calls. It’s decentralized, so anyone can run their own server. If your server is inside Iran, and your contacts connect locally, you avoid international censorship and heavy filtering.

**What do you need?**

- A VPS hosted by an Iranian ISP, so the server is reachable inside the NIN.
    
- A domain name pointing to that VPS, ideally registered with an Iranian registrar or configured to resolve within the NIN’s DNS system.
    
- The Matrix Synapse server software installed and configured on the VPS.
    

**How does it help?**  
Since the traffic never leaves the country’s internal network, it’s harder for the government to block without cutting off the NIN entirely. And because Matrix uses strong encryption, even if the government inspects packets, the content remains private.

---

Surviving inside the NIN means adapting to its limitations and exploiting its architecture. Running your own encrypted local communication service allows Iranians to stay connected securely, even when the global internet is a no-go.

It’s not perfect. there are risks, and setting up these services requires some technical knowledge. but it’s a crucial lifeline in a digital environment designed to isolate and surveil.
