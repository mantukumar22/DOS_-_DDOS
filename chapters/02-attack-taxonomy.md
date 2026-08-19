# Chapter 2 — Attack Taxonomy

DoS/DDoS attacks are usually classified two ways: **by OSI layer** (what they exploit) and **by methodology** (what they're trying to exhaust). Understanding both lets you map an observed symptom back to a likely cause and the right mitigation.

## 2.1 Layer-Based Classification

### Layer 3 (Network Layer) Attacks

**ICMP Flood (a.k.a. Ping Flood / historically "Ping of Death")**
- Overwhelms the target with ICMP Echo Request packets
- Consumes network bandwidth and processing resources
- Can cause network congestion and system overload

**IP Fragmentation Attacks**
- Sends fragmented IP packets to exhaust the target's reassembly resources
- **Teardrop attack** — overlapping fragment offsets that crash poorly-written reassembly code
- **Fraggle attack** — UDP-based fragmentation flooding

### Layer 4 (Transport Layer) Attacks

**SYN Flood**
- Exploits the TCP three-way handshake
- Attacker sends many SYN requests, typically with spoofed source IPs
- Victim responds with SYN-ACK but never receives the final ACK
- The connection (half-open) table fills up, preventing legitimate connections

**UDP Flood**
- Sends large volumes of UDP packets to random ports
- Forces the target to reply with ICMP "Destination Unreachable" for each closed port, consuming both bandwidth and CPU

**TCP Connection Flood (TCP State Exhaustion)**
- Establishes large numbers of *legitimate-looking* TCP connections
- Exhausts the server's connection pool and memory resources rather than network bandwidth

### Layer 7 (Application Layer) Attacks

These are harder to detect because the traffic looks like normal, protocol-compliant usage.

**HTTP Flood**
- Overwhelms web servers with a high volume of HTTP requests
- GET/POST flood attacks targeting specific, resource-heavy endpoints
- More sophisticated than network-layer attacks because each request can be indistinguishable from a real user's

**Slowloris**
- Holds connections open by sending partial HTTP requests
- Periodically sends partial headers to keep connections alive without ever completing the request
- Eventually exhausts the web server's connection pool with very little bandwidth used

**Slow POST**
- Sends a legitimate POST request very slowly, trickling the body in over a long period
- Keeps the server waiting for the complete request body, tying up resources for extended periods

## 2.2 Methodology-Based Classification

| Category | Goal | Mechanism | Typical Examples | Measured In |
|---|---|---|---|---|
| **Volumetric** | Overwhelm network bandwidth | High volume of traffic (Gbps scale) | UDP floods, ICMP floods, amplification attacks | Bits per second (bps) |
| **Protocol** | Exhaust server/network-equipment resources | Exploit protocol weaknesses | SYN floods, fragmented-packet attacks | Packets per second (pps) |
| **Application-Layer** | Crash or overwhelm the application itself | Target specific app vulnerabilities/logic | HTTP floods, DNS query floods, Slowloris | Requests per second (rps) |

A single real-world DDoS campaign frequently **blends all three** — e.g., a volumetric flood to saturate the pipe while a parallel Slowloris-style attack quietly exhausts application connections. This multi-vector blending is a defining feature of modern, AI-orchestrated campaigns (see Chapter 9).

## 2.3 Quick Reference — Symptom to Likely Category

| You observe... | Likely attack category |
|---|---|
| Bandwidth saturated, upstream links maxed | Volumetric (L3) |
| Firewall/load balancer connection tables full, CPU spikes on network gear | Protocol (L4) |
| Web server CPU/memory pegged, but bandwidth looks normal | Application (L7) |
| Many connections stuck "open" with little data flowing | Slowloris/Slow POST |
| 5xx errors spiking on specific endpoints | Logic abuse / targeted L7 |

Continue to [Chapter 3 — Attack Simulation Lab: hping3](03-lab-hping3.md).
