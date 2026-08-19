# Chapter 1 — Introduction & Fundamentals

## 1.1 What Is Denial of Service?

**Denial-of-Service (DoS)** is an attack in which the service offered by a system or network is deliberately denied, degraded, or made unavailable to legitimate users. Availability — one leg of the confidentiality/integrity/availability (CIA) triad — is the target. The attacker doesn't need to steal or alter data; they only need to exhaust a finite resource (bandwidth, CPU, memory, connection table entries, or application logic) faster than the victim can replenish it.

## 1.2 DoS vs. DDoS

| | DoS | DDoS |
|---|---|---|
| Source | Single attacking host | Many distributed hosts (a botnet) |
| Scale | Limited by one machine's/network's capacity | Amplified by the number of participating nodes |
| Traceability | Easier to block a single source IP | Sources are geographically and logically distributed, making IP blocking far less effective |
| Typical origin today | A single script/tool run locally | Compromised IoT devices, cloud VMs, or "booter/stresser" rental services |

**Objective (both):** make resources unavailable to legitimate users.
**Impact:** service disruption, financial loss, reputational damage, and — for critical infrastructure — potential safety consequences.

## 1.3 Common Symptoms of a DoS/DDoS Attack

- Slow performance of the network or system
- Increase in spam email (SMTP flooding can itself be a DoS vector)
- Unavailability of a particular resource or service
- Loss of access to a website or web application
- Disconnection of wireless or wired internet connections
- Denial of access to any internet service
- Unusual or asymmetric network traffic patterns
- Server crashes or repeated freezing

No single symptom proves an attack — but several appearing **together and suddenly** is a strong signal (see Chapter 5 for how this looks in logs).

## 1.4 Why Attackers Do This

Motives vary widely and matter for triage and response:

- **Hacktivism / ideological protest** — e.g., groups claiming responsibility for attacks tied to geopolitical events
- **Extortion ("ransom DDoS")** — pay-or-be-flooded threats against businesses
- **Competitive sabotage** — disrupting a rival's service (common in gaming and e-commerce)
- **Distraction** — a DDoS as cover while a separate intrusion or data exfiltration happens elsewhere
- **Booter/stresser-for-hire economics** — low-cost DDoS-as-a-service platforms have collapsed the skill barrier to entry
- **Testing / bragging rights** — demonstrating botnet capability

## 1.5 Key Terminology

| Term | Meaning |
|---|---|
| **Botnet** | A network of compromised devices (bots/zombies) controlled by an attacker, often via a C2 (command-and-control) server |
| **Amplification/Reflection attack** | Spoofing the victim's IP as the source of a small request to a third-party service (DNS, NTP, memcached) that replies with a much larger response, "reflecting" and amplifying traffic onto the victim |
| **Volumetric attack** | Aims to saturate available bandwidth (measured in bps/Gbps/Tbps) |
| **Protocol attack** | Exploits weaknesses in L3/L4 protocol handling to exhaust server or network-device state tables (measured in pps) |
| **Application-layer (L7) attack** | Targets application logic/resources with seemingly legitimate requests (measured in rps) |
| **C2 (Command and Control)** | Infrastructure the attacker uses to issue instructions to bots |
| **Booter/Stresser** | A commercial (usually illegal when misused) DDoS-for-hire service |

## 1.6 Learning Objectives for This Repo

By the end of this repo you should be able to:

1. Explain the fundamentals of DoS and DDoS attacks and how they differ.
2. Recognize the major attack vectors and techniques across the OSI stack.
3. Safely simulate representative attacks in an isolated lab for training purposes.
4. Detect attack patterns in raw logs and in a SIEM (Splunk).
5. Apply layered mitigation controls appropriate to each attack category.
6. Explain how AI has changed both the offensive and defensive sides of this problem as of 2025–2026.

Continue to [Chapter 2 — Attack Taxonomy](02-attack-taxonomy.md).
