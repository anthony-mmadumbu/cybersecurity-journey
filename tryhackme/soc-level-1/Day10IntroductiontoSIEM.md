# 🛡️ Introduction to SIEM – TryHackMe

**Path:** SOC Level 1 · Core SOC Solutions

## 📅 Date

Day 10 · Wednesday, 10 June 2026

## 📚 What I Learned

- **What a SIEM (Security Information and Event Management) is** — a centralised platform that collects logs from across the estate, normalises them into a common schema, correlates events across sources, and triggers alerts when activity matches predefined rules
- The **limitations of working with isolated logs** — endpoints, firewalls, servers, AD, and cloud platforms each produce their own log formats; without a SIEM, an analyst has to manually pivot between consoles and cannot correlate across sources at scale
- **Common log sources feeding a SIEM** — Windows Event Logs, Linux syslog, firewalls, IDS/IPS, web servers, DNS, Active Directory, cloud platforms (Azure / AWS / GCP), endpoint security tools (EDR), application logs
- **Log ingestion methods** — endpoint agents (the most common for Windows Event Logs), syslog forwarders, API connectors, log collectors / forwarders that aggregate before shipping to the SIEM
- **Correlation rules** — the heart of SIEM detection. A rule defines a condition (e.g. *EventID = 4688 AND ProcessName contains "miner" or "crypt"*) that, when matched by an incoming event, triggers an alert
- **The SIEM investigation loop** — Detect → Identify the event → Enrich with user/host/process context → Examine the rule that fired → Verdict (TP/FP) → Action (isolate or tune)
- **True Positive vs False Positive actioning** — TP actions trigger containment (e.g. isolate the host); FP actions feed back into detection engineering (tune the rule). Both close the investigation loop properly
- **Common SIEM products in industry** — Splunk (market leader), Microsoft Sentinel, IBM QRadar, Elastic Security (ELK), LogRhythm, ArcSight. Splunk and Sentinel are the two most commonly listed in UK SOC job adverts

## 🛠️ Lab — Investigating a CryptoMiner Alert in the SIEM

Played a SOC analyst with access to a SIEM dashboard. A suspicious activity was triggered, an alert fired, and I worked through the full investigation loop to a verdict and response action. All 6 questions answered correctly on first attempt and the room was completed at 100%.

### Dashboard Observations (Before the Alert Fired)

- **Top 10 event codes:** 4625 (failed logon) topping at 38.3%, then EventID 1 at 24.1%, then a tail of 3, 11, 4798, 4624, 4672 — typical Windows estate noise profile
- **Top 5 RDP connections:** by source IP — `102.152.62.75` (40%), `196.186.67.90` (30%), `102.152.51.213` (20%), `196.186.220.253`
- **Number of events:** ~3,700–4,000 per month (Apr–Jun)
- **MITRE ATT&CK distribution:** Technique=Command-and-Control (45.8%), T1036 Masquerading (29.2%), T1049 System Network Connections Discovery, T1016 System Network Configuration Discovery, plus "Info=Unknown Process"
- **Top process counts:** `chrome.exe` 9,007 · `cmd.exe` 3,187 · `svchost.exe` 1,636
- **IP+Port destination chart:** `169.254.1...` topping at 90 connections, `168.63.12...` at 80, then a tail of internal-looking IPs

### The Investigation Loop

After clicking **Start Suspicious Activity**, a new red entry appeared in the Process Name list (`cudominer.exe`, count: 1) and an alert popup fired: *"Potential CryptoMiner Activity Observed"*. I followed the alert through every stage of the investigation loop.

### Question & Answer Walkthrough

**Q1: After clicking on the Start Suspicious Activity button, which process caused the alert?**

✅ **cudominer.exe**

Spotted as a new red bar in the Process Name table immediately after the alert fired. The name itself is a giveaway — "cuda miner" is a NVIDIA GPU-based cryptocurrency mining tool, often abused for unauthorised mining on corporate hardware.

**Q2: Find the event that caused the alert and identify the user responsible for the process execution.**

✅ **chris**

Found in the events table — the row containing `cudominer.exe` had UserName = `Chris`. Pivoting from the alert to the underlying event is the analyst's first move: the alert tells you *something* happened; the event tells you *who, where, and what*.

**Q3: What is the hostname of the suspect user?**

✅ **HR_02**

Same events table — the row showed HostName = `HR_02`. Combined with Q2, we now have the *who* (Chris) and *where* (HR_02 endpoint). Notably, the user is in the **HR** department — meaning there's no business reason for them to be running GPU mining software, which strengthens the malicious verdict.

**Q4: Examine the rule and the suspicious process; which term matched the rule that caused the alert?**

✅ **miner**

The rule definition was visible in the SIEM:
```
Alert "Potential CryptoMiner Activity" 
  If EventID = 4688 
  AND Log_Source = WindowsEventLogs 
  AND ProcessName = (*miner* OR *crypt*)
```

The wildcard `*miner*` matched `cudominer.exe`. EventID 4688 is the Windows "A new process has been created" event — the canonical log source for process-execution-based detections.

**Q5: Which option best represents the event?**

✅ **True Positive**

All the evidence aligned: a clearly named mining binary, running from a user's temp folder (`C:\Users\Chris\temp\cudominer.exe` — a classic non-standard location for unauthorised software), executed by an HR user with no legitimate business case for GPU mining. Closing this as a False Positive would be a serious analyst error.

**Q6: Selecting the right ACTION will display the FLAG. What is the FLAG?**

✅ **THM{000_SIEM_INTRO}**

Selected **"True positive and isolate the host"** → Save Action → flag revealed.

The TP action triggered host isolation — the SIEM (or the SOAR layer connected to it) automatically removes the host from the network to prevent further damage while incident responders investigate.

## 🔐 Why It Matters

- SIEM is the **central nervous system of every modern SOC**. Almost every alert investigation starts in a SIEM, even if it pivots into EDR or threat intel from there
- The four log sources surfaced in this lab (Windows Event Logs, network device logs, EDR telemetry, IP/port destination data) reflect the **standard enterprise telemetry mix** an L1 analyst works with daily
- Knowing how to read a **correlation rule definition** is essential — it explains exactly why an alert fired, which is the foundation of both confident TP escalation and confident FP tuning
- The True Positive → Isolate vs False Positive → Tune cycle is **the actual workflow** a SOC engineer designs around. Closing alerts cleanly into one of these two paths is what keeps a SOC from drowning in noise

## ❓ One Thing I Didn't Fully Understand

All 6 questions correct on first attempt. The investigation loop was clear and clean — alert → event → enrich → rule → verdict → action. That cycle now feels repeatable for any future SIEM alert.

The most interesting part wasn't the questions but the rule itself: `ProcessName = (*miner* OR *crypt*)` is a classic example of a **simple-but-effective signature**. It's also exactly the kind of rule that produces both true positives (`cudominer.exe`) and false positives (`bitlocker.exe` would *also* match `*crypt*` — a tuning gap a real SOC would catch). Worth keeping in mind for future detection engineering work.

---

**Next up:** Pivoting into deeper investigation rooms (Splunk: The Basics, Investigating with Splunk, Investigating with ELK) to build hands-on SIEM querying skills