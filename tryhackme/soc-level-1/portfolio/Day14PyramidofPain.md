# Pyramid of Pain — Portfolio Write-up

**Room:** Pyramid of Pain
**Path:** SOC Level 1 · Cyber Defence Frameworks
**Date completed:** 16 June 2026 (Day 14)
**Flag:** `THM{PYRAMIDS_COMPLETE}`

---

## 1. Overview

This room introduced the **Pyramid of Pain** — a model developed by David J. Bianco in 2013 to rank indicators of compromise (IOCs) by how much pain they cause an adversary when defenders successfully detect on them. It is one of the most-cited models in detection engineering and sits beneath almost every serious conversation about threat hunting, IOC management, and SOC detection strategy.

The room is structured as theory across the six tiers, followed by a hands-on matching practical (Task 9) that confirms understanding by asking the learner to pair attacker-behaviour descriptions with the correct pyramid tier.

---

## 2. The Model

Bianco's central insight is that **not all IOCs are equal**. A SOC that detects on a malware hash is doing something useful, but the attacker can defeat the detection with a single recompile. A SOC that detects on the attacker's *behaviour* has built a rule that survives infrastructure changes, tool changes, and even some operator turnover.

The pyramid ranks indicator types by the attacker's cost to evade them:

```
                        TTPs                  ← Tough!
                  ┌────────────┐
                  │   Tools    │              ← Challenging
                ┌─┴────────────┴─┐
                │ Network/Host   │            ← Annoying
                │   Artifacts    │
              ┌─┴────────────────┴─┐
              │  Domain Names      │          ← Simple
            ┌─┴────────────────────┴─┐
            │   IP Addresses         │        ← Easy
          ┌─┴────────────────────────┴─┐
          │      Hash Values           │      ← Trivial
          └────────────────────────────┘
```

The defender's strategic objective is to detect at the highest level the available telemetry supports — because a rule against attacker behaviour will outlive every infrastructure rotation underneath it.

---

## 3. The Tiers — What Each Means in Practice

### Hash Values (Trivial)

**What they are:** Cryptographic fingerprints of files — MD5, SHA1, SHA256.

**How a SOC detects on them:** Block hashes at the EDR, populate hash lists in the SIEM, ingest hash IOCs from threat intelligence feeds.

**How an attacker evades:** Recompile the binary. Flip a single byte. Append a random comment to a script. Any of these produce a completely different hash and bypass the rule instantly. This is why hash detection is *trivial* to evade — and why it should never be a SOC's only line of defence.

### IP Addresses (Easy)

**What they are:** The numerical addresses of attacker infrastructure — C2 servers, exfiltration endpoints, scanning sources.

**How a SOC detects on them:** Block at the firewall, alert in the SIEM, feed into network detection rules.

**How an attacker evades:** Switch VPS provider, rotate cloud regions, use fast-flux DNS to constantly change the IP behind a domain. The cost is small — at most a few minutes' work plus the price of new infrastructure.

### Domain Names (Simple)

**What they are:** The named addresses of attacker infrastructure — phishing pages, C2 callback domains, fake login portals.

**How a SOC detects on them:** DNS sinkholes, DNS blacklists, proxy block lists.

**How an attacker evades:** Buy a new domain. The cost is the registration fee (typically £5–£15) and DNS propagation time. Slightly more friction than IP rotation because it requires registrar interaction — but still cheap in absolute terms. Notable: domain rotation leaves a paper trail (registrar records, payment methods) that IP rotation often doesn't.

### Network/Host Artifacts (Annoying)

**What they are:** Patterns left behind by the attacker's tooling — distinctive user-agent strings, fixed payload sizes, regular beacon intervals, characteristic registry modifications, named pipes, file paths.

**How a SOC detects on them:** Sigma rules over endpoint telemetry, network detection rules over packet metadata, behavioural alerting in the EDR.

**How an attacker evades:** Modify the tooling's defaults — change the user-agent, randomise beacon intervals, rotate payload sizes. This requires actual development effort and a feedback loop ("did the change work?") that the attacker has to build. This is the tier where evasion stops being "swap a value" and starts being "rewrite a behaviour."

### Tools (Challenging)

**What they are:** The actual software the attacker uses — custom malware families, frameworks like Cobalt Strike or Sliver, post-exploitation tools.

**How a SOC detects on them:** YARA rules for binary patterns, behavioural detections for tool-specific characteristics, threat intelligence on tool families.

**How an attacker evades:** Acquire or build new tooling. The cost is real developer or operator time, possibly the cost of buying access to new frameworks. This is where defenders start meaningfully hurting attacker economics.

### TTPs (Tough!)

**What they are:** The attacker's **Tactics, Techniques, and Procedures** — the *behaviours* themselves, independent of what tool implements them. Examples: lateral movement via remote service execution, credential dumping from LSASS, persistence via scheduled tasks, exfiltration over DNS.

**How a SOC detects on them:** Behavioural analytics, MITRE ATT&CK technique coverage, anomaly detection.

**How an attacker evades:** Retrain the operator. Develop a new playbook. Change deeply ingrained habits. This is months of work, not minutes — and it requires changing how the attacker *thinks*, not just what software they run.

---

## 4. The Practical — Task 9: Matching Exercise

The room's practical was a drag-and-match exercise. Six attacker-behaviour descriptions had to be paired with the correct pyramid tier:

| Description (as given by the room) | Correct tier |
|---|---|
| *"These signatures can be used to attribute payloads and artefacts to an actor."* | **Hash Values** |
| *"These addresses can be used to identify the infrastructure an attacker is using for their campaign."* | **IP Addresses** |
| *"An attacker has purchased this and used it in a typo-squatting campaign."* | **Domain Names** |
| *"These artefacts can present themselves as C2 traffic for example."* | **Network/Host Artifacts** |
| *"The attacker has utilised these to accomplish their objective."* | **Tools** |
| *"The attackers plans and objectives."* | **TTPs** |

All six mapped correctly. Flag returned: `THM{PYRAMIDS_COMPLETE}`.

The matching exercise is mechanically simple, but the precision of the descriptions matters. "*Signatures attributing payloads to an actor*" specifically points at the hash level, not the artefact level, because the room frames the signature as the file's cryptographic fingerprint rather than the file's behaviour. This kind of vocabulary discipline — distinguishing between "the artefact" and "the artefact's signature" — is exactly what a SOC interview will test.

---

## 5. The Asymmetric Cost-to-Rotate — One Idea Worth Surfacing

The room implies but never states directly: **the cost-to-rotate isn't linear across the pyramid. It's roughly exponential.**

| Tier | Concrete cost to rotate |
|---|---|
| Hash Values | A single recompilation command |
| IP Addresses | A VPS provider console click |
| Domain Names | A registration fee and DNS propagation delay |
| Network/Host Artifacts | Tooling modification + testing cycle |
| Tools | Acquire or build new tooling (days to weeks) |
| TTPs | Retrain the operator (weeks to months) |

This asymmetry is the reason detection engineering invests heavily at the top of the pyramid even though writing those rules is harder. A behavioural rule pays for itself across entire attacker campaigns, while a hash rule pays for itself once per file.

This also explains the strategic shift in mature SOCs from **IOC-driven detection** (blocking known-bad indicators as they appear) to **behaviour-driven detection** (alerting on patterns of activity regardless of which infrastructure produces them). The former scales linearly with attacker activity; the latter scales with attacker capability.

---

## 6. Real-World Application

Where the pyramid matters in production SOC work:

- **Threat intelligence consumption.** When a TI feed delivers an indicator, the pyramid tells you how durable the detection will be. A hash IOC needs to fire fast or it ages out within hours. A TTP IOC stays useful for years.
- **Detection rule prioritisation.** When choosing which detection to write next, prefer behavioural rules higher up the pyramid if you have the telemetry to support them.
- **Incident response framing.** Post-incident, the pyramid helps you assess detection maturity: did we catch this at the IOC level (lucky) or at the TTP level (well-built)?
- **Purple teaming.** When testing detections, the pyramid is a coverage map — where are we strong, where are we weak, what would a real attacker exploit?

---

## 7. Where This Sits in the Module

Pyramid of Pain is the first framework in the **Cyber Defence Frameworks** module. The rooms still to come build on it:

| Position | Room | What it adds |
|---|---|---|
| 1 | **Pyramid of Pain** (this room) | The IOC durability hierarchy |
| 2 | Cyber Kill Chain | The sequence of an attack from reconnaissance to objectives |
| 3 | Unified Kill Chain | An expanded, modernised attack lifecycle |
| 4 | MITRE | The taxonomic catalogue of TTPs at the top of the pyramid |
| 5 | SUMMIT | The capstone lab that applies all of the above |

The pyramid is the *strategic* lens — it tells me what to detect and why. The frameworks that follow give me the *tactical* vocabulary — the named techniques, the attack phases, the catalogued behaviours.

---

## 8. Skills Demonstrated

| Skill | Evidence |
|---|---|
| Framework knowledge | Articulated each tier with detection methods and concrete evasion costs |
| Strategic detection thinking | Connected the pyramid to real SOC decisions on detection rule prioritisation |
| Vocabulary discipline | Correctly matched precise descriptions to their corresponding indicator type |
| Cross-framework awareness | Identified open questions about how the pyramid relates to MITRE ATT&CK and D3FEND |

---

## 9. Honest Reflection

The Pyramid of Pain room is short and the practical is light — a 6-item matching exercise. That's appropriate because the framework itself is conceptually compact: a six-tier ranking with an underlying claim about evasion cost.

The harder thing is *applying* the framework — actually writing detections at each tier with real SOC tooling. That's not what this room delivers. The next time I want to demonstrate Pyramid of Pain in practice, I'll need to write actual Sigma rules at each tier or work through a lab that requires building detections, not just naming them.

For now: the framework is internalised. The practical is captured. The flag is logged. On to the next framework in the module.

---

**Tags:** `#pyramid-of-pain` `#bianco-2013` `#detection-engineering` `#ioc` `#ttps` `#cyber-defence-frameworks` `#soc-tier-1`