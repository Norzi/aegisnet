# OpenWRT + zapret + DoH/DoT: fixing DNS tampering and SNI blocking on the router

How I flashed my home router with OpenWRT, found out how DNS hijacking and DPI blocking really work by looking at packets, and ended up with a router that gives every device in the house working access to sites that are blocked only by name.

**Hardware:** Xiaomi Mi Router 4A Gigabit Edition, OpenWRT. A small device, but a 6 Mbit/s line does not need more.

## The story in four steps

### 1. zapret ran but did nothing

I had heard that **zapret** (a DPI bypass tool) helps in other countries, but I did not understand how. I installed it, it printed something, and nothing worked. I gave the log to an AI assistant. It said zapret was complaining about **DNS hijacking** by the ISP and needed encrypted DNS. I did not know what either term meant, so I learned them by testing.

### 2. DNS hijacking, seen with my own eyes

DNS is the address book of the internet: you ask "which IP is `example.com`?" and a DNS server answers.

- Without a VPN, I asked for a blocked site and got **`127.0.0.1`**, the address of my own computer. That means "nowhere".
- With a VPN on, the same name returned the real address.
- I tried another public DNS server. **Still `127.0.0.1`.**

That last test was the key one. If a different server gives the same fake answer, then the answer is not coming from the server. Plain DNS (UDP/TCP port 53) is not encrypted, so the ISP can read the question in transit and **answer in its place**, like a man in the middle.

**DoH / DoT** (DNS over HTTPS / over TLS) wrap the question in encryption, so the ISP cannot read or change it. I enabled it on my PC and the same name finally returned a real IP.

### 3. The site still did not open: DPI and the SNI

With the real IP, the site still failed. I captured the traffic with **Wireshark**: right after my computer sent the TLS handshake containing the site name (the **SNI**, a field that is sent unencrypted), a **TCP RST** (connection reset) arrived. A DPI box between me and the internet reads the SNI, sees a blocked name and kills the connection.

zapret includes a test that tries many bypass strategies against one site and tells you which work. I took the working one and put it into the zapret config. The site opened. In Wireshark I saw what it does: it changes how the handshake is sent so the DPI cannot read the site name in one piece, while the real server can still reassemble it.

Important limit I learned right away: **this only works against blocking by name.** A site blocked by **IP address** cannot be fixed with zapret. That needs a VPN.

### 4. From my PC to the whole network, via the router

Running zapret by hand on one PC was annoying, and I wanted it on the phone and TV too. A router can do it for every device. That needs **OpenWRT**.

- **The risk.** Flashing a router can brick it. I counted my savings and made sure I could afford a new one if it went wrong, then did it when I was home alone.
- **The method.** I used my mobile internet during the process. I used two AI assistants for different jobs: one to look up accurate information and one for step-by-step technical instructions. It was scary and it worked.
- **The chicken-and-egg problem.** Installing zapret and DoH/DoT packages needs sites that are blocked, so the router itself needed a VPN. I shared a proxy from my phone, logged in to the router over **SSH** and exported it as the proxy for the package manager (`https_proxy=http://<phone-ip>:<port>`). After that installation worked.
- **Result.** I installed zapret and DoH/DoT, entered the config, and every device on the network started using them without any setup on the device.

## Things that did not work, and what I took from them

- **sing-box on the router did not fit.** The Mi Router 4A has too little RAM and flash for it. I had originally flashed OpenWRT for this, so the plan failed, but zapret turned out to be the better find.
- **It did not matter in the end.** My homelab already provides a **SOCKS5 proxy** with the VPN to the whole network, so any device that needs a VPN connects to it. The router handles "blocked by name", the homelab handles "blocked by IP". See [aegislab](https://github.com/Norzi/aegislab).

## What this taught me

- How a DNS query works and how it can be **tampered with** on the way.
- Why **encrypted DNS is not enough**: the SNI in the TLS handshake is still readable, and DPI can use it.
- How to **prove** a theory with a packet capture instead of trusting an explanation.
- How to pick a bypass strategy by testing, not guessing.
- Basic OpenWRT work over SSH and working through a blocked package repository.

## Try the DNS test yourself

```bash
# 1. Plain DNS to a public resolver. If the answer is a local address
#    like 127.0.0.1 for a normal site, something is rewriting the reply.
nslookup example.com 1.1.1.1

# 2. The same question over HTTPS (curl 7.62 or newer).
curl --doh-url https://cloudflare-dns.com/dns-query https://example.com -I
```

## Config

My router config contains only default paths and a strategy found by the zapret test for my own ISP. It is specific to one provider and changes over time, so it is not published here. If you do the same, run the test yourself.
