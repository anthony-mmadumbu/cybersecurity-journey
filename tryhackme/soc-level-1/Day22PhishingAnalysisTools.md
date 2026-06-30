# 🛡️ Phishing Analysis Tools – TryHackMe

## 📅 Date
Day 22 · 30 June 2026

## 📚 What I Learned (3–5 lines max)
- **The phishing analyst's toolkit** — Talos Reputation Center for domain reputation, ANY.RUN for sandbox detonation, VirusTotal for hash and URL lookups, MXToolbox for header analysis, AbuseIPDB and Spamhaus for IP reputation.
- **The artefact extraction workflow** — sender IP, sender domain, `Return-Path:`, attachment hashes (MD5/SHA1/SHA256), URLs, tracking pixels — all logged as IOCs for ticket documentation and detection-rule creation.
- **Sandbox dynamic analysis** — observing process trees, network connections, DNS requests, and registry modifications to identify malicious behaviour that **static signature engines miss entirely**.
- **Cross-tool correlation** — pivoting from ANY.RUN MD5 to VirusTotal SHA256, mapping flagged IPs against Talos reputation, and building a complete IOC picture from multiple sources rather than trusting any single tool.

## 🛠️ What I Did
- Worked through a Talos Reputation Center warm-up exercise looking up `malware-test.com` (content category: Computer Security).
- Investigated **three real-world phishing case scenarios** as a Tier 1 SOC analyst:
  - **Case 1 — Phish3Case1.eml**: Netflix credential harvesting attack (spoofed `From:`, mismatched `Return-Path:`, SPF failure, `t.co` shortened URL).
  - **Case 2 — Payment-updateid.pdf**: Weaponised PDF with AcroRd32.exe spawning malicious network connections to 2.16.107.24, with svchost.exe flagged as Potentially Bad Traffic.
  - **Case 3 — CBJ200620039539.xlsx**: Malicious Excel attachment exploiting **CVE-2017-11882** (Microsoft Equation Editor buffer overflow), contacting `biz9holdings.com` and `findresults.site`.
- All questions across all three cases answered correctly. Detailed walkthrough in the companion portfolio piece (`Day22_Phishing_Analysis_Tools_Portfolio.md`).

## 🔐 Why It Matters
- This is **the first room in the SOC L1 path where I actually did production-style phishing triage** — opening files in sandboxes, extracting IOCs, correlating across tools, and arriving at verdicts. The workflow demonstrated here is the daily reality of a Tier 1 analyst at any MSSP.
- Knowing **which tool to reach for in which scenario** (header analyser for sender authentication, sandbox for attachment behaviour, reputation lookup for IP/domain context) is what separates an analyst who can articulate a triage methodology from one who just answers questions about tools without method.

## ❓ One Thing I Didn't Fully Understand
- The exact **relationship between MD5, SHA1, and SHA256 hashes** in the production workflow. The Case 2 investigation pivoted from MD5 (provided by ANY.RUN) to SHA256 (via VirusTotal) — but production SIEM and EDR alerting usually standardises on SHA256. Worth confirming whether MD5 pivots are common in real triage or just a quirk of how the tools display data, since MD5 is cryptographically broken and could theoretically produce collisions.