# 🛡️ Elastic Stack: The Basics – TryHackMe

## 📅 Date
Day 12 · 14 June 2026

## 📚 What I Learned (3–5 lines max)
- The Elastic Stack components: **Elasticsearch** (storage and search engine), **Logstash** (data ingestion and parsing), **Kibana** (visualisation and UI), and **Beats** (lightweight data shippers).
- How to navigate Kibana's **Discover** view, select an index, apply time filters, and read the field summary panel (Top 5 values, percentages, exists-in counts).
- Writing **KQL** queries with boolean logic (`and`, `or`, `not`) to combine multiple field conditions.
- Building saved tables and bar/pie chart visualisations in the **Visualize Library** to break log data down by fields like `source_state`, `UserName`, and `action`.

## 🛠️ What I Did
- Queried the `vpn_connections` index across Dec 31, 2021 – Feb 2, 2022 (2,861 records).
- Identified the busiest source IP (`238.163.231.224`) and busiest user (`James`) using field summaries.
- Used the `UserName: Emanda` filter to surface her dominant source IP (`107.14.1.247` — 53.6% of her traffic).
- Narrowed the time window to 11 Jan 2022 and identified the IP responsible for the day's spike (`172.201.60.191` — 97.2% of that day's connections).
- Built a bar visualisation excluding New York state to count connections from `238.163.231.224` (48 — all from Michigan).
- Wrote a KQL query for US-based connections by James or Albert (161 records).
- Investigated a terminated user (Johny Brown, terminated 1 Jan 2022) — found **1 VPN connection on 7 Jan**, six days after termination.
- Built a pie chart of actions broken down by user and a table filtered to `action: failed` — confirmed **Simon** had the most failures (274 in January).

## 🔐 Why It Matters
- The Elastic Stack underpins many real-world SOC platforms (Elastic Security, Wazuh, AlienVault USM) — being able to pivot through logs in Kibana is core Tier 1 work.
- The Johny Brown finding is a textbook insider-threat/credential-abuse indicator. Authentication from a terminated user account is a high-confidence detection — joiner/mover/leaver hygiene gaps are one of the most common audit findings in real organisations.
- Surfacing failed authentication patterns is the foundation of detecting **brute-force (MITRE ATT&CK T1110)**, password spraying, and credential stuffing attacks. Simon's 274 failures in a single month would absolutely warrant a SOC investigation in production.

## ❓ One Thing I Didn't Fully Understand
- The trade-off between **Lens** (drag-and-drop visualisations) and the older **Visualize Library** — the room moved between both but didn't fully spell out when one is preferred over the other. I want to read up on this since it affects how I'd build dashboards in a real SOC.