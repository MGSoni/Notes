# Networking & DNS Basics

**Problem it solves:** Users need a stable, human-friendly way to reach a server whose IP may not even be static, without every client having to know raw IPs.

## Core idea
- **IP address** identifies a machine on a network. IPv4 = 32 bits (~4B addresses, already insufficient globally). IPv6 = 128 bits (effectively unlimited).
- Most home/office connections get a **dynamic IP** from the ISP (a static IP costs extra). The ISP's router is the actual internet-facing IP; it forwards to your device using port/MAC info.
- **Domain name → IP** mapping is what makes URLs usable instead of raw IPs.
  - You buy a domain via a **registrar** (GoDaddy, Namecheap, Cloudflare).
  - Registrar relays the purchase to **ICANN**, the central authority.
  - Resolution is actually done by a distributed **DNS** system (hierarchical tree of servers) — not ICANN directly, since ICANN alone would be a bottleneck/SPoF.
  - Anyone can run a DNS server (Google, ISPs, orgs); if you don't configure one, your ISP's default is used.

## When to use / trade-offs
- Central lookup (single authority) → simple but becomes a bottleneck and single point of failure at scale.
- Distributed hierarchical DNS → solves both, at the cost of more moving parts and (eventual, not instant) propagation of DNS changes.

## Key number
- IPv4: 2^32 ≈ 4 billion addresses vs 100B–1T devices on the internet today → address exhaustion is why IPv6 (2^128) exists.

## Connects to
This is the entry point before Load Balancing — DNS resolves a domain to the IP(s) of your **Load Balancer(s)**, not to individual app servers.
