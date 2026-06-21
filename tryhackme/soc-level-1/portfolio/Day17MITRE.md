# MITRE — Portfolio Write-up

**Room:** MITRE
**Path:** SOC Level 1 · Cyber Defence Frameworks
**Date completed:** 19 June 2026 (Day 17)
**Format:** Research-based — no contained lab environment or final flag. The whole room is a guided exploration of MITRE's public knowledge bases.

---

## 1. Overview

This room introduced the **MITRE ecosystem of cybersecurity frameworks** — far more than just ATT&CK. The room is structured as a guided tour: each task introduces a different MITRE resource and asks research-based questions that require navigating the actual MITRE websites to find answers. Unlike most rooms in the Cyber Defence Frameworks module, there is no contained lab with a final flag — the room itself *is* the practical, testing whether you can navigate MITRE's public knowledge bases to answer operational questions.

The five MITRE resources covered:

1. **MITRE ATT&CK®** — the adversary tactics, techniques, and procedures knowledge base
2. **MITRE CAR** — the Cyber Analytics Repository
3. **MITRE D3FEND™** — the defensive techniques knowledge graph
4. **MITRE ATLAS** — adversarial threats to AI/ML systems
5. **MITRE AADAPT** — adversarial threats to digital asset payment technologies (blockchain/crypto)

This write-up walks through each resource using the research questions answered during the room as the practical anchor.

---

## 2. MITRE the Organisation

Worth getting straight first because interviewers ask it: **MITRE Corporation** is a US not-for-profit founded in **1958**, originally to support the US Air Force. It now runs multiple federally-funded research and development centres (FFRDCs) across cybersecurity, aerospace, healthcare, AI, and space systems. Cybersecurity is **one of several domains** MITRE works in, not its sole purpose.

Its stated mission: *"to solve problems for a safer world."*

MITRE produces ATT&CK, CAR, D3FEND, Engage, ATLAS, AADAPT, and many other frameworks — but it's not a vendor and doesn't sell security products. The frameworks are publicly available and free to use.

---

## 3. MITRE ATT&CK® — The Centre of the Ecosystem

**ATT&CK** stands for **Adversarial Tactics, Techniques, and Common Knowledge**. It is a globally-accessible knowledge base of adversary behaviour, drawn from observed real-world attacks and maintained by MITRE since 2013.

### Structural Vocabulary

ATT&CK uses a precise four-level hierarchy that's worth nailing down because interviewers test it:

| Level | What it represents | Example |
|---|---|---|
| **Tactic** | The adversary's *goal* at a stage of the attack | TA0005 — Defense Evasion |
| **Technique** | The *general method* used to achieve the tactic | T1564 — Hide Artifacts |
| **Sub-technique** | A more *specific variant* of the technique | T1564.001 — Hidden Files and Directories |
| **Procedure** | The *specific implementation* used by a named threat actor | "APT28 uses `attrib +h` to hide files on the target" |

The shorthand **TTPs** = Tactics, Techniques, and Procedures (all three together — *don't* use it to mean just "techniques").

### The Three ATT&CK Matrices

ATT&CK is published as three separate matrices, each tailored to a different attack surface:

| Matrix | Coverage |
|---|---|
| **Enterprise** | Windows, Linux, macOS, network, cloud (AWS/Azure/GCP/SaaS), containers, identity |
| **Mobile** | iOS and Android |
| **ICS** | Industrial Control Systems / Operational Technology |

The Enterprise matrix is what most SOCs work against. The current Enterprise matrix has 14 tactics, ranging from **TA0043 Reconnaissance** (start of attack) to **TA0040 Impact** (final destructive action).

### Task 2 — ATT&CK Framework Basics

The room tested baseline ATT&CK navigation with two research questions:

| Question | Answer | Where to find it |
|---|---|---|
| What Tactic does the Hide Artifacts technique belong to in the ATT&CK Matrix? | **Stealth** | attack.mitre.org → Techniques → Enterprise → Hide Artifacts |
| Which ID is associated with the Create Account technique? | **T1136** | attack.mitre.org → Techniques → Enterprise → Create Account |

**A caveat on the Hide Artifacts answer.** The room accepted "Stealth" as the answer, but in the canonical MITRE ATT&CK Enterprise matrix, **T1564 (Hide Artifacts) sits under TA0005 — Defense Evasion**, not under a tactic called "Stealth." This may be a TryHackMe-specific phrasing or a reference to a non-Enterprise variant of the matrix. In production work, the canonical source (attack.mitre.org) is what counts — if a room and the canonical framework disagree, trust the framework.

---

## 4. CTI Profiling Walkthrough — Mustang Panda (Task 3)

The room's first practical CTI exercise profiled **Mustang Panda** (also tracked as Earth Preta, RedDelta, TA416, Bronze President). This is a real-world Chinese state-sponsored APT group active since at least 2014, known for targeting NGOs, religious institutions, and government entities — particularly the Vatican, European NGOs, and South-East Asian governments.

| Question | Answer | Significance |
|---|---|---|
| In which country is Mustang Panda based? | **China** | State-sponsored attribution — the country of origin shapes likely targets and motivations |
| Which ATT&CK technique ID maps to Mustang Panda's Reconnaissance tactics? | **T1598** | T1598 = Phishing for Information — gathering target info before initial access |
| Which software is Mustang Panda known to use for Access Token Manipulation? | **Cobalt Strike** | One of the most widely-abused post-exploitation frameworks in the industry |

### Why This Workflow Matters

The exercise demonstrates the **CTI-to-detection-engineering pipeline** that mature SOCs run every day:

1. **Identify a relevant threat actor** (Mustang Panda targets organisations like yours).
2. **Pull their documented TTPs** from ATT&CK's Groups index.
3. **Map those TTPs onto your detection coverage** — what would catch their techniques?
4. **Identify gaps** — which Mustang Panda techniques have no detection coverage in your environment?
5. **Write detections** to close those gaps.

Cobalt Strike's prevalence here is also worth noting — it's listed as a tool used by dozens of named APT groups in ATT&CK. A SOC strong on Cobalt Strike detection (Sigma rules for its named-pipe defaults, beacon traffic patterns, common loaders) is implicitly defending against a huge slice of the threat landscape.

---

## 5. CTI and Threat Intelligence Walkthrough — APT33 (Task 4)

The room's second and deeper CTI exercise profiled **APT33** (also tracked as Elfin, Magnallium, HOLMIUM, Refined Kitten). This is an Iranian state-sponsored threat group active since at least 2013, well-documented for targeting the **aviation sector** (especially US military aviation), petrochemical, and energy industries. They've been associated with destructive operations including variants of the Shamoon wiper malware.

The exercise walked through a complete defensive-action chain: profile the group → identify their key technique → identify the tool → identify the mitigation → identify the detection strategy.

| # | Question | Answer | Notes |
|---|---|---|---|
| 1 | Which APT group has targeted the aviation sector and has been active since at least 2013? | **APT33** | The defining sector targeting fingerprint |
| 2 | Which ATT&CK sub-technique used by this group is a key area of concern for companies using Office 365? | **Cloud Accounts** (T1078.004) | Sub-technique of T1078 Valid Accounts |
| 3 | According to ATT&CK, what tool is linked to the APT group and the sub-technique you identified? | **Ruler** | Open-source pen-testing tool by SensePost that abuses Outlook/Exchange features — turned by APT33 into an attack tool |
| 4 | Which mitigation strategy advises removing inactive or unused accounts to reduce exposure to this sub-technique? | **User Account Management** | The mitigation framework for joiner/mover/leaver discipline |
| 5 | What Detection Strategy ID would you implement to detect abused or compromised cloud accounts? | **DET0546** | Detection strategy ID — the actionable counterpart to the mitigation |

### What This Exercise Was Testing

This is the most operationally useful sequence in the entire room. It walks the full **threat → technique → tool → mitigation → detection** chain that real SOC work follows. By the end of it, you've gone from a named threat actor (APT33) to a specific detection strategy ID you could implement against them.

In a real environment, the workflow would be:
1. *"APT33 targets our sector."*
2. *"Their key Office 365 technique is Cloud Accounts (T1078.004)."*
3. *"They use Ruler to exploit it."*
4. *"Our mitigation is tighter joiner/mover/leaver hygiene (User Account Management)."*
5. *"Our detection strategy (DET0546) covers abused or compromised cloud accounts."*

That's a complete, defensible answer to *"how would you defend against APT33?"* — built entirely from ATT&CK research.

---

## 6. MITRE CAR — Cyber Analytics Repository

**CAR** is MITRE's repository of **detection analytics** aligned to ATT&CK techniques. Where ATT&CK tells you *what attackers do*, CAR tells you *how to detect them doing it* — with concrete pseudocode for each analytic.

Each CAR analytic includes:
- The **ATT&CK technique(s) it detects**
- A **description** of what the analytic looks for
- **Pseudocode** showing the detection logic
- **Data model references** for what telemetry the analytic needs
- An **analytic type** (e.g. Detection, Situational Awareness, Hunting)

### Task 5 — CAR Research Questions

| Question | Answer | What it means |
|---|---|---|
| Which ATT&CK Tactic is associated with CAR-2019-07-001? | **Defense Evasion** | The tactic CAR-2019-07-001 helps detect against |
| What is the Analytic Type for Access Permission Modification? | **Situational Awareness** | The analytic's purpose — increases visibility rather than directly alerting |

**Analytic types matter** because they shape how the analytic is operationalised:
- **Detection** analytics fire alerts directly
- **Situational Awareness** analytics build context (used for hunting and triage, not direct alerting)
- **Hunting** analytics surface anomalies for human investigation

A SOC's analytics library should be balanced across all three — pure detection without context produces alerts no one can triage, while pure situational awareness without detection produces noise no one acts on.

---

## 7. MITRE D3FEND™ — Defensive Counterpart to ATT&CK

**D3FEND** is the defensive complement to ATT&CK — a knowledge graph of **defensive techniques**, each mapped to the ATT&CK technique(s) it counters. The project is funded by the NSA.

Where ATT&CK catalogues *offence*, D3FEND catalogues *defence* — and crucially, each D3FEND technique includes a **digital artifact** (the kind of telemetry or system component it operates on) and the **ATT&CK techniques it mitigates**.

### Task 6 — D3FEND Research Questions

| Question | Answer | Significance |
|---|---|---|
| Which sub-technique of User Behavior Analysis would you use to analyze the geolocation data of user logon attempts? | **User Geolocation Logon Pattern Analysis** | Detects impossible-travel scenarios (e.g. London login then Tokyo login 10 minutes later) |
| Which digital artifact does this sub-technique rely on analyzing? | **Network Traffic** | The data source feeding the analytic |

### Why D3FEND Matters

D3FEND closes a gap that ATT&CK alone doesn't address — *"now that I know what the attacker is doing, what specific defensive techniques counter it?"* For mature SOCs, the workflow is:

1. **ATT&CK** → identify the technique the adversary is using.
2. **D3FEND** → identify defensive techniques that counter it.
3. **CAR** → identify detection analytics that catch it.
4. **Engage** → identify deception or denial techniques to disrupt it.

Together, these four MITRE knowledge bases form a complete defensive design loop — adversary catalogue → defensive countermeasure catalogue → detection analytics → adversary engagement.

---

## 8. Other MITRE Frameworks — AADAPT and ATLAS (Task 7)

The room closed by introducing two newer, narrower MITRE frameworks for emerging attack surfaces.

### AADAPT — Adversarial Actions in Digital Asset Payment Technologies

AADAPT covers adversary techniques against **blockchain, cryptocurrency, and digital payment systems**. Technique IDs use the `ADT` prefix.

| Question | Answer |
|---|---|
| What technique ID is associated with Scrape Blockchain Data in the AADAPT framework? | **ADT3025** |

This framework is increasingly relevant as financial-sector SOCs deal with crypto-targeting threat actors (Lazarus Group's crypto thefts are the canonical example).

### ATLAS — Adversarial Threat Landscape for AI Systems

ATLAS covers adversary techniques against **AI and ML systems** — prompt injection, model evasion, data poisoning, and similar attacks.

| Question | Answer |
|---|---|
| Which tactic does LLM Prompt Obfuscation belong to in the ATLAS framework? | **Defense Evasion** |

ATLAS borrows ATT&CK's tactic vocabulary (Defense Evasion appears in both) but applies it to AI/ML-specific attack surfaces. As LLMs and AI systems become embedded in production environments, ATLAS will become the framework for AI-security threat modelling — comparable in importance to what ATT&CK is for traditional infrastructure.

---

## 9. How the MITRE Ecosystem Connects

The five resources covered in this room aren't independent — they form a layered defensive design loop:

```
                 ┌────────────────────┐
                 │      ATT&CK        │
                 │  (What attackers   │
                 │       do)          │
                 └─────────┬──────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
  ┌──────────┐      ┌──────────┐      ┌─────────────┐
  │   CAR    │      │ D3FEND   │      │ ATLAS/AADAPT│
  │ (How to  │      │ (How to  │      │ (Domain-    │
  │ detect)  │      │ defend)  │      │ specific)   │
  └──────────┘      └──────────┘      └─────────────┘
```

A mature SOC uses ATT&CK as the **shared vocabulary**, CAR as the **detection library**, D3FEND as the **defensive technique reference**, and domain-specific frameworks (ATLAS, AADAPT, ICS) when working on those specialised attack surfaces.

---

## 10. Where This Sits in the Module

MITRE is the **fourth framework room** in the Cyber Defence Frameworks module:

| Position | Room | What it adds |
|---|---|---|
| 1 | Pyramid of Pain | The IOC durability hierarchy — *what to detect* |
| 2 | Cyber Kill Chain | The traditional 7-phase attack lifecycle — *where in the attack* |
| 3 | Unified Kill Chain | The expanded 18-phase lifecycle handling lateral movement |
| **4** | **MITRE** (this room) | **The technique-level catalogue underneath every kill-chain phase** |
| 5 | SUMMIT | The capstone lab applying all of the above |

The MITRE room completes the conceptual foundation that SUMMIT (next) will exercise practically. By the time you walk into SUMMIT, you should be able to:

1. Identify *what* to detect (Pyramid of Pain — go for behaviours over IOCs)
2. Identify *where* the activity sits in the lifecycle (Kill Chain / UKC phase)
3. Identify *which technique* it represents (MITRE ATT&CK technique ID)
4. Identify *how to detect it* (MITRE CAR analytic)
5. Identify *how to defend against it* (MITRE D3FEND counter-technique)

That's the full defensive design loop. Every serious detection engineering decision in a real SOC runs through these five questions.

---

## 11. Skills Demonstrated

| Skill | Evidence |
|---|---|
| MITRE ATT&CK navigation | Answered Q&A across Tasks 2–7 by navigating attack.mitre.org and adjacent MITRE sites directly |
| CTI workflow execution | Profiled two named threat actors (Mustang Panda, APT33), pulling their techniques, tools, and target sectors from ATT&CK |
| Cross-framework integration | Walked the APT33 chain end-to-end: threat actor → technique → tool → mitigation → detection strategy |
| Ecosystem awareness | Articulated how ATT&CK, CAR, D3FEND, ATLAS, and AADAPT relate as a layered defensive design loop |
| Critical reading | Flagged the "Stealth" vs "Defense Evasion" discrepancy on Hide Artifacts — recognising that room answers and canonical framework sources don't always agree |
| Emerging-threat awareness | Recognised the relevance of ATLAS (AI/ML) and AADAPT (blockchain/crypto) for future SOC work |

---

## 12. Cross-Reference to Eviction (Earlier Practical)

The MITRE ecosystem covered in this room was applied in practice in the **Eviction** lab (documented separately) — a SOC analyst scenario at E-Corp where I used the **MITRE ATT&CK Navigator** to load APT28's known TTPs and threat-hunt the network for matching activity.

The two rooms are complementary:
- **MITRE (this room)** = the *framework theory* — what ATT&CK is, what CAR is, what D3FEND is, how they relate
- **Eviction** = the *applied practice* — using the ATT&CK Navigator tool to actually hunt a named threat actor in a simulated environment

Together they cover both the *knowledge* and the *operational use* of the MITRE ecosystem.

---

## 13. Honest Reflection

A few honest caveats about what this room does and doesn't deliver:

- **The room is a guided tour, not a deep dive.** Real ATT&CK proficiency comes from spending hours navigating the matrix for specific operational questions, not from answering 14 lookup questions. The room is a starting point — competence requires continuing to use attack.mitre.org regularly for real work.
- **The "Stealth" answer worries me.** It's not the canonical MITRE Enterprise tactic name. I documented the room's answer accurately but in any production setting I'd verify against the canonical source (attack.mitre.org) rather than rely on a room's phrasing.
- **MITRE ATT&CK Navigator** is the tool that ties this room and Eviction together — I'd recommend anyone reading this portfolio piece spend 20 minutes at `mitre-attack.github.io/attack-navigator` clicking through layers, group overlays, and technique highlights. It teaches more than the room itself.
- **The framework is constantly evolving.** ATT&CK gets versioned releases (current is v15+, updated several times a year); D3FEND, ATLAS, and AADAPT are all younger and more rapidly changing. Treat any specific technique ID or analytic ID as a snapshot of the framework at the time of completion (June 2026), not a permanent fact.

The next room in the module — SUMMIT — applies all of this in a capstone lab. For now: the MITRE ecosystem is mapped, the CTI workflow is internalised, and the five resources are no longer just acronyms.

---

**Tags:** `#mitre` `#mitre-attack` `#mitre-car` `#mitre-d3fend` `#atlas` `#aadapt` `#cti` `#threat-hunting` `#apt33` `#mustang-panda` `#cobalt-strike` `#cyber-defence-frameworks` `#soc-tier-1`