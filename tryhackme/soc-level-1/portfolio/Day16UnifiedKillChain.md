# Unified Kill Chain — Portfolio Write-up

**Room:** Unified Kill Chain
**Path:** SOC Level 1 · Cyber Defence Frameworks
**Date completed:** 19 June 2026 (Day 16)
**Flag:** `THM{UKC_SCENARIO}`

---

## 1. Overview

This room introduced the **Unified Kill Chain (UKC)** — a framework developed by **Paul Pols in 2017** as part of his MSc thesis at the Cyber Security Academy The Hague (Leiden University), in collaboration with Fox-IT. The UKC was a deliberate response to known limitations in Lockheed Martin's 2011 Cyber Kill Chain and an attempt to bridge that older intrusion-analysis model with the technique-level granularity of MITRE ATT&CK.

The room covers all 18 UKC phases organised into 3 macro-stages, and culminates in a hands-on matching practical (Task 8) where the learner pairs attacker actions to their correct UKC phase.

---

## 2. Why the UKC Was Created

The traditional Cyber Kill Chain (covered in the previous room) was developed in 2011 for a very specific threat model: a perimeter-based enterprise being attacked by an external adversary using email-borne malware. By the mid-2010s that model was showing its age in five concrete ways:

1. **No coverage of lateral movement.** The traditional Kill Chain assumes the attacker reaches their objective on the initially-compromised host. In reality, attackers almost always pivot.
2. **No coverage of insider threats.** The model assumes the attacker is outside the perimeter.
3. **Linear when reality is non-linear.** Attackers loop — running new reconnaissance from inside the network, weaponizing payloads against newly-discovered hosts, re-establishing persistence in different locations.
4. **Cloud/SaaS blindness.** "Delivery" doesn't translate cleanly when the attack surface is a misconfigured S3 bucket or a leaked OAuth token.
5. **No alignment with MITRE ATT&CK.** By 2017, ATT&CK had become the industry-standard technique catalogue, but there was no clean way to map Kill Chain phases to ATT&CK tactics.

Pols set out to fix all five. The result is a framework that's structurally larger (18 phases vs 7), explicitly acknowledges non-linear attack paths, and uses vocabulary borrowed directly from ATT&CK tactic names.

---

## 3. The Structure — Three Macro-Stages

The UKC's most important contribution isn't its 18 phases — it's the organisation of those phases into three macro-stages that correspond to genuinely different defensive postures:

```
┌─────────────────────────────────────────────────────────────┐
│  IN (Initial Foothold)                                       │
│  Getting into the target environment                         │
│  Defensive focus: prevent the foothold                       │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  THROUGH (Network Propagation)                               │
│  Moving laterally, escalating, expanding access              │
│  Defensive focus: contain the breach, limit blast radius     │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  OUT (Action on Objectives)                                  │
│  Achieving the attacker's goal                               │
│  Defensive focus: prevent exfiltration, recover quickly      │
└─────────────────────────────────────────────────────────────┘
```

The **Through** stage is the model's single biggest contribution. In the traditional Kill Chain, everything between Installation and Actions on Objectives was effectively absent — the model implied the attacker reached their goal on the host they initially compromised. In real attacks, the gap between initial foothold and final objective is often filled with weeks of lateral movement, credential theft, and privilege escalation. The UKC gives that whole middle section its own macro-stage with its own phases.

---

## 4. The 18 Phases

The UKC organises 18 phases across the three macro-stages. Several phases borrow vocabulary directly from MITRE ATT&CK tactic names, which is deliberate — Pols wanted the UKC to be a bridge framework, not a replacement.

### IN — Initial Foothold

The phases needed to establish a beachhead on the target network:

| Phase | What it covers |
|---|---|
| **Reconnaissance** | External information gathering on the target (OSINT, scanning) |
| **Weaponization** | Building or pairing the attack payload |
| **Social Engineering** | Manipulating users to assist the attack (phishing, vishing) |
| **Delivery** | Transmitting the payload to the target |
| **Exploitation** | Triggering the vulnerability to execute attacker code |
| **Persistence** | Establishing a foothold that survives reboots |
| **Defense Evasion** | Avoiding detection by security controls |
| **Command and Control** | Establishing communication back to the attacker |

### THROUGH — Network Propagation

The phases that the traditional Kill Chain doesn't have:

| Phase | What it covers |
|---|---|
| **Pivoting** | Using the compromised host as a stepping stone |
| **Discovery** | Internal reconnaissance — mapping the network from the inside |
| **Privilege Escalation** | Gaining higher-privileged access on a host |
| **Execution** | Running attacker code on additional hosts |
| **Credential Access** | Harvesting credentials for lateral movement |
| **Lateral Movement** | Spreading to additional hosts using harvested access |

### OUT — Action on Objectives

The phases that complete the attacker's mission:

| Phase | What it covers |
|---|---|
| **Collection** | Gathering the data the attacker came for |
| **Exfiltration** | Moving the data out of the target environment |
| **Impact** | Causing the intended damage (destruction, ransomware, disruption) |
| **Objectives** | The strategic goal achieved (financial, espionage, sabotage) |

Three observations:

1. **The Through stage is structurally distinct.** It contains six phases that have no equivalent in the traditional Kill Chain.
2. **Phase names borrow heavily from MITRE ATT&CK tactics** — Persistence, Defense Evasion, Discovery, Privilege Escalation, Execution, Credential Access, Lateral Movement, Collection, Exfiltration, and Impact are all ATT&CK tactic names. This is the bridging move — UKC vocabulary maps directly onto ATT&CK tactic IDs.
3. **The UKC distinguishes Action on Objectives (a phase) from Objectives (the strategic goal).** Action on Objectives is the act of taking the prize; Objectives is what the attacker ultimately wanted. Subtle but useful distinction in real incident analysis.

---

## 5. The Practical — Task 8: Six Scenario Mappings

The room's practical was a matching exercise. Six attacker behaviour descriptions had to be mapped to the correct UKC phase:

| # | Scenario | Correct phase | Macro-stage | Why this mapping |
|---|---|---|---|---|
| 1 | *"The Attacker uses tools to gather information about a system"* | **Reconnaissance** | In | External information gathering — the standard recon phase |
| 2 | *"The Attacker installs a malicious script to allow them remote access at a later date"* | **Persistence** | In | The defining characteristic of Persistence is "survives later" — the attacker wants access *later*, not immediately |
| 3 | *"The hacked machine is being controlled from an Attacker's own server"* | **Command and Control** | In | Classic C2 — compromised host beaconing back to attacker infrastructure |
| 4 | *"The Attacker uses the hacked machine to access other servers on the same network"* | **Pivoting** | **Through** | Using one compromised host as a stepping stone to reach others — the textbook UKC distinctive phase |
| 5 | *"The Attacker steals a database and sells this to a 3rd party"* | **Action and Objectives** | Out | The attacker achieves their strategic goal — exfiltration plus monetisation |
| 6 | *(Second Action and Objectives scenario)* | **Action and Objectives** | Out | The strategic mission completion phase |

All six mapped correctly. Flag returned: `THM{UKC_SCENARIO}`.

### What the Practical Was Actually Testing

The scenarios were deliberately chosen to probe whether the learner understands **what the UKC adds beyond the traditional Kill Chain**:

- **Reconnaissance** (Q1) — appears in both frameworks; the UKC just confirms baseline familiarity.
- **Persistence** (Q2) — in the traditional Kill Chain this is absorbed into "Installation"; the UKC makes it an explicit phase, and the practical tests whether the learner knows that.
- **Command and Control** (Q3) — appears in both frameworks.
- **Pivoting** (Q4) — **doesn't exist in the traditional Kill Chain at all.** This question is the practical's centrepiece — it tests whether the learner has internalised that the Through stage exists and what it covers.
- **Action and Objectives** (Q5, Q6) — appears in both frameworks but the UKC distinguishes the *action* from the *strategic objective*.

The practical isn't really testing whether you can match six scenarios. It's testing whether you understand **what the UKC adds** — specifically the Persistence/Pivoting/Lateral-Movement granularity that the traditional Kill Chain lacks.

---

## 6. UKC vs Traditional Kill Chain — Side-by-Side

| Aspect | Traditional Kill Chain (2011) | Unified Kill Chain (2017) |
|---|---|---|
| Number of phases | 7 | 18 |
| Macro-structure | Linear sequence | 3 macro-stages with intra-stage looping |
| Lateral movement | Glossed over | Explicit Through stage |
| Insider threat | Poorly modelled | Explicitly accommodated |
| Persistence | Absorbed into Installation | Explicit phase |
| Privilege escalation | Absorbed into Installation | Explicit phase |
| Cloud/SaaS handling | None | Improved (though still imperfect) |
| MITRE ATT&CK alignment | None | Vocabulary borrowed from ATT&CK tactics |
| Industry adoption | Universal household name | Growing — known to mature SOCs and threat-intel teams |

The UKC isn't a replacement for the traditional Kill Chain in casual conversation — most SOC people still say "Kill Chain" meaning Lockheed Martin's. But for serious intrusion analysis, the UKC's added granularity is worth the cost of remembering more phases.

---

## 7. The Bridge to MITRE ATT&CK

This is the connection that makes the UKC genuinely useful and not just "more phases for the sake of it":

| UKC phase | Corresponding MITRE ATT&CK tactic |
|---|---|
| Reconnaissance | TA0043 — Reconnaissance |
| Weaponization | *(no direct ATT&CK equivalent — ATT&CK starts at initial access)* |
| Delivery / Social Engineering | TA0001 — Initial Access |
| Exploitation | TA0001 — Initial Access (often), TA0002 — Execution (often) |
| Persistence | TA0003 — Persistence |
| Defense Evasion | TA0005 — Defense Evasion |
| Command and Control | TA0011 — Command and Control |
| Pivoting | *(no direct ATT&CK equivalent — closest is part of Lateral Movement)* |
| Discovery | TA0007 — Discovery |
| Privilege Escalation | TA0004 — Privilege Escalation |
| Execution | TA0002 — Execution |
| Credential Access | TA0006 — Credential Access |
| Lateral Movement | TA0008 — Lateral Movement |
| Collection | TA0009 — Collection |
| Exfiltration | TA0010 — Exfiltration |
| Impact | TA0040 — Impact |

The UKC effectively gives the Kill Chain's *strategic* lifecycle framing while inheriting MITRE ATT&CK's *tactical* vocabulary. Once you've internalised the UKC, the MITRE room (next in this module) becomes a matter of learning the specific *techniques* underneath tactics you already know.

---

## 8. Where This Sits in the Module

Unified Kill Chain is the **third framework** in the Cyber Defence Frameworks module:

| Position | Room | What it adds |
|---|---|---|
| 1 | Pyramid of Pain | The IOC durability hierarchy — *what to detect* |
| 2 | Cyber Kill Chain | The traditional 7-phase attack lifecycle — *where in the attack* |
| **3** | **Unified Kill Chain** (this room) | **The expanded 18-phase lifecycle covering lateral movement and ATT&CK alignment** |
| 4 | MITRE | The taxonomic catalogue of techniques *under* each UKC/ATT&CK tactic |
| 5 | SUMMIT | The capstone lab applying all of the above |

The three framework rooms together build a layered picture: Pyramid of Pain tells me *how durable* a detection is, Kill Chain/UKC tells me *where in the attack lifecycle* a behaviour sits, and MITRE will tell me *which specific technique* I'm looking at.

---

## 9. Skills Demonstrated

| Skill | Evidence |
|---|---|
| Framework genealogy awareness | Explained the historical context — why Pols developed the UKC as a response to specific Kill Chain limitations |
| Macro-structure thinking | Articulated the In/Through/Out stages and why each exists as a distinct defensive context |
| Cross-framework mapping | Mapped UKC phases directly onto MITRE ATT&CK tactic IDs |
| Practical phase mapping | Correctly identified all six scenarios in Task 8, including the distinctively UKC phases (Persistence, Pivoting) |
| Critical recognition of pedagogy | Identified that the practical specifically tested the phases UKC adds over the traditional Kill Chain, not just generic Kill Chain knowledge |

---

## 10. Honest Reflection

The UKC is genuinely useful, but a few honest caveats:

- **It's less widely adopted than the traditional Kill Chain.** Most SOC people in casual conversation will say "Kill Chain" and mean Lockheed Martin's. In interviews, lead with ATT&CK fluency, use the traditional Kill Chain as your default, and bring up the UKC when you want to demonstrate framework depth.
- **The 18 phases are a lot to hold in working memory.** In practice, I think most analysts internalise the three macro-stages and a handful of phases (Pivoting, Persistence, Lateral Movement) without memorising the full list. The macro-structure does the conceptual work; the specific 18 phases are a reference table.
- **The model still has gaps.** Cloud-native attacks (OAuth abuse, IAM misconfigurations, supply-chain compromises of build pipelines) don't map cleanly onto any kill-chain-style model. The UKC is better at this than the traditional Kill Chain, but neither was built for cloud-first realities.

The next framework in the module — MITRE ATT&CK — addresses the technique-level granularity that even the UKC doesn't reach. For now: the lifecycle framing is internalised, the macro-stages are clear, and the bridge to ATT&CK is built.

---

**Tags:** `#unified-kill-chain` `#paul-pols-2017` `#intrusion-analysis` `#lateral-movement` `#pivoting` `#mitre-attack` `#cyber-defence-frameworks` `#soc-tier-1`