# Eviction — Threat-Informed Defence Against APT28 Write-up

**Lab Context:** TryHackMe — SOC Level 1 · Cyber Defence Frameworks · Eviction
**Date completed:** 21 June 2026 (Day 19)
**Role Played:** SOC Analyst (acting as "Sunny" at E-Corp)
**Focus:** Using the MITRE ATT&CK Navigator to translate threat intelligence about APT28 into a concrete list of techniques to hunt for inside a corporate network

---

## Executive Summary

This write-up documents a threat-informed defence exercise: receiving a classified intelligence report that **APT28 (Fancy Bear)** may be targeting our organisation, then using the **MITRE ATT&CK Navigator** to identify the specific techniques to search for in our own environment. The exercise is the daily reality of a SOC analyst supporting threat intelligence functions — the gap between *"a known adversary may be active"* and *"here is what to look for in our logs"* is bridged by ATT&CK.

All four threat-hunt questions were answered correctly on first attempt and the room was completed at 100%.

---

## Background — The Scenario

**E-Corp** manufactures rare earth metals for government and non-government clients — a profile that maps closely to APT28's known targeting priorities (defence, government, materials supply chain). The intelligence team has flagged APT28 as a likely threat, and the SOC has been asked to:

1. Identify the **techniques** APT28 is known to use
2. Determine whether any of those techniques have **already been observed** inside the network
3. **Stop the intrusion** if one is already underway

This is the canonical workflow that turns abstract threat intelligence into actionable detection and response.

---

## About APT28 (Fancy Bear)

APT28 is a Russian state-sponsored threat group (also tracked as **Fancy Bear**, **Sofacy**, and **Sednit**) attributed to the **GRU** (Russian military intelligence — Unit 26165). They are best known for operations against governments, defence sectors, NGOs, and energy/materials firms — making E-Corp's rare earth metals manufacturing a textbook target profile.

Notable historical operations attributed to APT28 include:
- 2016 DNC compromise (US election interference)
- 2015 Bundestag intrusion
- Targeting of the World Anti-Doping Agency
- Sustained operations against NATO members and Eastern European governments

Their toolkit is well-documented and their TTPs are extensively catalogued in MITRE ATT&CK under group ID **G0007**, making them an ideal subject for threat-informed defence exercises.

---

## The Tool — MITRE ATT&CK Navigator

The **ATT&CK Navigator** (`mitre-attack.github.io/attack-navigator`) is a browser-based interactive visualisation of the MITRE ATT&CK Matrix. It supports:

- **Layers** — overlays showing the techniques used by a specific threat group, software, or campaign
- **Group selection** — load APT28's known techniques as a single colour-coded layer
- **Comparison** — overlay multiple layers (e.g. your SOC's detection coverage vs APT28's TTPs) to identify gaps
- **Annotation** — add notes to specific techniques for hunt planning

For this exercise, the Navigator was loaded with the APT28 group layer, revealing every technique attributed to the group across the full Enterprise matrix.

---

## Question & Answer Walkthrough

### Q1 — Cross-tactic technique used for both Recon and Initial Access

> *"What is a technique used by the APT to both perform recon and gain initial access?"*

**Answer:** ✅ **Spearphishing Link**

This technique appears under two distinct ATT&CK tactics for APT28:

| Tactic | Technique ID | Purpose |
|---|---|---|
| Reconnaissance (TA0043) | T1598.003 — Phishing for Information: Spearphishing Link | Identify potential targets and gather intelligence (do they click? do they enter credentials?) |
| Initial Access (TA0001) | T1566.002 — Phishing: Spearphishing Link | Deliver the actual payload to gain a foothold |

**Why this matters:** A single delivery mechanism — a malicious link in a targeted email — serves both reconnaissance (does the target take the bait?) and initial access (here's the malware). For Sunny at E-Corp, this means email security telemetry needs to be reviewed twice: once for *any* click as a reconnaissance signal, and again for *successful* compromise as an initial access signal.

---

### Q2 — Resource Development account types

> *"Which accounts might the APT compromise while developing resources?"*

**Answer:** ✅ **Email accounts**

Under the **Resource Development** tactic (TA0042), APT28 is documented using **Compromise Accounts: Email Accounts (T1586.002)** to build attack infrastructure *before* engaging the target.

**Why this matters:** This is a critical pre-attack indicator. APT28 doesn't typically launch spearphishing campaigns from obviously-suspicious infrastructure — they compromise legitimate email accounts and use those as the sending platform. For E-Corp, this means:

- Phishing emails may arrive from real, legitimate-looking external email addresses
- Sender reputation and domain age checks won't catch them
- Authentication header analysis (SPF/DKIM/DMARC) becomes the front-line check
- Email gateway behavioural detection (unusual sender behaviour from previously-known-good accounts) is the higher-fidelity signal

---

### Q3 — User Execution sub-techniques to hunt for

> *"Which two techniques of User Execution should Sunny look out for?"*

**Answer:** ✅ **Malicious file** and **malicious link**

Under User Execution (T1204), APT28 is documented using two specific sub-techniques:

| Sub-technique | Technique ID | Detection focus |
|---|---|---|
| Malicious Link | T1204.001 | User clicks a link from a phishing email — proxy logs, URL filtering alerts, web gateway telemetry |
| Malicious File | T1204.002 | User opens a malicious attachment — endpoint process creation, document macro execution, EDR alerts |

**Why this matters:** Both indicate that social engineering succeeded — the email got past the gateway, the user clicked or opened something, and the attack is now executing on a real endpoint inside the network. The next 60 seconds of telemetry after a user execution event is the highest-value triage window in the entire kill chain. Sunny's hunt should anchor on these events and pivot to the host involved.

---

### Q4 — Command and Scripting Interpreters to hunt for

> *"Which scripting interpreters should Sunny search for to identify successful execution?"*

**Answer:** ✅ **PowerShell** and **Windows Command Shell**

Under Command and Scripting Interpreter (T1059), APT28 is documented using:

| Sub-technique | Technique ID | Detection focus |
|---|---|---|
| PowerShell | T1059.001 | Unusual PowerShell invocation following user execution — encoded commands, suspicious modules, network connections from `powershell.exe` |
| Windows Command Shell | T1059.003 | Unusual `cmd.exe` execution following user execution — long command lines, unusual parent processes, network connections from `cmd.exe` |

**Why this matters:** This is the chain that catches the attack in flight. If Sunny sees a user click a malicious link or open a malicious file, **and** PowerShell or cmd.exe fires up within seconds **and** that interpreter behaves unusually (encoded commands, base64 strings, network beacons), the attack is live. The remediation window is measured in minutes, not hours.

---

## The Hunt Sequence — Putting It Together

The four questions form a coherent threat-hunting workflow:

```
1. Resource Development (Q2)
   APT28 compromises email accounts to build infrastructure
                    ↓
2. Reconnaissance + Initial Access (Q1)
   Spearphishing Link sent from a compromised account
                    ↓
3. User Execution (Q3)
   User clicks the link or opens the file
                    ↓
4. Execution via Scripting Interpreter (Q4)
   PowerShell or cmd.exe fires up, attack is live
                    ↓
   Sunny detects, isolates, and responds
```

Each step has a specific telemetry source Sunny needs to interrogate:

| Step | Primary telemetry source |
|---|---|
| Resource Development | External threat intelligence feeds (compromised account lists) |
| Spearphishing | Email gateway logs, URL filtering logs |
| User Execution | EDR endpoint logs, web proxy logs |
| Scripting Interpreter | EDR process creation events, Sysmon Event ID 1 |

---

## Why This Workflow Matters in Production

This is the **threat-informed defence** loop that distinguishes a reactive SOC from a proactive one. A reactive SOC waits for SIEM alerts to fire and triages whatever comes through. A proactive SOC starts with threat intelligence (*"APT28 is targeting our sector"*), pulls the threat actor's documented TTPs from ATT&CK, and **hunts** for evidence of those techniques in its own environment before any alert fires.

The skill being tested in this room isn't memorising APT28's techniques. It's knowing the workflow:

1. Receive threat intel about a named adversary
2. Find them in ATT&CK Groups (`attack.mitre.org/groups/G0007/`)
3. Load their layer in Navigator
4. Identify the techniques most relevant to your environment
5. Pivot to your SIEM/EDR/email gateway and hunt for evidence

A SOC analyst who can describe this workflow in an interview demonstrates threat-hunting maturity well beyond Tier 1 baseline.

---

## Skills Demonstrated

| Skill | Evidence |
|---|---|
| MITRE ATT&CK Navigator fluency | Loaded the APT28 group layer and navigated by tactic and technique |
| Threat actor profiling | Identified APT28 as Russian GRU and mapped target profile fit to E-Corp |
| Tactic-to-technique mapping | Identified the same technique (Spearphishing Link) appearing under two tactics |
| Threat-informed hunting workflow | Articulated the full hunt sequence from Resource Development → Execution |
| Telemetry source awareness | Mapped each ATT&CK technique to the specific log source needed to hunt it |
| First-attempt accuracy | All four questions answered correctly on first submission |

---

## Frameworks Referenced

- **MITRE ATT&CK** — Enterprise matrix, Groups index (G0007 APT28)
- **MITRE ATT&CK Navigator** — interactive visualisation tool
- **The Diamond Model** (implicitly) — Adversary (APT28) → Capability (techniques) → Infrastructure (compromised email accounts) → Victim (E-Corp profile)

---

## Honest Reflection

The room is short but the workflow it teaches is the highest-leverage skill on the SOC L1 path — turning threat intelligence into a concrete hunt list. The four questions feel almost too easy in isolation, but together they walk a real, end-to-end intelligence-to-detection workflow.

What this room *doesn't* deliver is the next-step practice: actually loading the APT28 layer in Navigator, then running corresponding queries in a real SIEM to find matches. That's the operational skill that would justify a full SOC L2 / threat-hunting interview claim. For now: the framework is internalised, the workflow is documented, and the next time threat intelligence drops a named APT, I know exactly which sequence to run.

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*

**Tags:** `#eviction` `#mitre-attack` `#mitre-attack-navigator` `#apt28` `#fancy-bear` `#threat-hunting` `#threat-informed-defence` `#cti` `#cyber-defence-frameworks` `#soc-tier-1`