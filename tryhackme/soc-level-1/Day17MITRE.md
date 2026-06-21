# 🛡️ MITRE – TryHackMe

## 📅 Date
Day 17 · 19 June 2026

## 📚 What I Learned (3–5 lines max)
- **MITRE Corporation** — a US not-for-profit research organisation (founded 1958) running federally-funded R&D centres across cybersecurity, aerospace, healthcare, and AI. Cybersecurity is one of several domains, not its sole mission.
- **MITRE ATT&CK®** — the central knowledge base of adversary tactics, techniques, sub-techniques, and procedures (TTPs). Organised into Tactics (the *why*, e.g. TA0005 Defense Evasion) and Techniques (the *how*, e.g. T1136 Create Account).
- **The wider MITRE ecosystem** beyond ATT&CK: **CAR** (analytics for detecting techniques), **D3FEND** (defensive countermeasures mapped to ATT&CK), **ATLAS** (adversarial AI/ML), and **AADAPT** (adversarial blockchain/crypto).
- **CTI workflow** — using ATT&CK to profile a named threat actor by pulling their documented techniques, then mapping those onto your detection coverage to identify gaps. This is the single highest-leverage workflow in modern threat hunting.

## 🛠️ What I Did
- Worked through the room's six research-based tasks, navigating the actual MITRE websites (attack.mitre.org, car.mitre.org, d3fend.mitre.org, atlas.mitre.org, aadapt.mitre.org) to answer 14 questions across five MITRE resources.
- **Task 2 (ATT&CK Framework basics):** Identified the tactic for the Hide Artifacts technique (**Stealth**) and the ID for the Create Account technique (**T1136**).
- **Task 3 (CTI Profiling — Mustang Panda):** Identified Mustang Panda's base country (**China**), Reconnaissance technique ID (**T1598**), and Access Token Manipulation software (**Cobalt Strike**).
- **Task 4 (Threat Intelligence — APT33):** Profiled APT33's aviation-sector targeting, identified Cloud Accounts (T1078.004) as the key Office 365 sub-technique, the **Ruler** tool, **User Account Management** mitigation, and detection strategy ID **DET0546**.
- **Task 5 (CAR):** Identified Defense Evasion tactic for CAR-2019-07-001 and Situational Awareness analytic type for Access Permission Modification.
- **Task 6 (D3FEND):** Found **User Geolocation Logon Pattern Analysis** sub-technique relying on **Network Traffic** digital artifact.
- **Task 7 (AADAPT & ATLAS):** Identified **ADT3025** for Scrape Blockchain Data and **Defense Evasion** for LLM Prompt Obfuscation.

## 🔐 Why It Matters
- MITRE ATT&CK is the single most-referenced framework in modern SOC work. It's the common vocabulary between threat intelligence, detection engineering, and incident response — without it, those three disciplines speak different languages.
- The **CTI-profiling workflow** I practised on Mustang Panda and APT33 is exactly what production threat-hunting teams do every day. The skill being tested isn't memorising techniques — it's knowing *how to navigate the ATT&CK matrix to answer specific operational questions*.
- The room introduced me to **ATLAS** and **AADAPT** — two newer MITRE frameworks I hadn't seen before, covering AI/ML adversarial threats and blockchain/crypto threats respectively. Both will be increasingly important in SOC work over the next 5 years.

## ❓ One Thing I Didn't Fully Understand
- The room accepted **"Stealth"** as the tactic for Hide Artifacts (T1564), but in the official MITRE ATT&CK Enterprise matrix that technique sits under **Defense Evasion (TA0005)**, not a tactic called "Stealth." This may be a TryHackMe-specific phrasing or an older framework convention. Worth verifying directly against attack.mitre.org/tactics in any production setting — when in doubt, the canonical source wins, not the room.