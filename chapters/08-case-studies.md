# Chapter 8 — Real-World Case Studies

Theory and lab practice mean little without grounding in how these attacks actually played out against real infrastructure. Each case below maps back to the taxonomy in Chapter 2.

## 8.1 BBC Websites — New World Hacking "Bangstresser" (December 31, 2015)

On the morning of December 31, 2015, the BBC's websites — including its main news site, the iPlayer catch-up service, and the iPlayer Radio app — went down for several hours, greeting visitors with "Error 500 – Internal Error" messages starting around 07:00 GMT. The BBC initially described it only as a "technical issue" before confirming it had been hit by a distributed denial-of-service attack.

A group calling itself **New World Hacking**, which says it primarily targets pro-ISIS online activity, later claimed responsibility, describing the incident as a **test** of a new tool it called *Bangstresser* — reportedly capable of generating floods up to roughly 600 Gbps — ahead of planned use against IS-affiliated targets. One member told the BBC the group didn't intend to take the site down for multiple hours, remarking that "our servers are quite strong."

**Category:** Volumetric, hacktivist/testing motive.
**Lesson for defenders:** even "test" traffic from a capable actor can produce outages at major-broadcaster scale; capacity headroom and upstream scrubbing agreements matter regardless of attacker intent.

## 8.2 DDoS Attacks on Dyn (October 21, 2016)

On October 21, 2016, three consecutive distributed denial-of-service attacks were launched against the DNS provider **Dyn**. Because Dyn provided DNS resolution for a large number of major internet platforms, the attack caused widespread unavailability across large swathes of users in Europe and North America — sites like Twitter, Reddit, Netflix, GitHub, and many others became unreachable, even though those companies themselves weren't directly attacked; **their DNS provider was**. The groups Anonymous and New World Hackers claimed responsibility, though little evidence was provided to substantiate that claim.

**Category:** Volumetric, powered by the **Mirai botnet** — a network of compromised IoT devices (routers, IP cameras, DVRs) — one of the first large-scale demonstrations of IoT-powered DDoS at internet-infrastructure scale.
**Lesson for defenders:** this incident is the canonical example of **supply-chain/dependency risk** in availability planning — your own hardening means little if a single upstream DNS or CDN provider is a single point of failure for your entire user base. It also marked the moment IoT botnets became a mainstream, headline-level threat, a trend that has only accelerated (see Chapter 9's Aisuru botnet discussion).

## 8.3 Google Cloud Armor — Largest DDoS Mitigated to Date (October 2023, disclosed publicly)

Google's Cloud Armor security team, in a post co-authored by Product Manager Emil Kiner and Security Reliability Engineer Tim April, disclosed that Google had mitigated the **largest DDoS attack recorded to that date**, peaking above **398 million requests per second**. The attack used a **novel technique called HTTP/2 Rapid Reset**, which abused stream multiplexing within the HTTP/2 protocol — opening and immediately cancelling large numbers of streams within a single connection to force the server to do disproportionate work per request while the client's own resource cost stayed low.

**Category:** Application-layer (L7), protocol-abuse variant — notable because it broke prior assumptions about how much a single connection could be forced to cost a server, at massive scale, using a legitimate, standardized protocol feature rather than a spoofed-packet flood.
**Lesson for defenders:** this attack forced a coordinated, industry-wide response (Google, Cloudflare, AWS, and others disclosed simultaneously) and led to patches in HTTP/2 server implementations to cap in-flight stream resets — a reminder that **protocol-level vulnerabilities can undercut volumetric-style defenses entirely**, since the attack traffic itself may look small and legitimate per-connection.

## 8.4 GitHub — 1.35 Tbps Memcached Amplification (February 2018)

*(Widely documented industry case, included for taxonomy completeness.)* GitHub was hit by a then-record 1.35 Tbps attack using **memcached amplification** — attackers spoofed GitHub's IP address in small requests to publicly exposed memcached servers, which responded with vastly larger payloads directed at GitHub. Amplification factors for memcached abuse were measured in the tens of thousands of times the original request size.

**Category:** Volumetric, reflection/amplification.
**Lesson for defenders:** amplification attacks depend on (a) IP spoofing going unfiltered upstream (BCP38 — Chapter 7) and (b) misconfigured third-party services responding to spoofed requests from arbitrary IPs; this remains one of the strongest arguments for widespread ingress filtering adoption across ISPs.

## 8.5 The Aisuru Botnet — Record-Breaking Hypervolumetric Attacks (2025–2026)

The most disruptive DDoS threat of 2025–2026, the **Aisuru** botnet (with a variant tracked as **KimWolf**), harnesses an estimated 500,000+ compromised IoT devices and Android-based systems — including Android TV and streaming devices — to launch hypervolumetric attacks. By December 2025, Aisuru had launched the **largest DDoS attack ever recorded, peaking at 31.4 Tbps**, lasting just 35 seconds. As of Arelion's 2026 threat landscape report, Aisuru alone drives roughly **one-third of all global DDoS attack traffic** observed on its backbone.

**Category:** Volumetric, IoT-botnet-driven, increasingly AI-orchestrated (see Chapter 9 for the full trend analysis).
**Lesson for defenders:** IoT device proliferation combined with weak default credentials and unpatched firmware continues to hand attackers an ever-larger pool of exploitable nodes; the DDoS defense posture appropriate for 2016-era Mirai traffic volumes is no longer sufficient against 2026-era hypervolumetric, multi-terabit floods.

## 8.6 Case Study Comparison Table

| Case | Year | Category | Peak Scale | Key Lesson |
|---|---|---|---|---|
| BBC / Bangstresser | 2015 | Volumetric | ~600 Gbps | Even "test" attacks can down major infrastructure |
| Dyn | 2016 | Volumetric (Mirai/IoT) | Multi-region outage via DNS | Upstream/DNS provider is a single point of failure |
| GitHub | 2018 | Volumetric (amplification) | 1.35 Tbps | Ingress filtering (BCP38) stops amplification abuse |
| Google Cloud Armor | 2023 | Application-layer (protocol abuse) | 398M rps | Protocol-level flaws can bypass volumetric defenses |
| Aisuru botnet | 2025–2026 | Volumetric (IoT/AI-orchestrated) | 31.4 Tbps | Modern IoT botnets dwarf 2016-era attack scale |

Continue to [Chapter 9 — DDoS in the AI Era](09-ddos-in-the-ai-era.md).
