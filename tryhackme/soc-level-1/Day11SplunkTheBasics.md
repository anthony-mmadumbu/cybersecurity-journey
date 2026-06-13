# 🛡️ Splunk: The Basics – TryHackMe

**Path:** SOC Level 1 · Core SOC Solutions

## 📅 Date

Day 11 · Saturday, 13 June 2026

## 📚 What I Learned

- **What Splunk is** — one of the leading SIEM solutions in the industry, used to collect, analyse, and correlate network and machine logs in real time
- The **four core Splunk components**:
  - **Forwarder** — lightweight agent installed on endpoints/servers that ships logs to Splunk
  - **Indexer** — receives, indexes, and stores the incoming data
  - **Search Head** — the web UI where analysts run SPL queries
  - **Deployment Server** — manages forwarder configuration at scale across the estate
- The **log ingestion workflow** — five steps in Splunk's "Add Data" wizard: Select Source → Set Source Type → Input Settings → Review → Done
- **Source type detection** — Splunk auto-detects common formats (JSON, CSV, syslog) and shows a preview before ingestion so analysts can verify the schema before committing
- **Search Processing Language (SPL) basics**:
  - Default fields ingested with every event: `index`, `source`, `sourcetype`, `host`
  - Field-value searches: `FieldName="value"` (case-sensitive on the value, case-insensitive on the field)
  - Negation: `FieldName!="value"` (the cleanest exclude syntax)
  - Implicit AND between multiple terms in a query
- **The Search & Reporting app** — the primary analyst workspace in Splunk, with tabs for Events, Patterns, Statistics, and Visualization
- The **time picker** — defaults to "All time" in the lab but in production analysts almost always scope to a specific window (Last 24h, Last 7 days, custom range) for performance and relevance

## 🛠️ Lab — Ingesting and Investigating VPN Logs in Splunk

Uploaded `VPN_logs.json` to a Splunk instance using the Add Data wizard, configured the index (`vpn_logs`), host (`Vpn_Connections`), and source type (`_json`), then ran a series of SPL queries to investigate the VPN connection data. All 5 questions answered correctly on first attempt and the room was completed at 100%.

### Ingestion Setup

| Setting | Value |
| --- | --- |
| Source file | `VPN_logs.json` |
| Source type | `_json` (auto-detected) |
| Index | `vpn_logs` (created during upload) |
| Host | `Vpn_Connections` |
| Total events ingested | **2,862** |

### Question & Answer Walkthrough

**Q1: How many events are present in the log file?**

✅ **2,862**

**SPL query used:**
```spl
source="VPNlogs.json" host="Vpn_Connections" index="vpn_logs" sourcetype="_json"
```

The query confirms all four ingestion metadata fields and returns the total event count visible in the search results pane. This is the canonical "did my data ingest correctly?" check after using the Add Data wizard.

**Q2: How many log events are captured by the user Maleena?**

✅ **60**

**SPL query used:**
```spl
UserName="Maleena"
```

Splunk had auto-extracted `UserName` as a field from the JSON structure during ingestion, so a simple field-value match was enough. Sixty events from a single user across the dataset gives an immediate sense of that user's VPN activity volume.

**Q3: What is the username associated with IP 107.14.182.38?**

✅ **Smith**

**SPL query used:**
```spl
Source_ip="107.14.182.38"
```

Returned 26 events, all with `UserName: Smith` in the extracted fields. This is the canonical **IP-to-identity pivot** — analysts perform this dozens of times a day during investigations: an alert fires on an IP, and the analyst needs to know who that IP belongs to before deciding next steps.

**Q4: What is the number of events that originated from all countries except France?**

✅ **2,814**

**SPL query used:**
```spl
Source_Country!="France"
```

This question tested the **negation operator** (`!=`) — easy to get wrong if you reach for `NOT` or try to write `Source_Country="!France"`. The `!=` syntax is the cleanest. The result also told me that France accounts for 2,862 − 2,814 = **48 events** in the dataset — useful context if I needed to investigate France-origin traffic later.

**Q5: How many VPN events were associated with the IP 107.3.206.58?**

✅ **14**

**SPL query used:**
```spl
Source_ip="107.3.206.58"
```

Same pattern as Q3 but a different IP — reinforcing that IP-based pivoting is a high-frequency SOC analyst task.

## 🔐 Why It Matters

- **Splunk is the most widely deployed SIEM in the UK enterprise market.** Almost every SOC job advert lists Splunk experience as desirable or required — fluency in SPL is one of the highest-ROI skills a transitioning analyst can build
- The investigation flow demonstrated here — **ingest data → verify totals → field-value search → negation → IP-to-identity pivot** — is the actual cycle an L1 analyst runs many times per shift
- Understanding the difference between **default fields** (index, source, sourcetype, host) and **extracted fields** (UserName, Source_ip, Source_Country) is foundational. Default fields tell you *where the data came from*; extracted fields are *what you actually search on*
- The Add Data wizard is the gateway to almost every Splunk hands-on lab from here onwards — getting comfortable with it now pays off in every later room

## ❓ One Thing I Didn't Fully Understand

All 5 questions correct on first attempt. The negation operator (`!=`) in Q4 is the most interview-relevant piece — it's the syntax beginners most often miss, and being able to write a clean exclude query is a small but visible signal of SPL fluency.

One thing worth noting: I used the default time picker (`All time`) for every query in this lab, which works fine on a small static dataset but would be a performance disaster on a production Splunk instance ingesting terabytes per day. In real SOC work, every SPL query should be scoped to a specific time window first. A lesson to carry forward.

---

**Next up:** Continuing through Core SOC Solutions — likely Incident Handling with Splunk or Investigating with Splunk