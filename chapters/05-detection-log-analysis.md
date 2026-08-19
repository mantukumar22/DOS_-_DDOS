# Chapter 5 — Detection: Reading the Logs

Web server logs are a valuable source of evidence when investigating denial-of-service attacks. Every major web service — Apache, NGINX, Microsoft IIS — records web requests in a broadly standardized log format. By examining these logs, analysts can uncover patterns that separate normal user traffic from malicious activity.

## 5.1 Key Indicators to Look For

| Indicator | Example | Why it matters |
|---|---|---|
| **High Request Rate** | `10.10.10.100 → 1000 GET /login` | A resource-heavy page like `/login` flooded with requests can overwhelm authentication logic — each request may trigger password checks and database queries |
| **Odd User-Agents** | `curl/7.6.88 → /index` repeatedly | Attackers spoof outdated or unusual User-Agents, or fail to spoof one at all; seeing `curl` or `Python-urllib/3.x` hammering the same endpoint is a red flag for automated (non-browser) traffic |
| **Geographic Anomalies** | IP origins dotted around the world | Legitimate traffic usually clusters where your real users are; a botnet may show IPs scattered globally with no organic geographic logic |
| **Burst Timestamps** | 50 requests in 1 second → `/search` | A sudden spike packed into the same second (or sub-second) window is an unnatural pattern that points to automation, not human browsing |
| **Server Errors (5xx)** | Spike of `503 Service Unavailable` | A surge of `500`–`511` responses indicates the origin is maxed out and struggling under the load |
| **Logic Abuse** | `GET /products?limit=999999` | Crafted queries designed to force the server to load and process huge amounts of data, slowing it down for everyone |

**No single indicator is proof.** Analysts should look for multiple, layered signals forming a coherent picture. For example: imagine an attacker controlling a worldwide botnet aimed at a single site. You might see requests from a wide range of IP addresses across different geographic regions, hammering several resource-intensive endpoints, with the same (or a small rotating set of) User-Agent strings. Maintaining a watchlist of these indicators is a valuable habit for any analyst's toolkit.

## 5.2 Targeted Resources — Why Attackers Pick What They Pick

If an attacker aims to disrupt a web service, they'll usually focus on endpoints that consume the most server resources per request, or that are most critical to maintaining site functionality. Pages like `/login` or search forms are prime targets because each request forces the server to query a database, validate input, and return results — far more expensive to process than static content like product images.

| Endpoint | Why it's a common target |
|---|---|
| `/login` | Involves authentication processes (hashing, DB lookups, session creation) |
| `/search` | Requires complex database queries, often with joins/full-text search |
| `/api/*` | Critical for dynamic content delivery; often lacks caching |
| `/register`, `/signup` | Requires database writes and validation |
| `/contact`, `/feedback` | Requires database entries and can trigger email notifications |
| `/cart`, `/checkout` | Requires session management, inventory checks, and payment processing |

## 5.3 Sample Log Walkthrough

Let's examine a sample condensed access log to see how a DoS attack might appear in a post-incident scenario.

1. **Normal user traffic** — every few seconds, a user requests a page and receives a response as expected.
2. **DoS attack begins** — beginning at `10:01:10`, the IP `203.0.113.55` begins sending repeated `GET` requests to `/login.php`.
3. **Web server down** — users start requesting pages and receiving `503` responses indicating the service is unavailable.

```text
# Normal traffic
192.168.1.10 [31/Aug/2025:10:01:02] "GET /index.html HTTP/1.1" 200 4523 "Mozilla/5.0"
192.168.1.15 [31/Aug/2025:10:01:05] "GET /products.html HTTP/1.1" 200 6112 "Mozilla/5.0"
192.168.1.20 [31/Aug/2025:10:01:09] "GET /contact.html HTTP/1.1" 200 2987 "Mozilla/5.0"

# Attack begins — same IP, same endpoint, same tiny response, non-browser UA, sub-second repetition
203.0.113.55 [31/Aug/2025:10:01:10] "GET /login.php HTTP/1.1" 200 245 "curl/7.68.0"
203.0.113.55 [31/Aug/2025:10:01:10] "GET /login.php HTTP/1.1" 200 245 "curl/7.68.0"
203.0.113.55 [31/Aug/2025:10:01:10] "GET /login.php HTTP/1.1" 200 245 "curl/7.68.0"
203.0.113.55 [31/Aug/2025:10:01:10] "GET /login.php HTTP/1.1" 200 245 "curl/7.68.0"
203.0.113.55 [31/Aug/2025:10:01:10] "GET /login.php HTTP/1.1" 200 245 "curl/7.68.0"
...100 more lines
```

This log snippet is highly condensed for illustration — a real DoS or DDoS incident may have hundreds or thousands of requests flooding the logs simultaneously, often from *many* source IPs rather than one, which is exactly what distinguishes DDoS from DoS in the log data itself.

## 5.4 What Made This One Easy to Spot

Stacking the indicators from §5.1 against the sample:

- ✅ Same source IP hammering the same endpoint (high request rate)
- ✅ Non-browser User-Agent (`curl/7.68.0`)
- ✅ Sub-second repetition (burst timestamps — identical second, identical response size)
- ✅ Followed by a spike in `503` responses (server errors)

A real distributed attack would spread the "high request rate" indicator across many source IPs instead of one — which is precisely why a **single-IP block** stops a DoS but rarely stops a DDoS, and why aggregate/statistical detection (Chapter 6) matters more than IP-by-IP rules at scale.

## 5.5 Limitations of Log-Based Detection Alone

- Logs only show what reached the application layer — a volumetric L3/L4 flood may take the server offline *before* it can log anything meaningful.
- Sophisticated attackers rotate User-Agents and IPs specifically to defeat naive log-signature rules (see Chapter 9 on AI-assisted evasion).
- Log volume during a real attack can itself overwhelm log storage/ingestion, so centralized, rate-limited, or sampled logging pipelines matter operationally.

These limitations are exactly why log analysis is paired with SIEM-based statistical detection, covered next.

Continue to [Chapter 6 — Detection: SIEM & Splunk](06-detection-splunk-siem.md).
