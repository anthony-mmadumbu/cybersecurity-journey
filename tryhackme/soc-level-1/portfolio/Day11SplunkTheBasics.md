# Splunk: The Basics — Log Ingestion, SPL Fundamentals, and the IP-to-Identity Pivot Write-up

**Lab Context:** TryHackMe — SOC Level 1 · Core SOC Solutions · Splunk: The Basics
**Role Played:** SOC Analyst running a Splunk Enterprise instance with full search head access
**Focus:** Ingesting JSON VPN logs into a new index, then running a progression of SPL queries that mirror real Tier 1 investigation patterns

---

## Executive Summary

This write-up documents the first hands-on Splunk lab in the SOC Level 1 path — moving from the conceptual SIEM rooms (Introduction to SIEM, Introduction to EDR) into actually operating one of the most widely deployed SIEM products in industry.

The lab tested three skill clusters in sequence:

1. **Data ingestion** — using the Add Data wizard to bring a JSON log file into a new Splunk index with the right source, source type, host, and index metadata
2. **SPL fundamentals** — writing field-value searches, negation queries, and IP-based pivots
3. **Investigation patterns** — mirroring real SOC L1 workflow where an analyst pivots from raw log volume → user-specific activity → IP-to-identity resolution → exclusion-based filtering

All 5 lab questions were answered correctly on first attempt; the room was completed at 100%. The deeper takeaway is that **SPL fluency is a force multiplier for SOC L1 work** — the same five-step ingestion wizard and the same handful of operators (`=`, `!=`, implicit AND, field-value match) cover the vast majority of day-to-day analyst queries. Mastery of the basics scales further than mastery of edge cases.

---

## Background — Why Splunk Specifically

Splunk Enterprise is the dominant on-premises and cloud-deployable SIEM in the UK enterprise market. It is named by name in nearly every SOC Analyst job advert at Tier 1 level. Its strengths:

- **Mature query language (SPL)** — expressive, fast, and well-documented
- **Wide vendor support** — almost every security product produces Splunk-ready logs out of the box
- **Search-time field extraction** — fields can be parsed at query time without changing the underlying data
- **Apps and add-ons** — pre-built integrations for hundreds of products via Splunkbase
- **Massive community knowledge base** — questions answered, queries shared, blueprints available

The alternative SIEMs (Microsoft Sentinel, IBM QRadar, Elastic Security, LogRhythm, ArcSight) have their own query languages and idioms, but the *investigation patterns* learned in Splunk transfer directly. Learning SPL is investing in skills that compound across the wider SIEM ecosystem.

---

## Splunk Architecture Recap

The four primary components a SOC analyst encounters:

| Component | Role |
| --- | --- |
| **Forwarder** | Lightweight agent on endpoints/servers; ships logs to Splunk over an encrypted channel |
| **Indexer** | Receives, parses, indexes, and stores incoming events; the storage and search engine layer |
| **Search Head** | The web UI; where analysts run SPL queries, build dashboards, and define alerts |
| **Deployment Server** | Manages forwarder configuration centrally — used at enterprise scale to push config to thousands of agents |

In smaller deployments, indexer and search head can run on the same host. In production at scale, they are separated and clustered for performance and redundancy.

For this lab, the Splunk Enterprise instance presented all of these layers through a single web interface — typical for hands-on training environments.

---

## Stage 1 — Data Ingestion via the Add Data Wizard

Splunk's Add Data wizard is a five-step flow that almost every analyst encounters within their first hour of using the product:

1. **Select Source** — choose the log file or source type and how to bring it in (Upload, Monitor, Forward)
2. **Set Source Type** — Splunk auto-detects common formats (JSON, CSV, syslog, etc.) and previews the parsed events before ingestion
3. **Input Settings** — choose the destination *index* and the *host* metadata field
4. **Review** — confirm every setting before committing
5. **Done** — the data is now ingested and searchable

### Ingestion Configuration Used in This Lab

| Setting | Value | Why |
| --- | --- | --- |
| Source file | `VPN_logs.json` | Provided in `/root/Rooms/SplunkBasic/` on the AttackBox |
| Source Type | `_json` | Auto-detected; preview confirmed structured JSON parsing |
| Index | `vpn_logs` | Created during upload — keeping a dedicated index per data source is best practice |
| Host | `Vpn_Connections` | Logical name for the data origin |

The preview screen during Set Source Type was particularly informative — Splunk had already parsed the JSON and exposed every field (`Company`, `EventTime`, `extracted_index`, `port`, `protocol`, `Source_Country`, `Source_ip`, `source_state`, `UserName`) as searchable columns. This is the moment an analyst confirms that **the data will be queryable in the way they expect** before committing.

### Total Events Ingested

**2,862 events** ingested cleanly. The `VPN_logs.json` dataset spans VPN connection records (`built` and `teardown` actions) for various users across multiple source IPs and countries.

---

## Stage 2 — SPL Investigation Walkthrough

The five questions formed a deliberate progression from broad verification queries to focused pivots. Each one mirrors a real Tier 1 investigation pattern.

### Q1 — Ingestion Verification

**Question:** How many events are present in the log file?
**Answer:** 2,862

```spl
source="VPNlogs.json" host="Vpn_Connections" index="vpn_logs" sourcetype="_json"
```

#### What This Query Demonstrates

Four of Splunk's default fields scoped explicitly:

- `source` — the originating file or stream
- `host` — the logical origin Splunk associates with the events
- `index` — the storage bucket
- `sourcetype` — the data format

Together, these four fields form Splunk's "address" for any given log set. In production, an analyst rarely scopes all four because some are redundant — but as a **post-ingestion sanity check**, naming all four confirms that every part of the ingestion configuration worked as intended.

#### Real-World Parallel

After any new log source is onboarded, the analyst's first job is to verify the data is searchable and the event count matches what the source claimed to send. This query is that verification step.

---

### Q2 — Field-Value Search

**Question:** How many log events are captured by the user Maleena?
**Answer:** 60

```spl
UserName="Maleena"
```

#### What This Query Demonstrates

A simple field-value match on a JSON-extracted field. Three points worth noting:

- **No need to re-specify the index** — Splunk's "default search context" still scopes to the data the analyst was just looking at, but in production a query like this would normally include `index=vpn_logs UserName="Maleena"` for performance
- **Field names are case-sensitive** in SPL — `UserName="Maleena"` ≠ `username="Maleena"`. Splunk extracted the field with its original JSON capitalisation
- **String values are case-insensitive by default** — `UserName="maleena"` would return the same 60 events

#### Real-World Parallel

User-specific activity searches are one of the most common SOC L1 queries:
- "How many failed logons did this account have in the last 24 hours?"
- "What did this user do in the hour before their account got disabled?"
- "Are there events for this user from outside our office hours?"

All variants of the same `UserName=` pattern.

---

### Q3 — IP-to-Identity Pivot

**Question:** What is the username associated with IP 107.14.182.38?
**Answer:** Smith

```spl
Source_ip="107.14.182.38"
```

#### What This Query Demonstrates

The query returned 26 events. Inspecting the extracted fields on any returned event revealed:

```
Company: CyberT
EventTime: 2022-01-31T18:22:08
Source_Country: United States
Source_ip: 107.14.182.38
UserName: Smith
```

This is the **IP-to-identity pivot** — one of the highest-frequency operations in real SOC work.

#### Real-World Parallel

An alert fires referencing an IP address. The analyst needs to know:

1. Who logged in from that IP?
2. Are they someone who should be using that IP (expected travel, known VPN exit, branch office)?
3. Are there any other accounts using the same IP simultaneously (account-sharing red flag)?

In this lab the answer was simple — one user (Smith) on one IP. In a real investigation, the answer might be three users sharing one IP (perfectly normal — multiple staff behind one corporate NAT), or one user across twenty IPs in five countries in a day (very abnormal — possible account takeover or credential leak).

The query is trivial. The *interpretation* is where the analyst adds value.

---

### Q4 — Negation Operator

**Question:** What is the number of events that originated from all countries except France?
**Answer:** 2,814

```spl
Source_Country!="France"
```

#### What This Query Demonstrates

The **negation operator** (`!=`) — the cleanest way to write an exclude condition in SPL. Three syntactic alternatives exist:

| Syntax | Works? | Notes |
| --- | --- | --- |
| `Source_Country!="France"` | ✅ Recommended | Cleanest, most readable |
| `NOT Source_Country="France"` | ✅ Works | Equivalent; `NOT` is the boolean operator form |
| `Source_Country="!France"` | ❌ Wrong | Treated as literal string match on "!France" |

Using `!=` is the SPL idiom. Reaching for `NOT` is a common beginner habit; reaching for `"!France"` is the most common beginner *mistake*.

#### What the Result Tells Us

2,862 total events − 2,814 non-France events = **48 events from France**. This kind of arithmetic-by-difference is a useful analyst habit: every exclusion query implicitly tells you something about the excluded set.

#### Real-World Parallel

Exclusion queries are everywhere in SOC work:

- *"Show me all logons except service accounts."* → `EventCode=4624 NOT user IN (svc_*, sa_*)`
- *"Show me outbound connections except to our allowlisted vendor IPs."* → `direction=outbound NOT dst_ip IN (10.1.0.0/16, 192.168.50.0/24)`
- *"Show me alerts except the ones we've already triaged today."* → `index=alerts NOT triaged=true`

Mastering negation early is a multiplier.

---

### Q5 — Repeated Pivot Pattern

**Question:** How many VPN events were associated with the IP 107.3.206.58?
**Answer:** 14

```spl
Source_ip="107.3.206.58"
```

#### What This Query Demonstrates

The same IP-pivot pattern as Q3, but applied to a different IP — reinforcing that the pattern is reusable. Different IP, fourteen events instead of twenty-six, different user behind it — but the query shape is identical.

#### Why the Lab Repeated the Pattern

The repetition is pedagogically deliberate. The room is teaching that **the analyst's job is rarely a one-time clever query — it's the same handful of query patterns applied over and over to different inputs**. SOC work is pattern-based investigation, not one-off detective work.

---

## Cross-Question Lessons Carried Forward

1. **Ingestion verification is step zero of every Splunk investigation.** Before searching for anything specific, confirm that the data is there, that the event count matches expectations, and that the fields you need are extracted. Skipping this step is how analysts end up writing perfectly correct queries against silently incomplete data.
2. **The four default fields (`index`, `source`, `sourcetype`, `host`) are the metadata of every event.** Knowing what each one means and how to use them for scoping is foundational. In production, every query should include `index=` at minimum for performance.
3. **Extracted fields are where the analytical value lives.** `UserName`, `Source_ip`, `Source_Country` were all parsed automatically because the data was JSON. For non-structured logs (syslog, custom application logs), an analyst may need to write field extractions manually — a skill for later rooms.
4. **The IP-to-identity pivot is the single most common SOC L1 query pattern.** Master it once; reuse it for the rest of your career.
5. **`!=` is the right way to exclude in SPL.** Learn this early; it removes one of the most common beginner stumbles.
6. **Always think about time scope in production.** This lab used "All time" because the dataset was static and small. Production Splunk instances ingest terabytes per day — every query needs a time window first.
7. **SPL fluency compounds.** Every query you write makes the next one faster. The same six operators (`=`, `!=`, `AND`, `OR`, `NOT`, `IN`) cover most of what an L1 analyst writes — depth matters less than reps.

---

## How This Room Builds on Earlier SOC Level 1 Rooms

The progression through the SOC Level 1 path is now visible:

| Room | What It Taught | How Splunk Builds On It |
| --- | --- | --- |
| Introduction to SIEM | Conceptual model — log sources, correlation, alerting | Splunk *is* the SIEM; SPL is how you operate it |
| Introduction to EDR | Endpoint-level investigation surface | EDR alerts often feed into Splunk for correlation with other sources |
| SOC Workbooks | Decision-diamond investigation playbooks | Each step in a workbook usually maps to one SPL query |
| SOC Alert Triage | TP/FP verdict logic | SPL is how the analyst gathers evidence to support the verdict |
| SOC Metrics | MTTD, MTTA, MTTR, FPR | SPL queries are what populate the dashboards behind every SOC metric |
| Pyramid of Pain | Detection at increasing levels of attacker pain | Behavioural detections in Splunk are how those high-pyramid detections actually fire |
| Eviction (APT28) | Translating threat intel into hunt queries | Splunk SPL is the language those hunt queries are written in |

This room is a hinge point — the conceptual work of the earlier rooms now starts to land on actual queries an analyst would write at a job.

---

## Tools, Concepts & Frameworks Referenced

- **Splunk Enterprise** (version 8.2.6 in the lab) — the market-leading on-premises SIEM
- **SPL (Search Processing Language)** — Splunk's query language
- **Add Data wizard** — the five-step ingestion flow (Select Source → Set Source Type → Input Settings → Review → Done)
- **Default Splunk fields** — `index`, `source`, `sourcetype`, `host`
- **Field extraction** — automatic for JSON/CSV, configurable for unstructured data
- **Search & Reporting app** — the primary analyst workspace
- **SPL operators used in this lab** — `=`, `!=`, implicit AND

## Summary of Questions, Answers, and SPL Used

| # | Question | Answer | SPL Query |
| --- | --- | --- | --- |
| 1 | How many events in the log file? | 2,862 | `source="VPNlogs.json" host="Vpn_Connections" index="vpn_logs" sourcetype="_json"` |
| 2 | How many events from user Maleena? | 60 | `UserName="Maleena"` |
| 3 | Username for IP 107.14.182.38? | Smith | `Source_ip="107.14.182.38"` |
| 4 | Events from all countries except France? | 2,814 | `Source_Country!="France"` |
| 5 | VPN events for IP 107.3.206.58? | 14 | `Source_ip="107.3.206.58"` |

All 5 answered correctly on first attempt. Room completed at 100%.

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*