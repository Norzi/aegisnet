# aegisnet: building my own VPNs under heavy network censorship

How I taught myself what a VPN actually is, by building one after another on my own servers, in a place where half of the internet is blocked and where "use a VPN" is common advice but almost nobody knows what happens after pressing the button.

This repository is a record of the path, in order: what I built, what I did not understand, what finally clicked, and what broke. It contains no server addresses, keys, subscription links or panel data.

## Projects in this repository

| Folder | What it is |
|---|---|
| [`multipath/`](multipath/) | **6+6**: how I got 12 Mbit/s from a 6 Mbit/s line with sing-box, HAProxy and aria2 |
| [`router/`](router/) | **OpenWRT + zapret + DoH/DoT**: DNS hijacking and SNI blocking found with Wireshark, fixed on the router |

## Timeline

| Age | What I built | What I learned |
|---|---|---|
| 12-14 | My first VPN: **SSH over WebSocket** with a custom payload and an HTTP proxy, in a mobile VPN app | A VPN is a chain of settings (tunnel type, payload, proxy). Proxies get blocked too, so the first thing to change is the proxy, not only the tunnel. There was no AI back then: I read the configuration line by line and worked out what each field did |
| 16-17 | **VLESS + Reality** on my own rented VPS, managed with the **3x-ui** panel | My first real server: SSH access, a web panel, clients, ports. The first time a ping went through my own config was the moment I understood I could build this myself |
| later | **AmneziaWG** | The most durable option in my conditions (see lessons below) |
| later | A node on **Google Cloud** behind a front | Hosting behind infrastructure that is expensive to block. Still alive after months, running on a trial account, so its future is open |

## How I learned without guides

Guides written for "normal" conditions did not work where I live, so I had to find my own way:

1. **Watching how others talk.** I read VPN community chats and saw words like "panel", "clients", screenshots of control panels with the data hidden. I pasted what I saw into an AI assistant and asked what each thing meant, until the vocabulary made sense. That is how I found out that **3x-ui** is the usual tool for running such servers.
2. **Reverse-engineering by example.** I took a working free configuration, gave it to the assistant and asked: "what do I have to set in 3x-ui to build something like this?" I followed the answer step by step and got my first VLESS Reality config. Only after it worked did I go back and study *why*.
3. **Checking who owns an IP.** I used IP lookup services to see which hosting companies owned the addresses of VPNs that worked for me. It taught me that the hosting range matters: some are blocked in bulk, so picking a provider is a technical decision, not only a price one.

## Concepts, in my words

| Term | What it is |
|---|---|
| **VLESS** | A lightweight proxy protocol from the Xray project. It does not encrypt by itself; it relies on the transport layer under it (TLS, Reality) |
| **VMess** | The older Xray protocol with its own encryption. VLESS replaced it in most setups |
| **TCP, TLS, WebSocket** | Transport and security layers you combine with the protocol: plain TCP, TLS-wrapped traffic, or traffic carried over a WebSocket |
| **Reality** | Makes the connection look like a normal TLS handshake with a real, popular website. To anyone who is not an authorized client, the server answers as that website, with its real certificate. No own domain or certificate needed |
| **XHTTP** | A transport that sends data as ordinary HTTP requests. This makes it possible to pass through CDNs and reverse proxies. It is a separate idea from Reality and can be combined with it |
| **3x-ui** | A web panel for creating and managing Xray inbounds and clients without hand-editing configs |
| **AmneziaWG** | A fork of WireGuard that disguises its traffic (junk packets, altered headers) so deep packet inspection cannot recognize it as WireGuard |
| **Fronting** | Placing the server behind a large provider's infrastructure so that the visible destination is the provider, which is costly for a censor to block |

## What I observed (the honest part)

- **Even VLESS + Reality did not last.** The IP address of my VPS was blocked within one to two weeks, every time. Each ban meant paying for an extra IP address and hoping the new one was not already blocked.
- **AmneziaWG survived the longest** in my conditions, because it targets the weakness that gets WireGuard blocked: its recognizable packet signature.
- **Cost was a real constraint.** Paying a foreign host was hard from where I live, and the minimum top-up was a large part of a monthly income here. I took the risk with an intermediary and rented the cheapest VPS, because one user does not need more. It worked, and I did not get scammed, but I would not call it an easy start.
- **Running on a cloud front is the most stable setup so far**, with the caveat that it sits on a limited-time trial account.

## Skills this built

First contact with Linux and SSH (copying commands at first, real understanding came later with my homelab project), reading configurations, understanding how protocols and transports combine, operating a VPS, and thinking in terms of IP reputation and traffic detection: what a censor can see, and what it cannot.

## Next in this repository

- Helper scripts (automation is already in the [aegislab](https://github.com/Norzi/aegislab) repository)

## Note

Most of this was built on my own servers and my own subscriptions. The one exception is the free, publicly shared config described in `multipath/`, which I only used for my own downloads and treat as something that can disappear at any time. Server details are intentionally left out.
