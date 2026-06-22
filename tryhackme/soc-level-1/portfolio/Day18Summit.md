# SUMMIT — Climbing the Pyramid of Pain: Detection Engineering Write-up

**Lab Context:** TryHackMe — SOC Level 1 · Cyber Defence Frameworks · SUMMIT (Pyramid of Pain Lab)
**Date completed:** 20 June 2026 (Day 18)
**Role Played:** SOC Analyst / Detection Engineer
**Focus:** Iteratively chasing an adversary up the Pyramid of Pain by building detection rules at each level — Hash Values, IP Addresses, Domain Names, Host Artefacts, Network Tools, and TTPs — using real SOC tooling

---

## Executive Summary

This write-up documents a full six-round detection engineering engagement against "Sphinx", a simulated adversary in TryHackMe's SUMMIT room. Each round, Sphinx delivered a new malware sample designed to evade my previous detection. Each round, I built a detection rule at the next level of the Pyramid of Pain — forcing Sphinx to invest progressively more resources to keep evading.

By the final sample, Sphinx surrendered: *"I have no choice but to completely retrain myself and conduct extensive research to figure out how you're catching me."* That is the exact outcome the Pyramid of Pain model promises — and this lab proves it can be achieved in practice with the right tooling and analytical discipline.

A flag was earned at every level of the pyramid — six flags across all six tiers — culminating in the final TTP-level detection.

---

## Background — The Pyramid of Pain

The Pyramid of Pain (David J. Bianco, 2013) ranks indicators of compromise by how much **pain** they cause an adversary when defenders detect on them. The lower the level, the easier the attacker can rotate the indicator. The higher the level, the more expensive the rotation.

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

The strategic objective of a mature SOC is to write detections as high up the pyramid as the available telemetry supports — because a behavioural rule will outlive every infrastructure rotation underneath it.

---

## Round-by-Round Walkthrough

### Round 1 — sample1.exe · Hash Values (Trivial)

**Sphinx's move:** Delivered the initial sample to the environment.

**My response:** Submitted `sample1.exe` to the **Malware Sandbox**, which extracted its SHA256 hash. Added that hash to the platform's block list.

**Rule logic:**
```
File Hash (SHA256): [sample1 hash] → Action: Block
```

**Sphinx's evasion next round:** Recompiled the binary with a single byte change — produced a completely new hash and bypassed the rule.

**Lesson:** Hash detection is necessary but not sufficient. A SOC that relies on hash blocking alone defeats only attackers who don't bother to recompile — that is, almost none of them.

✅ Flag: `THM{f3cbf08151a11a6a331db9c6cf5f4fe4}`

---

### Round 2 — sample2.exe · IP Addresses (Easy)

**Sphinx's move:** Recompiled the malware. New hash. Same C2 infrastructure.

**My response:** Sandbox detonated `sample2.exe` and observed it beacon to `154.35.10.113`. Created an egress firewall rule:

**Rule logic:**
```
Type: Egress
Source: Any
Destination: 154.35.10.113
Action: Deny
```

The Source field is set to `Any` deliberately — the attacker can rotate which compromised host beacons out, but only the C2 destination matters for this rule.

**Sphinx's evasion next round:** Rotated to a new cloud-hosted IP — moments of work and a few dollars in hosting cost.

**Lesson:** IP-based blocking is operationally useful but defeated by any attacker willing to spend on infrastructure rotation.

✅ Flag: `THM{2ff48a3421a938b388418be273f4806d}`

---

### Round 3 — sample3.exe · Domain Names (Simple)

**Sphinx's move:** New binary, new IP — but the new IP resolved from a domain Sphinx had registered: `emudyn.bresonicz.info`.

**My response:** Created a DNS-level block at the resolver:

**Rule logic:**
```
Category: Malware
Domain: emudyn.bresonicz.info
Action: Deny
```

DNS sinkholing at the resolver catches the request before the connection is even attempted — and works regardless of which IP the domain resolves to in any given hour.

**Sphinx's evasion next round:** Bought a new domain and modified DNS records — slightly more friction than IP rotation because of the registrar transaction, but still cheap.

**Lesson:** Domain blocking is more durable than IP blocking — domain registration leaves a paper trail (registrar records, payment methods) that infrastructure rotation alone doesn't.

✅ Flag: `THM{4eca9e2f61a19ecd5df34c788e7dce16}`

---

### Round 4 — sample4.exe · Host Artefacts (Annoying)

**Sphinx's move:** New binary, new infrastructure, new domain. But the same tooling left a distinctive footprint on the compromised host — specific registry modifications associated with the malware's persistence behaviour.

**My response:** Built a **Sigma Registry Modification rule** that catches the artefact pattern regardless of which binary writes it. The detection is now host-side, not network-side — and survives any number of infrastructure rotations.

**Lesson:** This is the first pyramid level where detection survives a clean infrastructure swap. The attacker now needs to modify their tooling, not just their hosting.

✅ Flag: `THM{46b21c4410e47dc5729ceadef0fc722e}`

---

### Round 5 — sample5.exe · Network Tools (Challenging)

**Sphinx's move:** Modified the tooling to evade the host-side rule. But the underlying custom protocol still beaconed home with a distinctive network signature.

**My response:** Analysed the outgoing connections log alongside the sandbox report. The signature wasn't the IP, port, or domain — it was the **fixed payload size (97 bytes)** combined with a **fixed beacon interval (1800 seconds)**.

**Rule logic (Sigma Network Connection rule):**
```
Remote IP: Any
Remote Port: Any
Size: 97 bytes
Frequency: 1800 seconds
ATT&CK ID: TA0011 (Command and Control)
```

**The breakthrough insight:** when stuck looking for an indicator, look for **what doesn't change** rather than what does. The IP rotates, the port rotates, the domain rotates — but the size and timing of the beacon are baked into the tool's design and would require code-level changes to alter.

✅ Flag: `THM{c956f455fc076aea829799c0876ee399}`

---

### Round 6 — sample6.exe · TTPs (Tough!)

**Sphinx's move:** Rewrote the tooling to vary beacon size and timing. New protocol, new infrastructure, new everything.

**My response:** Examined the command logs from all previous samples. A pattern emerged: every variant of Sphinx's tooling, regardless of how it was modified, eventually wrote exfiltrated data to a file at `%temp%\exfiltr8.log` using `cmd.exe`. This wasn't a tool requirement — it was an **operator habit**. A muscle-memory convention Sphinx repeated across operations.

**Rule logic (Sigma Process Creation rule):**
```
Process Name: cmd.exe
CommandLine: Contains
String: %temp%\exfiltr8.log
ATT&CK ID: TA0010 (Exfiltration)
```

This rule catches the **act**, not the **tool**. Sphinx can swap binaries, infrastructure, protocols, and tooling — the rule still fires as long as the operator keeps writing to that filename. The only way to evade is to change the operator's habits.

**Sphinx's response:** *"I have no choice but to completely retrain myself and conduct extensive research to figure out how you're catching me."*

That sentence is the entire point of the Pyramid of Pain. Defenders win when they force attackers to change how they *think*, not just what they *use*.

✅ Flag: `THM{c8951b2ad24bbcbac60c16cf2c83d92c}` (final flag)

---

## Detection Engineering Summary Table

| Round | Sample | Pyramid Level | Rule Type | ATT&CK Tactic | Flag |
|---|---|---|---|---|---|
| 1 | sample1.exe | Hash Values | Hash block | — | `THM{f3cbf08151a11a6a331db9c6cf5f4fe4}` |
| 2 | sample2.exe | IP Addresses | Egress firewall | — | `THM{2ff48a3421a938b388418be273f4806d}` |
| 3 | sample3.exe | Domain Names | DNS block | — | `THM{4eca9e2f61a19ecd5df34c788e7dce16}` |
| 4 | sample4.exe | Host Artefacts | Sigma Registry Modification | TA0005 Defence Evasion | `THM{46b21c4410e47dc5729ceadef0fc722e}` |
| 5 | sample5.exe | Network Tools | Sigma Network Connection | TA0011 Command and Control | `THM{c956f455fc076aea829799c0876ee399}` |
| 6 | sample6.exe | TTPs | Sigma Process Creation | TA0010 Exfiltration | `THM{c8951b2ad24bbcbac60c16cf2c83d92c}` |

---

## Lessons Learned

1. **The pyramid's geometry is exponential, not linear.** Rotating a hash is a single recompilation command. Rotating an IP is a console click. Rotating a TTP requires retraining the operator. A behavioural rule pays for itself across entire campaigns; a hash rule pays for itself once per file.

2. **Detection engineering is iterative pivoting.** Each round forced me to look at a different telemetry source — sandbox output, network logs, command logs, registry events. A mature SOC has all of these wired into the SIEM; the analytical skill is knowing which source to interrogate when a given rule type fails.

3. **Look for what doesn't change.** Sample5's breakthrough wasn't finding the right indicator — it was abandoning the search for "the right indicator" and looking instead at what was *stubbornly invariant* across infrastructure rotations. Fixed payload size and fixed beacon interval became the signature.

4. **Operator habits are detectable signal.** Sample6's exfiltration filename (`exfiltr8.log`) was an operator habit, not a tool requirement. Real adversaries reuse infrastructure, scripts, and conventions across operations — detecting these habits is how mature SOCs identify the same threat actor across years.

5. **Low false-positive rules earn their place.** A network rule that fires on every long-lived TCP connection is noise. A network rule that fires only on a fixed-size, fixed-interval beacon is signal. Tuning for low FP is what makes detection rules survive in production.

6. **MITRE ATT&CK gives rules a common language.** Mapping each rule to a tactic (TA0005, TA0010, TA0011) supports threat hunting, reporting, and gap analysis. Industry-standard taxonomy is not optional in modern SOC work.

---

## Tools Used in This Engagement

- **Malware Sandbox** — static and dynamic analysis to extract IOCs (hashes, IPs, domains, behavioural signatures)
- **Firewall Manager** — egress IP blocking
- **DNS Manager** — domain blocking at resolution
- **Sigma Rule Builder** — Registry Modification, Network Connection, and Process Creation rule types
- **Cloud instance metadata service** — environment context during platform navigation
- **Network logs / command logs** — supporting evidence for behavioural rules

---

## Frameworks Referenced

- **Pyramid of Pain** (David J. Bianco, 2013) — the strategic model for the entire engagement
- **MITRE ATT&CK** — Defence Evasion (TA0005), Exfiltration (TA0010), Command and Control (TA0011)
- **Sigma** — the open detection rule format used across SIEM platforms

---

## Skills Demonstrated

| Skill | Evidence |
|---|---|
| Detection engineering across the full pyramid | Built rules at every tier from Hash Values to TTPs |
| Sandbox analysis | Extracted IOCs from six malware samples |
| Multi-platform SOC tooling | Used firewall, DNS, Sigma rule builder across the engagement |
| MITRE ATT&CK mapping | Tagged each rule with appropriate tactic ID |
| Analytical patience | Pushed past sample5's strict validation by finding the invariant signature |
| Operator-habit recognition | Identified `exfiltr8.log` as the durable behavioural signature in sample6 |

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*

**Tags:** `#summit` `#pyramid-of-pain` `#detection-engineering` `#sigma` `#mitre-attack` `#purple-team` `#cyber-defence-frameworks` `#soc-tier-1`