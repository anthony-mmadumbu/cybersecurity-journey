# Elastic Stack: The Basics — Portfolio Write-up

**Room:** Elastic Stack: The Basics
**Path:** SOC Level 1 · Cyber Defence Frameworks → SIEM Tooling
**Date completed:** 14 June 2026 (Day 12)
**Completion:** 100% — all questions answered correctly on first submission

---

## 1. Overview

This room introduced the **Elastic Stack (ELK)** — Elasticsearch, Logstash, Kibana, and Beats — as a Security Information and Event Management (SIEM) platform. The practical exercises used a dataset of 2,861 VPN connection events spanning 31 Dec 2021 to 2 Feb 2022, simulating a realistic SOC log-analysis scenario for a fictitious company "CyberT".

The hands-on objectives were to:
1. Navigate Kibana's Discover view, select indices, and apply time-based filters.
2. Use KQL (Kibana Query Language) to filter and search log data.
3. Build visualisations and saved tables in the Visualize Library.
4. Investigate user activity patterns, including a terminated-user scenario.
5. Identify anomalous spikes and failed-authentication patterns.

This write-up walks through each task as I'd present it in a SOC investigation, with the reasoning behind each step rather than just the answers.

---

## 2. The Elastic Stack in Context

| Component | Role | SOC equivalent |
|-----------|------|----------------|
| **Elasticsearch** | Distributed search and analytics engine — stores log data as JSON documents in indices | Comparable to Splunk's indexer tier |
| **Logstash** | Server-side data pipeline — ingests, parses, transforms, and forwards data | Comparable to Splunk's heavy forwarder / Cribl |
| **Kibana** | Web UI for searching, visualising, and dashboarding | Comparable to Splunk Web / Search & Reporting |
| **Beats** | Lightweight agents (Filebeat, Winlogbeat, Packetbeat, Auditbeat) that ship data from endpoints | Comparable to Splunk Universal Forwarder |

In production, Elastic Stack is the backbone of **Elastic Security** (Elastic's own XDR offering), **Wazuh** (open-source SIEM), and **AlienVault USM Anywhere**. Knowing how to query Elasticsearch indices via Kibana is a directly transferable Tier 1 skill.

---

## 3. Investigation Walkthrough

### Task 1 — Establish the dataset baseline

> *Select the index `vpn_connections` and filter from 31st December 2021 to 2nd Feb 2022. How many hits are returned?*

**Approach:** In Discover, switched the data view to `vpn_connections` and set the absolute time range to `Dec 31, 2021 @ 10:29:48.05 → Feb 2, 2022 @ 10:30:22.19`.

**Answer:** **2,861 hits**

**SOC relevance:** The first move in any log investigation is to confirm the dataset scope — knowing the total event volume frames every subsequent percentage and ratio.

---

### Task 2 — Identify the most active source IP

> *Which IP address has the maximum number of connections?*

**Approach:** In the Available fields panel, expanded the `Source_ip` field. Kibana automatically computes the Top 5 values for the field over the current dataset.

**Answer:** **238.163.231.224** (3.2% — the single highest value)

**SOC relevance:** This is exactly the kind of pivot a Tier 1 analyst uses to spot the noisiest IP in a log set — useful both for investigating potential abuse and for ruling out benign high-volume infrastructure (load balancers, proxies, NAT egress points).

---

### Task 3 — Identify the top user by traffic

> *Which user is responsible for the overall maximum traffic?*

**Approach:** Expanded the `UserName` field summary.

**Answer:** **James** (4.0%)

**SOC relevance:** In a real environment, the heaviest user is rarely the threat — but the heaviest *unexpected* user often is. Establishing the top-talker baseline lets you spot deviations from it later.

---

### Task 4 — Pivot on a single user

> *Apply Filter on UserName Emanda; which SourceIP has max hits?*

**Approach:** Added a filter `UserName: Emanda` (the dataset narrowed from 2,861 to 56 hits). Re-examined the `Source_ip` field summary against this filtered subset.

**Answer:** **107.14.1.247** (53.6% of Emanda's connections)

**SOC relevance:** This demonstrates **pivot-based investigation** — once you have an indicator (a user), you pivot on it to see what other indicators it's associated with (their dominant source IPs). In real incident response, this is how you go from a single alert to an attack chain.

---

### Task 5 — Investigate a time-bound spike

> *On 11th Jan, which IP caused the spike observed in the time chart?*

**Approach:** Adjusted the time range to `Jan 11, 2022 @ 00:00 → Jan 11, 2022 @ 23:30`. The dataset narrowed to 283 hits with a clearly visible cluster in the early hours. Expanded `Source_ip` against this window.

**Answer:** **172.201.60.191** (97.2% of that day's traffic — clearly the anomaly)

**SOC relevance:** Time-bound investigation is fundamental to alert triage. When a SIEM fires on an anomalous event count, the analyst zooms into the relevant window and looks for the single field value that's distorting the chart — exactly this workflow.

---

### Task 6 — Build a conditional visualisation

> *How many connections were observed from IP 238.163.231.224, excluding the New York state?*

**Approach:** Switched to Visualize Library, built a **Bar vertical stacked** chart with:
- **Horizontal axis:** Top values of `source_state`
- **Vertical axis:** Count of records
- **Filters:** `Source_ip: 238.163.231.224` AND `NOT source_state: New York`

**Answer:** **48** — all from Michigan.

**SOC relevance:** Using NOT logic to exclude known/expected activity is a daily SOC technique. Anomalies often surface most clearly once you filter out the noise floor of legitimate traffic.

---

### Task 7 — Save a custom table view

> *Create a table with the fields IP, UserName, Source_Country and save.*

**Approach:** Added `Source_ip`, `UserName`, and `Source_Country` as columns in Discover, then used **Save** to persist the view as `Table001`. Confirmed it appeared under Recently viewed.

**Answer:** No answer needed — saved view confirmed.

**SOC relevance:** Saved searches and tables become reusable triage views in a SOC. A well-named library of queries is half the speed advantage senior analysts have over juniors.

---

### Task 8 — Multi-condition KQL query

> *Create a search query to filter the logs where Source_Country is the United States and show logs from User James or Albert. How many records were returned?*

**Approach:** Wrote the KQL:
```
Source_Country: "United States" and UserName: "James" or "Albert"
```

**Answer:** **161 records**

> **Critical reflection on operator precedence:** The query I wrote (and TryHackMe accepted) parses as `(Source_Country: "United States" AND UserName: "James") OR "Albert"` — KQL's `AND` binds tighter than `OR`. A strictly correct query for the *intended* logic ("US country AND (James OR Albert)") would be:
> ```
> Source_Country: "United States" and (UserName: "James" or UserName: "Albert")
> ```
> This is a teachable moment: even when a SIEM accepts a query, the analyst is responsible for verifying that the **logic matches the intent**. In a real investigation, getting precedence wrong can either flood you with false positives or — worse — silently hide the events you were hunting for.

**SOC relevance:** Multi-condition filtering is the bread-and-butter of SIEM work. Understanding boolean precedence isn't optional.

---

### Task 9 — Insider/credential-abuse detection (the most important question in the room)

> *A user Johny Brown was terminated on the 1st of January, 2022. Create a search query to determine how many times a VPN connection was observed after his termination.*

**Approach:** KQL filter `UserName: "Johny Brown"` over the full time range. Kibana returned a single hit dated **Jan 7, 2022 @ 02:28:47** — six days after his termination.

**Answer:** **1 connection**

**SOC relevance — this is the highest-signal finding in the dataset:**
- **MITRE ATT&CK mapping:** This pattern aligns with **T1078 — Valid Accounts**, and specifically **T1078.004 — Cloud Accounts** if the VPN is cloud-fronted. Use of credentials from a terminated employee is one of the most common initial-access vectors in real breaches (notably the 2021 Colonial Pipeline incident, where the breached account was a dormant VPN account without MFA).
- **NIST mapping:** This is a failure of the **NIST SP 800-53 control AC-2 (Account Management)** — specifically AC-2(3), "Disable Inactive Accounts," and AC-2(13), "Disable Accounts for High-Risk Individuals." Terminated-user accounts must be disabled at offboarding, not at a later cleanup cycle.
- **Detection design:** In production, this would be a **scheduled correlation rule** in the SIEM — joining HR's offboarding feed against authentication logs and alerting on any auth from a terminated UPN. Most mature SOCs run this as a daily Joiner/Mover/Leaver (JML) detection.

A single hit might look unremarkable in a 2,861-event dataset, but in a real SOC this would be a **P1 incident** — credential compromise or unauthorised access by a former employee.

---

### Task 10 — Identify the worst-offending failed-auth user

> *Which user was observed with the greatest number of failed attempts?*

**Approach:** Built a donut (Pie) visualisation in the Visualize Library with:
- **Slice by:** Top values of `UserName` (inner ring), Top values of `action` (outer ring)
- **Size by:** Count of records

This produced a layered pie where the outer ring split each user's activity into `built`, `teardown`, and `failed`. Simon's slice was visibly dominated by `failed`.

**Answer:** **Simon**

**SOC relevance:** Nested aggregations are the visual analyst's equivalent of `GROUP BY user, action ORDER BY count(*)`. Spotting one user dominating a single action category is a high-signal anomaly.

---

### Task 11 — Quantify failed attempts in January

> *How many wrong VPN connection attempts were observed in January?*

**Approach:** Built a table visualisation with rows for `action`, `UserName`, `EventTime` (per 12 hours), and the metric Count of records, filtered to `action: failed`.

**Answer:** **274 failed attempts** — concentrated on 11 Jan 2022, all attributed to Simon.

**SOC relevance — this is the second high-signal finding:**
- **MITRE ATT&CK mapping:** **T1110 — Brute Force**, specifically **T1110.001 (Password Guessing)** or **T1110.003 (Password Spraying)** depending on the source-IP and target-account spread.
- **Threshold consideration:** 274 failures from one user account in a single day is well above any reasonable user-error baseline. Most production SOCs would alert at **10–25 failures per account per hour**.
- **Likely scenarios to investigate:**
  1. **Active brute-force attack** against Simon's account — needs immediate account lockout, password reset, and source-IP geolocation.
  2. **Compromised credential** being tested by an attacker — needs forced re-authentication and a check for any successful auth following the failures.
  3. **Misconfigured client/service account** repeatedly failing with a cached old password — benign but operationally important to fix.
- **Detection design:** A baseline detection rule would be: `count(action="failed") by UserName > 25 within 1 hour` → SIEM alert. The TryHackMe data would have triggered such a rule decisively.

---

## 4. Skills Demonstrated

| Skill | Evidence |
|-------|----------|
| Kibana / Discover navigation | Time-range filtering, index selection, field summary use |
| KQL query authoring | Multi-condition queries with `AND`, `OR`, `NOT`, including reflection on operator precedence |
| Pivot-based investigation | Filtering by user → examining associated source IPs |
| Time-bound anomaly investigation | Narrowed to single day to isolate spike-causing IP |
| Visualisation design | Bar vertical stacked, donut/pie, multi-row table |
| Saved-search workflow | Persisted `Table001` for reuse |
| Threat framing | MITRE ATT&CK mapping (T1078, T1110) and NIST 800-53 mapping (AC-2) for findings |

---

## 5. Key Findings — How I'd Present These in a Real SOC Brief

1. **Confirmed insider/credential-abuse indicator:** Terminated user Johny Brown authenticated via VPN six days after offboarding. **Recommended action:** P1 incident response — account disable verification, full session audit for Jan 7, source-IP geolocation, HR confirmation of termination date.

2. **Probable brute-force or credential-stuffing attack on user Simon:** 274 failed VPN authentications in January, clustered on 11 Jan. **Recommended action:** P2 — force password reset, check for any successful auths from same source IPs immediately following the failures, enable account lockout policy if not already in place, enforce MFA.

3. **Anomalous traffic source:** IP `172.201.60.191` was responsible for 97.2% of all VPN connections on 11 Jan 2022 — a dramatic single-day spike from a single IP. **Recommended action:** P3 — investigate the IP's ASN, geolocation, and threat-intel reputation; check whether the connections were legitimate (e.g. a single user on a high-uptime device) or scripted.

---

## 6. Reflection

This room confirmed that the underlying logic of log investigation is the same regardless of SIEM — Splunk's SPL and Elastic's KQL are different syntaxes for the same conceptual operations (filter, group, count, pivot, time-bound). The Pyramid of Pain framing from earlier rooms also held: the IP-based findings here (Tasks 5, 6, 11) sit low on the pyramid, while the **behavioural** findings (terminated-user authentication, single-user failure burst) sit higher and are far more durable as detection logic.

The one weakness I'm aware of: I haven't yet built my own KQL queries from scratch in a fresh environment — every query in this room had heavy scaffolding from the prompts. My next priority is to load logs into a personal Elastic instance and write queries against arbitrary investigation questions without prompt guidance.

---

**Tags:** `#elastic-stack` `#elk` `#kibana` `#kql` `#siem` `#log-analysis` `#vpn-logs` `#mitre-attack` `#t1110` `#t1078` `#nist-800-53` `#soc-tier-1`