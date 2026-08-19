# Chapter 7 — Mitigation & Defense-in-Depth

No single control stops every DoS/DDoS category. Effective defense layers controls that each target a different point in the attack taxonomy from Chapter 2.

## 7.1 Core Mitigation Controls

| Control | What it does | Best against |
|---|---|---|
| **Traffic analysis** | Continuous baselining and anomaly detection (Chapter 6) | All categories — the detection layer everything else depends on |
| **Filtering / ACLs** | Drop traffic matching known-bad signatures (source ranges, protocol anomalies, malformed packets) | Protocol attacks, known botnet ranges |
| **Firewalls** | Stateful inspection, connection-rate limiting at the network edge | SYN floods, basic volumetric |
| **Rate limiting** | Cap the number of requests/connections a single IP (or API key/session) is allowed to make in a time window | Application-layer floods, brute-force-style abuse |
| **Load balancers** | Distribute traffic across multiple backend instances, absorb spikes, enable graceful scaling | Volumetric and connection-based attacks, up to backend capacity |
| **Reverse proxies / CDN** | Cache static content at the edge, absorb request volume before it reaches origin, terminate slow/incomplete connections | HTTP floods, Slowloris/Slow POST, cache-friendly volumetric |
| **DoS prevention software / WAF** | Application-aware filtering (SQLi/XSS rules, behavioral bot detection, JS challenges, CAPTCHAs) | Application-layer/logic-abuse attacks |

## 7.2 Network-Level Best Practices

- **BCP38 / Ingress filtering** — ISPs and network operators dropping packets with obviously spoofed source addresses at the edge, which directly undercuts spoofing-dependent attacks like SYN floods and many amplification techniques.
- **Anycast routing** — announcing the same IP from multiple geographically distributed points of presence so attack traffic is naturally divided across many facilities rather than concentrated on one, and can be absorbed closer to its source.
- **Upstream scrubbing centers** — routing suspect traffic through a specialized third-party (or your own ISP's) scrubbing infrastructure that strips malicious packets before delivering clean traffic to you.
- **Blackholing / RTBH (Remote Triggered Black Hole)** — a last-resort control that null-routes traffic to a specific IP under sustained attack, sacrificing that one target to protect the rest of the network. Should be automated and time-boxed, since it also completes the DoS for legitimate users of that IP.
- **Rate limiting at multiple layers** — network device, load balancer, WAF, and application code, so a single bypass doesn't remove all protection.

## 7.3 Application-Layer Best Practices

- Enforce **timeouts** on incomplete requests/headers (defeats Slowloris/Slow POST by design; see 4.3–4.4).
- Cap **request body size** and **query complexity** (defeats logic-abuse queries like `?limit=999999`).
- Use **connection limits per source IP** at both the load balancer and web server layers.
- Serve **cacheable content from a CDN edge** so origin servers only see requests that truly need dynamic processing.
- Apply **progressive challenge mechanisms** (rate-based CAPTCHA/JS challenge) on sensitive, expensive endpoints like `/login` and `/search` rather than static pages.
- **Graceful degradation** — design critical services to shed non-essential load (e.g., disable expensive recommendation features) under detected stress rather than crashing outright.

## 7.4 Organizational Practices

- **Incident response runbook** specific to DoS/DDoS — who has authority to engage upstream scrubbing, blackhole a prefix, or fail traffic over to a secondary provider, and how fast.
- **Capacity planning with headroom** — know your infrastructure's real ceiling *before* an attack forces you to find out live.
- **Regular authorized DoS-resilience testing** (Chapter 3–4 tools, used properly, in a signed engagement) to validate that mitigations actually trigger as expected — a control that's never been tested is a control you don't actually have. This mirrors the "continuous testing" recommendation increasingly emphasized in 2026 industry threat reports (Chapter 9).
- **Vendor contracts and escalation paths** pre-negotiated with your CDN/scrubbing provider — negotiating DDoS response SLAs mid-attack is far too late.

## 7.5 Matching Controls to Attack Category (Summary)

| Attack category | Primary controls |
|---|---|
| Volumetric (L3) | Anycast, upstream scrubbing, ISP-level filtering, sufficient bandwidth headroom |
| Protocol (L4) | SYN cookies, firewall connection-rate limits, BCP38 ingress filtering |
| Application (L7) | WAF, rate limiting, request timeouts, CDN caching, behavioral bot detection |
| Slowloris/Slow POST | Server-level request/header timeouts (nginx default, `mod_reqtimeout` on Apache), reverse proxy in front of origin |
| Logic abuse | Input validation, query complexity limits, per-endpoint rate limiting |

Continue to [Chapter 8 — Real-World Case Studies](08-case-studies.md).
