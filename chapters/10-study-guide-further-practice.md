# Chapter 10 — Study Guide & Further Practice

## 10.1 Self-Check Questions

Work through these after each chapter to test retention (no answer key on purpose — write your own explanation in your own words, then verify against the chapter):

1. What's the practical difference between a protocol attack and an application-layer attack, in terms of what resource is actually exhausted?
2. Why does a single-IP firewall block stop a DoS but usually fail against a DDoS?
3. Why is Slowloris effective even though it uses almost no bandwidth?
4. Name three log-level indicators that, together, suggest a coordinated DDoS rather than an organic traffic spike.
5. Why do fixed-threshold alerts fail against pulse attacks, and what's the alternative?
6. Which mitigation control specifically defeats IP-spoofing-dependent amplification attacks, and why does it have to be deployed upstream (at the ISP) rather than by the victim?
7. What made the Google Cloud Armor 2023 incident notable beyond its sheer scale?
8. What is the Aisuru botnet, and why does it represent a step-change from earlier IoT botnets like Mirai?
9. In what sense does AI "not favor one side" in the DDoS arms race?

## 10.2 Hands-On Lab Checklist

- [ ] Build an isolated attacker/target lab (Kali + Metasploitable2 or similar), verified to have **no route to the internet**
- [ ] Snapshot both VMs
- [ ] Run and packet-capture a SYN flood with hping3; identify the signature in Wireshark
- [ ] Run and packet-capture a UDP flood; note the ICMP unreachable responses
- [ ] Simulate a Slowloris-style connection-exhaustion pattern conceptually; observe connection table growth on the target
- [ ] Configure an access log pipeline; generate a burst of requests and locate the indicators from Chapter 5 in the raw log
- [ ] Stand up (or use an existing) Splunk/ELK instance; build the timechart query from Chapter 6.1 against your generated logs
- [ ] Build the statistical outlier query from Chapter 6.3 and tune it against your own baseline traffic
- [ ] Map at least one mitigation control from Chapter 7 to each attack you simulated
- [ ] Restore VM snapshots when done

## 10.3 Recommended External Practice

- **TryHackMe — "Detecting Web DDoS"** room: covers DoS/DDoS fundamentals, attack motives, and — most usefully — a full log-analysis and Splunk-based detection task set against a simulated `access.log`, directly reinforcing Chapters 5–6.
- **TryHackMe — general DoS/DDoS-adjacent rooms** under Hacktivities and Active Reconnaissance paths for supporting network-fundamentals practice.
- **CIC-DDoS2019 dataset** (Canadian Institute for Cybersecurity) — the standard public dataset used across most of the 2025–2026 ML-DDoS-detection papers cited in Chapter 9; useful for hands-on model-building practice if you want to go beyond signature-based detection.

## 10.4 Certification Mapping

| Certification | Where this material maps |
|---|---|
| **CEH (Certified Ethical Hacker)** | Directly aligned — this repo mirrors a standard "Module 10: DoS/DDoS" structure |
| **CompTIA Security+** | Domain covering attacks/threats/vulnerabilities (availability attacks) |
| **CompTIA Network+** | Understanding of TCP/IP handshakes, ICMP, protocol behavior underlying Chapters 2–3 |
| **GCIH / GCIA (SANS/GIAC)** | Chapters 5–7 (detection, incident handling, mitigation) are most directly relevant |
| **OSCP** | Chapters 3–4 lab practice, applied under strict "systems you're authorized to test" scope |

## 10.5 Glossary

| Term | Definition |
|---|---|
| **Amplification attack** | Spoofing a victim's IP in a small request to a third-party service that replies with a much larger response, multiplying attack volume |
| **Anycast** | Routing technique announcing the same IP from multiple locations, naturally distributing incoming (including attack) traffic |
| **BCP38** | Best Current Practice 38 — network ingress filtering to prevent IP-spoofed packets from leaving a network |
| **Blackholing / RTBH** | Null-routing traffic destined for an under-attack IP as a last-resort containment measure |
| **Booter/Stresser** | A commercial DDoS-for-hire platform |
| **Botnet** | A network of compromised devices controlled by an attacker, typically via C2 infrastructure |
| **C2 (Command and Control)** | Infrastructure used to issue instructions to compromised bots |
| **DDoSaaS** | DDoS-as-a-Service — rentable attack platforms, often built on established botnets |
| **HTTP/2 Rapid Reset** | A 2023-disclosed technique abusing HTTP/2 stream multiplexing/cancellation to force disproportionate server-side cost per client request |
| **Pulse attack** | A short, high-intensity burst designed to degrade service while staying under fixed-threshold detection |
| **RPS / PPS / BPS** | Requests-per-second / Packets-per-second / Bits-per-second — the standard measurement unit for L7, L4/protocol, and L3/volumetric attacks respectively |
| **Scrubbing center** | Third-party (or ISP) infrastructure that filters malicious traffic before delivering clean traffic to the target |
| **SYN cookie** | A stateless technique for handling the TCP handshake that avoids allocating resources until the handshake actually completes, mitigating SYN floods |
| **Volumetric attack** | An attack aimed at saturating available network bandwidth |
| **WAF (Web Application Firewall)** | An application-aware filtering layer that can apply behavioral/logic rules against L7 attacks |

## 10.6 Closing Note

DoS/DDoS is one of the few attack categories where the offense and defense are locked in a genuinely symmetric, continuously escalating race — and as Chapter 9 covers, AI has only sharpened that symmetry rather than resolving it in favor of either side. The fundamentals in Chapters 1–7 don't go stale; they're the substrate every new AI-driven technique in Chapter 9 still gets built on top of. Master the fundamentals first — the packet-level mechanics, the log signatures, the layered mitigations — and the AI-era material becomes a natural extension rather than a separate subject.

---
[← Back to README](../README.md)
