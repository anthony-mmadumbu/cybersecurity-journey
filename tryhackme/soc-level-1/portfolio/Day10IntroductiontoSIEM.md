# Introduction to SIEM — Centralised Detection, Correlation, and the Analyst's Investigation Loop Write-up

**Lab Context:** TryHackMe — SOC Level 1 · Core SOC Solutions · Introduction to SIEM
**Role Played:** SOC Analyst with access to the SIEM dashboard, events table, rule editor, and action interface
**Focus:** Walking the full SIEM investigation loop — from alert trigger through event identification, context enrichment, rule analysis, verdict, and response action — on a real cryptominer detection

---

## Executive Summary

This write-up documents an end-to-end SIEM investigation of a "Potential CryptoMiner Activity" alert. The exercise tested whether the analyst could fluently work through the **full investigation loop** that defines Tier 1 SOC work: alert detection → event identification → context enrichment → rule examination → verdict (TP/FP) → response action.

All six questions were answered correctly on first attempt and the room was completed at 100%. The deeper takeaway is structural: a SIEM is not just a log aggregator — it's the **central investigation surface** that turns raw telemetry from across the entire estate into actionable alerts, and the analyst's job is to operate fluidly within that surface.

This room pairs with the *Introduction to EDR* room earlier in the SOC Level 1 path. Together they cover the two foundational analyst tools: EDR (deep visibility on a single endpoint) and SIEM (broad visibility across the entire estate). Mature SOCs run both.

---

## Background — What a SIEM Is and Why It Exists

**SIEM (Security Information and Event Management)** is the central platform that:

1. **Ingests logs** from every security-relevant source in the environment
2. **Normalises** them into a common schema so analysts can search across sources uniformly
3. **Correlates** events using rules and analytics to detect patterns no single log source would reveal
4. **Alerts** the SOC when correlated activity matches predefined detection logic
5. **Provides the investigation surface** — dashboards, queries, drill-downs — analysts use to triage

### The Limitations of Working with Isolated Logs

Without a SIEM, a SOC analyst investigating a single incident would have to:

- Log into the Windows endpoint console for Event Logs
- Log into the firewall console for outbound connection logs
- Log into the EDR console for process telemetry
- Log into Active Directory for authentication events
- Log into the cloud platform for IAM events
- Log into the proxy for URL logs
- ...and *manually cross-reference timestamps* across all of them

That doesn't scale. One alert across six consoles already takes 20 minutes of context-switching. A real SOC handling hundreds of alerts a day cannot operate this way. **The SIEM exists to dissolve the gap between log sources.**

### Common Log Sources Feeding a SIEM

| Source | Typical Format | What It Tells You |
| --- | --- | --- |
| Windows Event Logs | Windows Event XML / EVTX | Process creation (4688), authentication (4624/4625), service installs, registry, etc. |
| Linux syslog | Syslog / journalctl | Authentication, sudo, service activity, crontab |
| Firewall logs | Vendor-specific / CEF | Allowed and denied connections, NAT events |
| IDS/IPS | Vendor-specific / Snort / Suricata | Signature matches, anomaly alerts |
| EDR | Vendor-specific / JSON | Process trees, file events, network connections, behavioural alerts |
| Active Directory | Windows Event Log | Authentication events, group membership changes |
| Web servers | Common Log Format / JSON | HTTP requests, response codes, user agents |
| DNS | Vendor-specific | Queried domains, resolved IPs |
| Cloud platforms | JSON via API | IAM events, resource changes, network flow logs |
| Email gateway | Vendor-specific | Inbound/outbound mail, attachment analysis, sender reputation |

### Common SIEM Products in Industry

Splunk is the market leader and the product most frequently named in UK SOC job adverts; Microsoft Sentinel is the fastest-growing alternative (cloud-native, tightly integrated with the Microsoft security stack). Other significant products include IBM QRadar, Elastic Security (ELK Stack), LogRhythm, Securonix, and Exabeam. The next set of rooms in the SOC Level 1 path drills deeper into Splunk and ELK specifically.

---

## The SIEM Investigation Loop — The Core of This Lab

The investigation tested the full loop in compact form:

```
                  ┌────────────────────────────────────────┐
                  │       ALERT FIRES (correlation rule)   │
                  └──────────────────┬─────────────────────┘
                                     │
                  ┌──────────────────▼─────────────────────┐
                  │  IDENTIFY THE EVENT (find the matching │
                  │  log record — user, host, process)     │
                  └──────────────────┬─────────────────────┘
                                     │
                  ┌──────────────────▼─────────────────────┐
                  │  ENRICH WITH CONTEXT (department,      │
                  │  business need, baseline behaviour)    │
                  └──────────────────┬─────────────────────┘
                                     │
                  ┌──────────────────▼─────────────────────┐
                  │  EXAMINE THE RULE (why did it fire?    │
                  │  which term matched?)                  │
                  └──────────────────┬─────────────────────┘
                                     │
                  ┌──────────────────▼─────────────────────┐
                  │  VERDICT — True Positive or False      │
                  │  Positive?                             │
                  └──────────────────┬─────────────────────┘
                                     │
                  ┌──────────────────▼─────────────────────┐
                  │  ACTION — TP → contain (isolate host)  │
                  │         FP → tune the rule             │
                  └────────────────────────────────────────┘
```

Every alert in a real SOC runs through this loop, regardless of which SIEM product is used. Fluency here transfers between Splunk, Sentinel, QRadar, and ELK — the products differ in interface, but the loop is the same.

---

## The Investigation — A Suspected CryptoMiner

### Stage 1 — Dashboard at Baseline

The SIEM dashboard showed a healthy estate profile before the suspicious activity was triggered:

| Widget | Observation |
| --- | --- |
| Top 10 event codes | 4625 (failed logon) at 38.3%, then EventID 1, 3, 11, 4798, 4624, 4672 |
| Top 5 RDP sources | `102.152.62.75` (40%), `196.186.67.90` (30%), others |
| Number of events | ~3,700–4,000 per month (Apr–Jun) |
| MITRE ATT&CK distribution | Command-and-Control (45.8%), T1036 Masquerading (29.2%), T1049, T1016 |
| Top process counts | `chrome.exe` 9,007, `cmd.exe` 3,187, `svchost.exe` 1,636 |
| IP+Port destination | `169.254.1.x` (90 connections), `168.63.12.x` (80), tail of internal IPs |

This baseline matters because it tells the analyst what *normal* looks like. The cryptominer alert fired against a backdrop of otherwise expected enterprise activity — that contrast is what makes the new red bar (`cudominer.exe`, count: 1) immediately stand out.

### Stage 2 — Alert Fires

Clicking **Start Suspicious Activity** triggered the alert popup:

> ⚠️ *Potential CryptoMiner Activity Observed, find and click on the triggered event to show the details.*

Simultaneously, a new red entry appeared in the Process Name list: `cudominer.exe` with count 1.

#### Q1 — Which process caused the alert?

**Answer:** `cudominer.exe`

**Found:** Process Name list on the dashboard (new red bar).

The name itself is informative. "CUDA Miner" is a category of cryptocurrency mining software designed to run on NVIDIA GPUs using their CUDA parallel-compute framework. The presence of `cudominer.exe` on a corporate endpoint is almost always one of:

- Unauthorised cryptocurrency mining by an insider (most common in this kind of scenario)
- A commodity cryptojacking malware payload (e.g. XMRig variants disguised with various names)
- Legitimate research / GPU benchmarking that should be in an asset-inventory exception list

In a SIEM context, the *name* is signal — but the *user, host, and execution path* are what determine the verdict.

### Stage 3 — Find the Underlying Event

Clicking **Find Event** drilled from the alert into the events table:

| SourceModule | HostName | UserName | ProcessName | Opcode | Source |
| --- | --- | --- | --- | --- | --- |
| eventlog | HR_01 | haroon | `C:\Windows\System32\MicrosoftEdgeSH.exe` | Info | Win... |
| eventlog | Admin_02 | Moin | `C:\Program Files (x86)\java\jre1.8.0_181\bin\javaws.exe` | Info | Win... |
| eventlog | IT_01 | Bell | `C:\Python3\python.exe` | Info | Win... |
| **eventlog** | **HR_02** | **Chris** | **`C:\Users\Chris\temp\cudominer.exe`** | **Info** | **Win...** |
| eventlog | IT_02 | Amelia | `C:\Program Files\QuickTime\quicktime.exe` | Info | Win... |
| eventlog | HR_03 | Daina | `C:\Program Files\Quicken\qw.exe` | Info | Win... |

#### Q2 — Identify the user responsible for the process execution

**Answer:** `chris`

**Found:** UserName column on the matching event row.

#### Q3 — What is the hostname of the suspect user?

**Answer:** `HR_02`

**Found:** HostName column on the same event row.

### Stage 4 — Enrich with Context

Three pieces of context, all from the events table row, paint the verdict picture:

| Context | Observed Value | Interpretation |
| --- | --- | --- |
| User | `chris` | Single end-user identity, not a service account |
| Host | `HR_02` | An HR department endpoint — no GPU compute business case |
| Process path | `C:\Users\Chris\temp\cudominer.exe` | Running from a user's *temp* folder — classic location for unauthorised or malicious software |

Compare this row to its neighbours in the table: every other process is in a normal program directory (`Program Files`, `Windows\System32`, `Python3`). The `temp` folder is the outlier — and the *user-named temp folder* doubly so, because most legitimate temp activity uses `%TEMP%` (i.e. `C:\Users\<name>\AppData\Local\Temp`), not a directory the user has deliberately created.

The combination — **HR department user, user-created temp folder, GPU mining binary** — gives the analyst enough to confidently call this a true positive even before examining the rule.

### Stage 5 — Examine the Correlation Rule

The SIEM rule editor showed the rule that fired:

```
Alert "Potential CryptoMiner Activity" 
  If EventID = 4688 
  AND Log_Source = WindowsEventLogs 
  AND ProcessName = (*miner* OR *crypt*)
```

#### Q4 — Which term matched the rule?

**Answer:** `miner`

**Found:** the wildcard `*miner*` in the rule definition matches `cudominer.exe`.

#### Reading the Rule Like an Analyst

This is a small but well-formed correlation rule. Three clauses, all required:

1. **`EventID = 4688`** — Windows Security log "A new process has been created" event. This is the canonical event for process-execution-based detections on Windows
2. **`Log_Source = WindowsEventLogs`** — scoping clause; restricts the rule to logs ingested from Windows Event Log forwarders rather than, say, application logs
3. **`ProcessName = (*miner* OR *crypt*)`** — the actual detection signature; wildcards on either side of "miner" or "crypt" let the rule catch `cudominer.exe`, `xmrig-miner.exe`, `cryptolocker.exe`, etc.

#### Honest Critique — Why This Rule Would Need Tuning in Production

Reading the rule like a detection engineer, two things stand out:

- **The `*crypt*` wildcard would generate false positives.** Legitimate binaries containing "crypt" exist — for example, `bitlocker.exe` doesn't, but tools like `crypttool.exe`, `truecrypt.exe`, `veracrypt.exe`, and several encryption SDK binaries do. In production this rule would need an allowlist of known-good "crypt" binaries
- **`*miner*` is cleaner** because almost no legitimate enterprise binary contains the substring "miner" — but even here, names like `examiner.exe` or `determiner.exe` (hypothetical but plausible) would trigger

This is the kind of analysis a SOC engineer does after every detection: *does this rule have a high enough true-positive rate to stay in production, or does it need an allowlist before it generates analyst fatigue?* (See the SOC Metrics room earlier in this path — high FP rate is the #1 cause of L1 burnout.)

### Stage 6 — Verdict

#### Q5 — Which option best represents the event? (False Positive / True Positive)

**Answer:** True Positive

The evidence chain is airtight:

| Evidence | Verdict Direction |
| --- | --- |
| Process name `cudominer.exe` | → Malicious (GPU mining binary) |
| Execution path `C:\Users\Chris\temp\` | → Malicious (user-created temp folder, non-standard) |
| User department (HR) | → Malicious (no GPU compute business case) |
| EventID 4688 with this combination | → Confirmed execution, not just dropped file |
| No conflicting threat intel | → No legitimate-tool flip |

This is the inverse of the EDR room's DESKTOP-DEV01 case — there, every behavioural indicator looked malicious but threat intel flipped the verdict to benign. Here, every indicator points the same direction, with no exonerating context. **True Positive is the only defensible verdict.**

### Stage 7 — Action

#### Q6 — Select the right action; what is the flag?

**Answer:** `THM{000_SIEM_INTRO}`

**Action selected:** *True positive and isolate the host*

Selecting this action triggered the SIEM (or its connected SOAR layer) to automatically isolate `HR_02` from the network — cutting it off from internal resources and external destinations while preserving forensic state for investigators. This is the canonical TP response for process-execution-based detections on endpoint hosts.

#### Why Isolation Matters

Host isolation is the SOC's strongest reversible response action. It:

- Stops further lateral movement from the compromised host
- Prevents data exfiltration over the network
- Cuts off C2 communications if the binary is part of a wider campaign
- Preserves the host in its current state for incident responders to image and examine
- Is reversible — once IR completes, the host can be reconnected after remediation

Compared to the alternatives (do nothing, kill the process, wipe the host), isolation is the highest-leverage early action.

---

## Cross-Investigation Lessons Carried Forward

1. **SIEM is the investigation surface; EDR is the depth-of-visibility surface.** Both rooms (Intro to EDR and Intro to SIEM) teach the same investigation loop but from different starting points. A real SOC pivots between them constantly.
2. **The investigation loop is product-agnostic.** Whether the SIEM is Splunk, Sentinel, QRadar, ELK, or LogRhythm, the loop is identical: alert → event → enrich → rule → verdict → action. Learn the loop, the products follow.
3. **Read every rule before forming a verdict.** Q4 in this lab forces the analyst to read the actual rule that fired. This is a habit worth cultivating — many junior analysts close alerts without ever reading the rule definition, which leaves them unable to spot tuning gaps or explain decisions to senior analysts.
4. **Context flips verdicts (or confirms them).** The EDR room taught this with the UpdateAgent.exe case (behaviour looked malicious, threat intel flipped to benign). This room taught the inverse — context *confirmed* malice. Both directions matter; both depend on context-enrichment being part of the investigation loop.
5. **TP and FP are not equal-weight decisions.** A TP missed becomes an incident; an FP escalated wastes L2 time. The action interface in this SIEM made the asymmetry visible — TP triggers containment, FP triggers tuning. Knowing which to choose, with confidence, is the entire job of an L1 analyst.
6. **Process execution paths tell stories.** `C:\Windows\System32\*` is normal. `C:\Program Files\*` is normal. `C:\Users\<name>\temp\*` is *not* normal. Building intuition for which paths are typical versus suspicious is one of the fastest ways an L1 analyst becomes effective.

---

## SIEM vs EDR — The Combined Picture

Now that both rooms are complete, the comparison is concrete:

| Dimension | EDR | SIEM |
| --- | --- | --- |
| Scope | One endpoint at a time | The entire estate, correlated |
| Depth on a single host | Very high (process tree, command lines, file activity, registry, memory) | Medium (whatever the endpoint forwards) |
| Cross-source correlation | Limited (endpoint-internal only) | This is its whole job |
| Primary view | Process chain + IOC table per detection | Aggregated dashboards + event search |
| Best at | "What exactly happened on this host?" | "What's happening across our environment?" |
| Investigation start point | A behavioural detection on an endpoint | A correlation rule firing across log sources |
| Response capabilities | Native (process kill, host isolate, file quarantine) | Often via integrated SOAR (network isolate, account disable, ticket) |

In practice: alerts often *originate* in one or the other, but resolving them well almost always requires both. A SIEM alert about a suspicious process gets richer detail in the EDR. An EDR detection about a single endpoint gets context (other affected hosts, lateral movement attempts) in the SIEM.

---

## Tools, Concepts & Frameworks Referenced

- **SIEM** — Splunk, Microsoft Sentinel, IBM QRadar, Elastic Security (ELK), LogRhythm, ArcSight
- **Log sources** — Windows Event Logs, Linux syslog, firewall logs, IDS/IPS, web/DNS server logs, AD, cloud (Azure/AWS/GCP), EDR telemetry, email gateway
- **Windows Event ID 4688** — A new process has been created (the canonical process-execution event)
- **Correlation rules** — boolean logic across normalised event fields (EventID, Log_Source, ProcessName, etc.) with wildcard matching
- **MITRE ATT&CK** — T1036 (Masquerading), T1049 (System Network Connections Discovery), T1016 (System Network Configuration Discovery), and the broader Command-and-Control tactic dominated the lab's MITRE distribution chart
- **TP/FP action loop** — True Positive triggers containment; False Positive triggers detection-rule tuning

---

## Summary of Questions, Answers & Source

| # | Question | Answer | Where Found in SIEM |
| --- | --- | --- | --- |
| 1 | Which process caused the alert? | `cudominer.exe` | Dashboard → Process Name list (new red bar after alert) |
| 2 | Identify the user responsible | `chris` | Events table — UserName column |
| 3 | Hostname of the suspect user | `HR_02` | Events table — HostName column |
| 4 | Term that matched the rule | `miner` | Rule definition viewer |
| 5 | Best representation of the event | True Positive | Verdict question |
| 6 | The flag after selecting the right action | `THM{000_SIEM_INTRO}` | Action interface → "True positive and isolate the host" → Save Action |

All 6 answered correctly on first attempt. Room completed at 100%.

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*