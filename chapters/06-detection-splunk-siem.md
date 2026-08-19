# Chapter 6 — Detection: SIEM & Splunk

Manually reading logs doesn't scale past a handful of requests. A SIEM (Splunk, ELK, Sentinel, etc.) lets you turn the indicators from Chapter 5 into repeatable, automated queries and visualizations across millions of events.

## 6.1 Visualizing Request Volume Over Time

The most fundamental DDoS-detection query is a simple timechart of request volume, bucketed by a short time span and broken out by the requested URI:

```spl
index="main" | timechart span=1min count by uri limit=5
```

**What this does:** counts events per 1-minute bucket, split into series by `uri`, keeping the top 5 URIs. Plotted as an area/line chart, this makes an attack visually obvious:

- **Normal user requests** — a few requests to various pages every minute; a low, relatively flat baseline across multiple URIs.
- **DoS attack** — a single URI (e.g., `/login.php`) spikes to something like 1,000 requests within a single one-minute window, dwarfing every other series on the chart. On a chart with a normal baseline of single digits per minute, a spike to 1,000 is not subtle — it's the visual signature of an attack in progress.

## 6.2 Drilling Down by User-Agent and Client IP

Once volume anomalies are visible, pivot to see *who* is generating them:

```spl
index="main" uri="/login.php" | timechart span=1min count by useragent
```

```spl
index="main" uri="/login.php" | timechart span=1min count by clientip
```

Filtering over the same requests by `useragent` and `clientip` lets you see more detail about where the traffic originated:

- If one `clientip` accounts for the entire spike → likely a single-source **DoS**.
- If the spike is spread across many distinct `clientip` values hitting the same URI in the same window, with a similar (or rotating) `useragent` → likely **DDoS** from a botnet.

## 6.3 Building a Repeatable Statistical Detection

Rather than eyeballing a chart every time, encode the Chapter 5 indicators as SPL logic that flags anomalies automatically. A simplified pattern:

```spl
index="main"
| bin _time span=1m
| stats count by _time, uri, clientip
| eventstats avg(count) as avg_count, stdev(count) as stdev_count by uri
| where count > (avg_count + 3*stdev_count)
```

This flags any `(uri, clientip, minute)` combination whose request count exceeds three standard deviations above that URI's historical average — a basic but effective statistical outlier detector that doesn't require hardcoding a fixed threshold per endpoint.

## 6.4 Correlating Errors With Volume

Combine the volume view with the error-rate indicator from Chapter 5 to confirm actual *impact*, not just anomalous traffic (which could be a legitimate flash-crowd event, e.g. a marketing campaign going viral):

```spl
index="main"
| bin _time span=1m
| stats count as total, count(eval(status>=500)) as errors by _time
| eval error_rate = round((errors/total)*100, 2)
| where error_rate > 10
```

A traffic spike *without* a corresponding error-rate spike is more likely organic growth; a spike *with* a jump in 5xx responses strongly suggests the backend is being overwhelmed.

## 6.5 Building a Practical Dashboard

A minimal DDoS-detection dashboard should include:

1. **Requests-per-minute timechart**, top URIs
2. **Requests-per-minute by client IP**, top offenders
3. **Error-rate-over-time** panel
4. **Geo-distribution map** of source IPs (geographic anomalies indicator)
5. **User-Agent distribution** (odd/rotating UA indicator)
6. **Alert** wired to the statistical outlier query in §6.3, routed to your SOC's paging/ticketing system

## 6.6 Why Statistical Baselines Beat Fixed Thresholds

A fixed threshold ("alert if > 500 req/min") breaks in two directions:

- Too low → false-positives on legitimate traffic spikes (product launches, viral posts, breaking news)
- Too high → misses low-and-slow or pulse-style attacks that stay under a static bar (see Chapter 9 — AI-tuned "pulse attacks" are specifically designed to duck under naive thresholds)

Per-endpoint statistical baselining (§6.3), combined with multi-signal correlation (volume + errors + UA + geography), is far more resilient — and is the same underlying principle modern AI-assisted DDoS defenses use, just implemented with more sophisticated models (Chapter 9).

Continue to [Chapter 7 — Mitigation & Defense-in-Depth](07-mitigation-defense.md).
