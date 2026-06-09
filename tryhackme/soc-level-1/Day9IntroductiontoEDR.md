# 🛡️ Introduction to EDR – TryHackMe

**Path:** SOC Level 1 · Core SOC Solutions

## 📅 Date
9 June 2026

Day 9

## 📚 What I Learned

- **What an EDR (Endpoint Detection and Response) is** — a security solution that continuously monitors endpoints, detects advanced threats, and provides built-in response actions like host isolation, process kill, and file quarantine
- The **difference between EDR and traditional Antivirus** — AV is signature-based and reacts to known files; EDR is behaviour-based and reacts to the *story* of an attack (process chains, parent-child relationships, command-line arguments, network behaviour)
- **EDR architecture** — lightweight endpoint agent collecting telemetry → central console (cloud or on-prem) for aggregation, correlation, and analyst investigation
- **Telemetry types collected** — process creation events, file operations, registry modifications, network connections, command-line arguments, user/account context, hash values
- **Detection & response capabilities** — detections enriched with Threat Intel tags, MITRE ATT&CK technique IDs, IOC tables, process chain visualisations, and automated containment actions (Quarantined, Flagged, Blocked, Logged)
- The **alert detail anatomy** in a real EDR console — Summary, Process Info, IOC/Indicators, and Actions/Response tabs — and how to pivot between them during triage
- That **context flips verdicts**: behavioural indicators alone can scream "malicious", but Threat Intel + asset inventory enrichment can turn a high-confidence-looking detection into a benign business-tool false positive

## 🛠️ Lab — Investigating a Multi-Stage Incident at TECH THM

Played a SOC analyst at TECH THM with access to the EDR console. Four detections required triage across three hosts and three users. All five questions answered correctly using only the information visible in the EDR.

### Dashboard Overview Observed

- **Active Hosts:** 51 · **New Detections:** 4 · **Detection Sources:** Anomaly, Behavior, Threat Intel
- **Recent Detections:**
  - DESKTOP-HR01 → Initial Access via Malicious Office Document (High)
  - WIN-ENG-LAPTOP03 → Credential Dumping via LSASS Memory Access (High)
  - DESKTOP-DEV01 → Execution from AppData Directory (Medium)
  - DESKTOP-UATSERVER → Suspicious Persistence via Scheduled Task (Medium)
- **Detections by Tactics (MITRE ATT&CK)** chart visualising Execution, Defense Evasion, Persistence, and Command and Control activity over time

### Question & Answer Walkthrough

**Q1: Which tool was launched by CMD.exe to download the payload on DESKTOP-HR01?**

✅ **CURL.exe**

Found under the **Summary** tab of the *Initial Access via Malicious Office Document* detection. Attack chain: `invoice.docm` opened via `WINWORD.EXE` → spawned `CMD.exe` → launched `cURL` to download a payload from an external domain. Classic Office macro → command shell → ingress tool transfer chain (MITRE T1059 + T1105).

**Q2: What is the absolute path to the downloaded malware on the DESKTOP-HR01 machine?**

✅ **C:\Users\Public\install.exe**

Found under the **IOC/Indicators** tab. The Public folder is a classic staging location — low permissions, accessible to all users, often used by attackers because it doesn't trigger user-specific monitoring. EDR action: **Quarantined**.

**Q3: What is the absolute path to the suspicious syncsvc.exe on the WIN-ENG-LAPTOP03 machine?**

✅ **C:\Users\haris.khan\AppData\Local\Temp\syncsvc.exe**

Found under the **IOC/Indicators** tab of the *Credential Dumping via LSASS Memory Access* detection. AppData\Local\Temp is another classic malware location. EDR action: **Flagged, Quarantined**.

**Q4: On which URL was the exfiltration attempt being made on WIN-ENG-LAPTOP03?**

✅ **https://files-wetransfer.com/upload/session/ab12cd34ef56/dump_2025.dmp**

Found under the **Process Info** tab → *Network Activity → Attempted exfil to*. The attacker used a typosquatted WeTransfer-lookalike domain (`files-wetransfer.com` instead of legitimate `wetransfer.com`) to exfiltrate the LSASS memory dump. EDR status: **Blocked by EDR**. ATT&CK ID: **T1003.001** (LSASS Memory).

**Q5: What was UpdateAgent.exe labelled by Threat Intel on DESKTOP-DEV01?**

✅ **Known internal IT utility tool**

Found under the **Process Info** tab → *Threat Intel* field. This is the most educational detection in the lab — every behavioural indicator looked malicious:

- Launched from `AppData\Roaming` (classic malware location)
- Spawned by `explorer.exe`
- Unsigned binary
- Made outbound HTTP connection to `10.10.20.5:8080`

But the Threat Intel enrichment flipped the verdict — UpdateAgent.exe is a legitimate internal IT tool, not malware. Without that context, this would have been escalated as a high-confidence true positive. Lesson: **behaviour alone is not enough — context is what turns telemetry into a correct verdict**.

## 🔐 Why It Matters

- EDR is the **primary investigation tool** for a SOC L1 analyst. Almost every alert triage in real-world SOC work involves pivoting between Summary, Process Info, IOCs, and Response tabs in an EDR console
- Knowing how to read process chains, IOC tables, and Threat Intel enrichment is the difference between a False Positive escalation and a correct verdict
- EDR products like **CrowdStrike Falcon, Microsoft Defender for Endpoint, SentinelOne, Carbon Black, and Cybereason** are listed by name in nearly every UK SOC job advert — fluency with the conceptual model (regardless of vendor) is core to interview readiness

## ❓ One Thing I Didn't Fully Understand

All five questions correct on first attempt. The most thought-provoking part wasn't the questions themselves — it was the DESKTOP-DEV01 detection. Every behavioural signal said "malicious", but the Threat Intel tag said "known internal tool". That single flip captures the entire reason EDR exists: **enriched context turns alerts into verdicts**. Without it, analysts either over-escalate (and burn out the SOC) or under-escalate (and miss real attacks).

---

**Next up:** Continuing through Core SOC Solutions