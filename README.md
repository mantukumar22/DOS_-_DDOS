# DoS & DDoS Masterclass
### Attack Simulation, Detection & Defense Strategies — with a chapter on the AI Era

A structured, chapter-wise study repo on Denial-of-Service (DoS) and Distributed Denial-of-Service (DDoS) attacks: how they work, how they're simulated in a lab for defensive training, how to detect them in logs and SIEM tools, how to mitigate them, and how the threat (and the defense) has changed now that AI is in the loop on both sides.

This repo is written for **defenders** — SOC analysts, blue teamers, students preparing for certifications like CEH/Security+/OSCP, and anyone who needs to understand DDoS well enough to detect and stop it. It is **not** an attack toolkit. Commands referenced here are the same well-known, publicly documented syntax found in vendor manuals, RFCs, and every mainstream security certification curriculum — included strictly so defenders can recognize what an attack looks like on the wire and in logs.

---

## ⚠️ Legal & Ethical Notice

- Only run any of the tools or techniques referenced in this repo against systems you **own** or have **explicit, written authorization** to test (e.g., your own lab VMs, a CTF platform like TryHackMe/HTB, or an engagement covered by a signed scope-of-work).
- Unauthorized DoS/DDoS activity is illegal in most jurisdictions under computer misuse / fraud statutes (e.g., the U.S. Computer Fraud and Abuse Act, UK Computer Misuse Act) and can result in criminal prosecution, civil liability, and permanent loss of trust.
- This material is for **education, detection engineering, and authorized red-team/blue-team exercises only**.

---

## 📚 How This Repo Is Organized

| # | Chapter | Summary |
|---|---------|---------|
| 1 | [Introduction & Fundamentals](chapters/01-introduction-fundamentals.md) | What DoS/DDoS is, key terminology, impact, symptoms |
| 2 | [Attack Taxonomy](chapters/02-attack-taxonomy.md) | Layer-based (L3/L4/L7) and methodology-based (volumetric/protocol/application) classification |
| 3 | [Attack Simulation Lab: hping3](chapters/03-lab-hping3.md) | Building a safe lab and using hping3 for SYN/UDP/ICMP/XMAS/etc. flood simulation |
| 4 | [Attack Simulation Lab: Frameworks & Scripts](chapters/04-lab-frameworks-scripts.md) | LOIC, HOIC, Slowloris, Nmap DoS scripts, Scapy, Metasploit auxiliary modules |
| 5 | [Detection: Reading the Logs](chapters/05-detection-log-analysis.md) | Web server log indicators, sample log walkthroughs, what a real attack pattern looks like |
| 6 | [Detection: SIEM & Splunk](chapters/06-detection-splunk-siem.md) | SPL queries, timecharts, building detections at scale |
| 7 | [Mitigation & Defense-in-Depth](chapters/07-mitigation-defense.md) | Rate limiting, WAF/CDN, Anycast, BCP38, blackholing, upstream scrubbing |
| 8 | [Real-World Case Studies](chapters/08-case-studies.md) | Dyn 2016, Google Cloud Armor 398M rps, GitHub 1.35 Tbps, BBC/Anti-IS, Aisuru botnet |
| 9 | [DDoS in the AI Era](chapters/09-ddos-in-the-ai-era.md) | AI-driven attack automation, agentic bots, AI-assisted detection/mitigation, 2025–2026 research & industry data |
| 10 | [Study Guide & Further Practice](chapters/10-study-guide-further-practice.md) | Lab checklist, TryHackMe/CTF rooms, cert mapping, glossary |

---

## 🧪 Suggested Lab Environment

- Isolated hypervisor (VMware Workstation / VirtualBox) on a **host-only or NAT network with no route to the internet**
- Attacker VM: Kali Linux
- Target VM(s): Metasploitable2, an intentionally vulnerable Windows 10 VM, or a disposable web app (DVWA/Juice Shop)
- Optional: a Splunk (or ELK) instance ingesting the target's access logs for detection practice

## 🗺️ Suggested Learning Path

1. Read Chapters 1–2 for theory.
2. Build the lab and work through Chapters 3–4 to see what attacks look like from the *attacker* vantage point (safely, in isolation).
3. Flip perspective — Chapters 5–6 teach you to spot the same attacks from the *defender* vantage point in logs and SIEM.
4. Chapter 7 closes the loop with mitigation controls.
5. Chapters 8–9 ground the theory in real incidents and the current (2025–2026) AI-driven threat landscape.
6. Chapter 10 is your checklist and glossary for review.

## 🙏 Sources

Chapters 1–8 are compiled and rewritten from standard, publicly-taught DoS/DDoS curriculum (CEH-style course material, hping3/Metasploit documentation, TryHackMe's "Detecting Web DDoS" room, and public post-incident write-ups such as the Dyn 2016 Wikipedia summary and Google's Cloud Armor blog post on the October 2023 HTTP/2 Rapid Reset attack). Chapter 9 is sourced from 2025–2026 industry threat reports (Radware, Akamai, Arelion, Corero, Cloudflare/StationX aggregation) and peer-reviewed ML/DDoS-detection research (Springer, MDPI, Nature Scientific Reports, arXiv), cited inline.

---

*Maintained as a personal/team study repo. Contributions welcome via PR — please keep additions defensive/educational in framing.*
