# Cyber Kill Chain — Portfolio Write-up

**Room:** Cyber Kill Chain
**Path:** SOC Level 1 · Cyber Defence Frameworks
**Date completed:** 18 June 2026 (Day 15)
**Flag:** `THM{7HR347_1N73L_12_4w35om3}`

---

## 1. Overview

This room introduced the **Cyber Kill Chain®** — a model developed by **Lockheed Martin in 2011** that adapted the military concept of a "kill chain" (find, fix, track, target, engage, assess) to the cybersecurity domain. The model decomposes a successful intrusion into seven discrete phases an adversary must complete to achieve their objective.

The room covers the theory of each phase, the strategic principle of "breaking the chain," and culminates in a practical analysis (Task 9) requiring the learner to map six attacker techniques from the **2013 Target Corporation data breach** to their correct Kill Chain phases.

---

## 2. The Model

Lockheed Martin's core insight: an intrusion isn't a single event — it's a *process*. By decomposing the process into stages, defenders can:

1. **Identify where** in the attack lifecycle a piece of activity sits.
2. **Map controls** to specific phases (email gateway for Delivery, EDR for Installation, network detection for C2, etc.).
3. **Disrupt at the earliest possible phase** — the further left in the chain you break it, the less damage occurs.

The seven phases, in order:

```
1. Reconnaissance        ← Attacker gathers intel
2. Weaponization         ← Attacker builds the payload
3. Delivery              ← Payload reaches the victim
4. Exploitation          ← Payload executes & exploits a vulnerability
5. Installation          ← Attacker establishes persistence
6. Command & Control     ← Compromised host calls home
7. Actions on Objectives ← Attacker achieves their goal
```

---

## 3. The Seven Phases

### 1. Reconnaissance

**What it is:** The attacker gathers information about the target — employees, infrastructure, technologies in use, vendors, contractors. Split into:
- **Passive recon:** OSINT, LinkedIn scraping, WHOIS, Shodan, certificate transparency logs — no traffic touches the target.
- **Active recon:** Port scanning, vulnerability scanning, service enumeration — leaves traces on the target.

**Defender visibility:** Near zero for passive recon; some for active recon (via IDS/IPS and edge logs).

**Why it matters:** This is the only phase that happens *entirely outside* your perimeter, making it the hardest to detect proactively. Most defenders only learn about reconnaissance retrospectively, after the breach.

### 2. Weaponization

**What it is:** The attacker pairs an exploit with a deliverable payload — e.g. embedding a malicious macro in an Office document, or pairing CVE-2017-0199 with a remote template injection payload.

**Defender visibility:** Effectively zero — this happens on the attacker's infrastructure.

**Why it matters:** Defenders detect *artefacts of* weaponization (YARA rules for known builders, malware family signatures) rather than the act itself. This phase is invisible to the SOC in real time.

### 3. Delivery

**What it is:** The weaponised payload reaches the victim. Common vectors:
- Email attachment (spearphishing)
- Drive-by download from a compromised website
- USB drop
- Watering hole (compromising a site the target visits)
- Supply chain (compromising a trusted vendor)

**Defender visibility:** **This is where the SOC first gets visibility.** Email gateways, web proxies, DLP, and network IDS all live at this layer.

**Why it matters:** Breaking the chain at Delivery is the single highest-leverage defensive opportunity. A spearphishing email blocked at the gateway costs the SOC nothing; a spearphishing email that lands costs hours of investigation even if it doesn't succeed.

### 4. Exploitation

**What it is:** The payload executes and exploits a vulnerability — could be a user double-clicking (social engineering), an auto-running macro, an RCE in a public-facing app, or a deserialisation flaw.

**Defender visibility:** EDR gets its first real chance to catch the attack here.

**Why it matters:** Patching, application control, and EDR behavioural detection all live at this phase. A well-tuned EDR will catch most exploitation attempts on managed endpoints.

### 5. Installation

**What it is:** The attacker establishes persistence on the compromised host. Common techniques:
- Registry Run keys
- Scheduled tasks
- Windows services
- WMI event subscriptions
- Startup folder entries
- Dynamic linker hijacking (on Linux/macOS)

**Defender visibility:** High — modern EDRs are good at this layer. Sysmon Event ID 11/13/15 cover most persistence artefacts.

**Why it matters:** This maps cleanly onto **MITRE ATT&CK Tactic TA0003 — Persistence**, which makes it the easiest Kill Chain phase to map to ATT&CK technique coverage.

### 6. Command & Control (C2)

**What it is:** The compromised host establishes communication back to the attacker. Channels:
- HTTP/HTTPS beacons to attacker-controlled servers
- DNS tunnelling
- Fronting through legitimate services (Slack, Discord, Telegram, GitHub)
- Domain generation algorithms (DGA)
- Fallback channels (secondary C2 in case primary is blocked)

**Defender visibility:** High at the network layer — proxy logs, DNS logs, NetFlow. This is **the longest-running phase in real breaches** — often weeks or months — making it the highest-yield place for retrospective network detection.

**Why it matters:** Most APT detections in production environments happen here, not at Exploitation. A SOC that's strong on egress monitoring catches more attacks than a SOC that's strong on ingress filtering alone.

### 7. Actions on Objectives

**What it is:** The attacker does what they came to do:
- Data exfiltration
- Ransomware deployment
- Lateral movement to crown-jewel systems
- Account takeover
- Destruction or sabotage

**Defender visibility:** Variable — depends entirely on internal monitoring posture (DLP, internal network sensors, audit logs).

**Why it matters:** By the time you detect at this phase, you've already lost something. Detection here is necessary for incident response but is, in detection-engineering terms, a *failure mode* — you wanted to catch the attack earlier in the chain.

---

## 4. The Practical — Target 2013 Breach Walkthrough

The room's Task 9 practical anchored the framework against one of the most-studied breaches in cybersecurity history: the **2013 Target Corporation data breach**.

### Breach Background

- **Dates:** 27 November – 15 December 2013 (the US Thanksgiving / Black Friday shopping season)
- **Impact:** Approximately **40 million credit and debit card records** compromised, plus an additional ~70 million records containing personal information
- **Initial vector:** Attackers compromised **Fazio Mechanical Services**, an HVAC contractor with network access to Target's vendor portal, via the **Citadel trojan** delivered through spearphishing
- **Pivot:** From Fazio's network access, attackers moved laterally into Target's internal network and ultimately reached the Point-of-Sale (POS) systems
- **Payload:** A custom RAM-scraping malware (BlackPOS / Kaptoxa) that harvested unencrypted card data from POS terminal memory before encryption
- **Outcome:** $18.5 million multistate settlement — the largest data-breach settlement in US history at that time

This breach is a touchstone case because it demonstrated multiple Kill Chain failures simultaneously: a weak third-party security posture (Fazio), insufficient network segmentation (vendor portal → POS network), and inadequate egress monitoring (exfiltration ran for weeks).

### The Mapping

The lab provided six attacker techniques to map to Kill Chain phases:

| Phase | Technique mapped | What this looked like in the Target breach |
|---|---|---|
| **Reconnaissance** | *(no input required — implicit in the scenario)* | Attackers identified Fazio Mechanical Services as a third-party with privileged network access, likely via OSINT on Target's vendor relationships |
| **Weaponization** | **Powershell** | Attackers built their toolkit (PowerShell-based tooling for lateral movement and post-exploitation) |
| **Delivery** | **Spearphishing attachment** | Citadel trojan delivered to Fazio Mechanical via a spearphishing email |
| **Exploitation** | **Exploit public-facing application** | Attackers leveraged compromised Fazio credentials to access Target's externally-facing vendor portal (Ariba) |
| **Installation** | **Dynamic linker hijacking** | Persistence established on compromised hosts via linker-level hijacking — a technique that subverts how the OS loads shared libraries |
| **Command & Control** | **Fallback channels** | Secondary C2 channels maintained for resilience if primary infrastructure was blocked |
| **Actions on Objectives** | **Data from local system** | RAM-scraping of POS terminals to harvest 40 million card records from local system memory |

All six mapped correctly. Flag returned: `THM{7HR347_1N73L_12_4w35om3}`.

### A Vocabulary Caveat Worth Flagging

The lab's mapping of **PowerShell → Weaponization** is pedagogically convenient but slightly inconsistent with how MITRE ATT&CK classifies PowerShell. In ATT&CK, **PowerShell (T1059.001)** sits in the **Execution** tactic (TA0002), not in any "weaponization" tactic. ATT&CK doesn't have a Weaponization tactic at all — that concept is Kill Chain-specific.

The pragmatic reading: the room is teaching that an attacker can *weaponize* PowerShell as their primary attack tool. The cleaner mapping in a real intrusion analysis would have been to Installation or Execution, depending on what the PowerShell was actually doing. This is the kind of inconsistency between frameworks that comes up in real SOC work — Kill Chain, ATT&CK, and Diamond Model don't always agree on where a technique sits, and the analyst's job is to navigate that ambiguity.

---

## 5. The "Break the Chain" Defence Philosophy

The Kill Chain's strategic claim is simple and powerful: **defenders only need to win once per attack; attackers need to win every phase**.

This asymmetry inverts the usual defender's disadvantage. In most security domains, the attacker has the advantage of choosing time, place, and method. The Kill Chain reminds defenders that they only need to break *one* link to make the whole chain fail.

The corollary is that defensive investment should be balanced across all seven phases — a SOC strong on Delivery (email gateway) but weak on C2 (egress monitoring) will catch opportunistic attacks but miss persistent ones. A balanced posture beats deep coverage of any single phase.

### Mapping Defensive Controls to Phases

| Phase | Primary defensive controls |
|---|---|
| Reconnaissance | OSINT monitoring, brand protection, deception (canary tokens) |
| Weaponization | Threat intelligence on tooling, YARA signatures |
| Delivery | Email gateway, web proxy, DLP, user awareness training |
| Exploitation | Patching, EDR, application control, MFA |
| Installation | EDR (persistence detection), Sysmon, file integrity monitoring |
| Command & Control | Egress filtering, DNS monitoring, proxy logs, network detection |
| Actions on Objectives | DLP, internal network monitoring, privileged access management |

A SOC maturity assessment can use this table as a self-audit: where are we strong, where are we weak, and what would a phase-by-phase attack look like against our actual control set?

---

## 6. Limitations of the Traditional Kill Chain

The room covered these, and they're worth keeping in mind because interviewers will ask:

1. **Too perimeter-focused.** The model assumes the attacker is outside and works their way in. It doesn't cleanly handle **insider threats**, where the attacker is already past Delivery and Exploitation by virtue of having legitimate access.

2. **Too linear.** Real attacks loop back — an attacker who's established Installation often runs additional Reconnaissance from inside the network, then Weaponizes new payloads for lateral movement. The Kill Chain implies a one-way flow that doesn't match reality.

3. **Weak on cloud and SaaS.** The model predates the modern cloud-first enterprise. Concepts like "Delivery" don't translate cleanly when the attack surface is a misconfigured S3 bucket, a leaked API key, or an OAuth consent grant.

4. **Conflates the early phases.** Reconnaissance and Weaponization are often treated together in practice because defenders have almost no visibility into either. The model spends two phases on what is effectively one black-box pre-attack stage.

5. **No coverage of post-exfiltration.** What happens after Actions on Objectives — data monetisation, victim shaming (ransomware), supply-chain follow-on attacks — sits outside the model entirely.

These limitations are exactly why **Paul Pols developed the Unified Kill Chain in 2017**, expanding the model to 18 phases organised into three macro-stages and explicitly accommodating insider threats, lateral movement loops, and post-compromise activity. That's the next room in this module.

---

## 7. Where This Sits in the Module

Cyber Kill Chain is the **second framework** in the Cyber Defence Frameworks module. Its position:

| Position | Room | What it adds |
|---|---|---|
| 1 | Pyramid of Pain | The IOC durability hierarchy — *what to detect* |
| **2** | **Cyber Kill Chain** (this room) | **The attack lifecycle — *where* in the attack to detect** |
| 3 | Unified Kill Chain | An expanded, modernised attack lifecycle handling cloud, insiders, and lateral movement |
| 4 | MITRE | The taxonomic catalogue of TTPs that map onto Kill Chain phases |
| 5 | SUMMIT | The capstone lab applying all of the above |

The Kill Chain and the Pyramid of Pain interlock cleanly:
- **IOCs (hashes, IPs, domains)** cluster at Delivery and C2 phases.
- **Network/host artefacts** cluster around Installation and C2.
- **TTPs** span Installation, C2, and Actions on Objectives — and represent the most durable detection opportunity in each phase.

Knowing both frameworks lets you answer two questions simultaneously: *where in the attack lifecycle is this signal?* (Kill Chain) and *how durable is my detection if I write a rule on it?* (Pyramid of Pain).

---

## 8. Skills Demonstrated

| Skill | Evidence |
|---|---|
| Framework knowledge | Articulated all seven phases of the Kill Chain with concrete techniques and defensive controls |
| Real-world case study analysis | Mapped six techniques from the 2013 Target breach to their correct Kill Chain phases first time |
| Cross-framework awareness | Flagged the PowerShell→Weaponization inconsistency between Kill Chain and MITRE ATT&CK |
| Defensive strategic thinking | Connected each Kill Chain phase to its corresponding control category and defensive opportunity |
| Recognition of model limitations | Identified the perimeter-focus, linearity, and cloud-blindness criticisms that drove the development of the Unified Kill Chain |

---

## 9. Honest Reflection

The Kill Chain is genuinely useful, but I want to be clear about what it is and isn't:

- It **is** an excellent communication tool. Saying "we detected at C2" is more precise than saying "we detected an attack."
- It **is** an excellent self-audit framework for defensive coverage.
- It **is not** a real-time triage tool. By the time you can map an alert to a Kill Chain phase, you're already doing intrusion analysis — the alert itself rarely tells you which phase it sits in.
- It **is not** sufficient on its own for modern environments. The model's 2011 perimeter-focus shows its age, and any serious detection programme today layers it with Unified Kill Chain (for richer lifecycle modelling) and MITRE ATT&CK (for technique-level granularity).

The next room in the module addresses limitation #2 directly. The MITRE room a couple of days later addresses limitations #3 and #4. For now: the framework is internalised, the Target breach walkthrough is documented, and the flag is captured.

---

**Tags:** `#cyber-kill-chain` `#lockheed-martin` `#intrusion-analysis` `#target-breach-2013` `#detection-engineering` `#mitre-attack` `#cyber-defence-frameworks` `#soc-tier-1`