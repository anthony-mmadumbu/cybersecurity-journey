# SOC Workbooks and Lookups — Building Investigation Playbooks Write-up

**Lab Context:** TryHackMe — SOC Level 1 · SOC Team Internals
**Role Played:** SOC Analyst / Workbook Designer
**Focus:** Building three investigation workbooks from scratch — Email Phishing, PowerShell Download, and Internal Port Scanning — and reasoning about how asset inventory and network diagrams underpin every step

---

## Executive Summary

This write-up documents the design of three SOC investigation workbooks built in TryHackMe's SOC Workbooks Builder, each addressing a distinct and common alert type:

- **Workbook 01** — External Email with Script or Binary Attachment
- **Workbook 02** — Executable File Download using PowerShell
- **Workbook 03** — Port Scanning from Internal IP

All three were completed correctly, each yielding a flag that doubles as the workbook's lesson title — `THM{the_most_common_soc_workbook}`, `THM{be_vigilant_with_powershell}`, and `THM{asset_inventory_is_essential}`. Beyond the lab, the deeper takeaway is the **structural pattern** common to all three: every workbook is built around a single "is this actually bad?" decision, and the value of a SOC analyst is in the work that flows *into* that decision and the action that flows *out* of it.

---

## Background — Why Workbooks and Lookups Matter

A SOC alert is, in isolation, almost meaningless. "An IP at 10.41.12.7 ran PowerShell" is not actionable until you know:

- Whose laptop is `10.41.12.7`? (asset inventory)
- Is that subnet the developer VPN or the finance department? (network diagram)
- Is PowerShell expected on that machine? (asset role / baseline)
- What does the analyst *do next* once they know the answer? (workbook)

**Lookups** (asset inventory, network diagrams, identity directory, threat intel feeds) provide the *context*. **Workbooks** provide the *procedure*. Together, they turn raw alerts into structured investigations.

A useful way to think about workbook design:

| Design Style | Description | Best For |
| --- | --- | --- |
| Many granular workbooks | Hundreds of detailed step-by-step playbooks for every alert type | Mature SOCs with SOAR automation, less reliance on analyst judgment |
| A few high-level workbooks | Modular guides that lean on analyst experience and decision-making | Smaller SOCs, training environments, situations needing flexibility |

Whichever style a SOC adopts, the L1 analyst's job is the same: **divide the investigation into modular blocks and follow the right branch at each decision point**.

---

## Common Pattern Across All Three Workbooks

Before diving into each workbook, it's worth surfacing the **shared skeleton** that emerged across all three:

```
Start
  → Take ownership of the alert + gather context (asset inventory / identity / network map)
  → Analyse the artefact (email, file, scan pattern)
  → [Optional] Build supporting evidence (process tree, SIEM searches, threat intel)
  → DECISION DIAMOND: Is this actually malicious / unexpected / out of policy?
       ├── YES → Gather evidence → Write L2 report → Escalate
       └── NO  → Document reasoning → Tune the detection rule → Close as FP
```

The decision diamond is always a *single, well-phrased yes/no question* — not a vague "is this bad?" The phrasing of that question is the most important design decision in any workbook because it determines what evidence the analyst needs to gather before reaching it.

---

## Workbook 01 — External Email with Script or Binary Attachment

### Alert Scenario

A user receives an external email containing a script or binary attachment. The SIEM has flagged it post-delivery for review.

### Decision Diamond

> **Is the attachment malicious, origin faked, or email unexpected for the recipients' roles?**

This phrasing is excellent because it combines **three independent risk dimensions**:

1. **Is the attachment malicious?** — technical analysis (sandbox, code review)
2. **Is the origin faked?** — authentication checks (SPF, DKIM, sender domain reputation)
3. **Is the email unexpected for the recipients' roles?** — business context (does Finance normally receive scripts?)

Any one of these being true is enough to escalate. All three being false is required to close as FP.

### Workbook Flow

```
Start
  → Take ownership of the alert, use identity inventory to get context of the email recipients
  → Investigate the email using EML analyser: its SPF/DKIM, content, and sender domain reputation
  → Continue to the attachment analysis: use sandbox for binaries and manual code review for scripts
  → Decision (see above)
       ├── YES → Gather triage evidence (recipient list, sandbox report, EML analyser results, other attack indicators)
       │         → Write an alert report for L2 explaining the findings, attach the triage evidence to the report
       │         → Escalate to L2
       └── NO  → Write a short alert comment explaining why you are confident the email is safe and expected
                 → Close as FP
```

### Lookups Used in This Workbook

| Lookup | Purpose |
| --- | --- |
| Identity inventory | Resolves "e.huffman@tryhackme.thm" to "Eddie Huffman, IT Manager" — critical context for assessing whether the email is expected |
| EML analyser | Validates SPF/DKIM and sender reputation |
| Sandbox | Detonates the attachment in isolation |

### Design Notes

The decision to **identify recipients before doing technical analysis** is deliberate — context determines what counts as "expected". A `.ps1` script to a developer is different from the same script to a finance assistant.

**Flag earned:** `THM{the_most_common_soc_workbook}` — fitting because external email with attachments is the single most common alert type a SOC L1 will triage.

---

## Workbook 02 — Executable File Download using PowerShell

### Alert Scenario

A PowerShell process on a corporate endpoint has downloaded an executable file from the internet.

### Decision Diamond

> **Is the file malicious, downloaded from an untrusted resource, or originating from a suspicious process?**

Again, three independent dimensions:

1. **Is the file malicious?** — threat intel + static analysis
2. **Is the source untrusted?** — URL reputation, domain age, hosting service
3. **Is the launching process suspicious?** — parent process / account context

### Workbook Flow

```
Start
  → Assign the alert to yourself, get context of the affected machine using asset inventory
  → Use threat intel to analyse the URL, and perform static analysis of the downloaded executable
  → Build a process tree to find out the parent process and account used to start PowerShell
  → Decision (see above)
       ├── YES → Save the results of the "[WIN] Process Tree" and "[WIN] Login Timeline" SIEM searches
       │         → Assign the alert to the system owner (IT team) and change alert's severity to Critical
       │         → Write an alert report for L2. Attach your evidence, assumptions, and SIEM search results
       │         → Escalate to L2
       └── NO  → Contact the user in alert via email and confirm if the user approves PowerShell usage
                 → Write a brief comment explaining your verdict. Reach SOC engineers to tune the detection rule
                 → Close as FP
```

### Lookups Used in This Workbook

| Lookup | Purpose |
| --- | --- |
| Asset inventory | Whose machine is this? Is PowerShell expected on it? |
| Threat intel feeds | Is the URL reputable? Is the hash known-bad? |
| Process tree (SIEM) | What launched PowerShell? Under which user account? |
| Login timeline (SIEM) | Are there preceding suspicious logins? |

### Design Notes

Two design choices are worth highlighting:

- **The "YES" path adds severity escalation and IT team handover.** Not all positive verdicts are equal — a confirmed malware execution warrants raising severity to Critical and looping in the system owner, not just escalating to L2 in isolation.
- **The "NO" path includes user verification.** PowerShell is a legitimate admin tool. Before closing as FP, the analyst contacts the user to confirm intent — and only then does the workbook recommend tuning the detection rule. This protects against the analyst rationalising away a real attack as "probably the user".

**Flag earned:** `THM{be_vigilant_with_powershell}` — a direct nod to PowerShell's dual nature as a legitimate admin tool and a primary attack vector.

---

## Workbook 03 — Port Scanning from Internal IP

### Alert Scenario

An internal IP address has been observed scanning ports against corporate resources.

### Decision Diamond

> **Is the source IP unlisted (not a known scanner) or does the scanning pattern remind a malicious Discovery attempt?**

This decision is structured to recognise that **port scanning is not inherently malicious** in a corporate environment — vulnerability scanners (Nessus, Zabbix, OpenVAS, Qualys) scan internal IPs continuously as part of normal operations. The decision combines:

1. **Is the source IP listed as a known scanner in the asset inventory?**
2. **Does the *pattern* of the scan match expected vulnerability scanning?**

If the source is unlisted *and* the pattern looks malicious (e.g. targeted at high-value services, off-hours, narrow port range characteristic of reconnaissance), escalate.

### Workbook Flow

```
Start
  → Assign the alert to yourself, get the IPs context from the network map and asset inventory
  → List ports scanned by the IP, find out which services could be running on the ports
  → If source is a corporate Nessus/Zabbix/etc, verify if the scanning pattern is expected
  → Decision (see above)
       ├── YES → Collect your evidence: attack time range, scanned ports, and IP context
       │         → Use EDR to verify if there are any alerts related to data exfiltration attempts
       │         → Scan for open ports on the source IP, find out services that can be running on the ports
       │         → Write an alert report, explain your verdict, and attach the evidence for L2 review
       │         → Escalate to L2
       └── NO  → Write an alert comment, explain your verdict, and reach out to SOC engineers to tune the rule
                 → Close as FP
```

### Lookups Used in This Workbook

| Lookup | Purpose |
| --- | --- |
| Network map | What is this IP? What subnet is it on? |
| Asset inventory | Is it a registered scanner (Nessus, Zabbix, etc.) or an unknown host? |
| EDR | Is the scanning host *also* exhibiting exfiltration symptoms? |
| Port service mapping | Which services are exposed on the scanned ports? |

### Design Notes

The "YES" path's inclusion of **scanning the *source* IP for open ports** is a clever twist — if an internal host is performing reconnaissance, it has likely already been compromised. Pivoting to scan the attacker's foothold helps identify the breach point. This is post-compromise hunting, not just alert triage.

**Flag earned:** `THM{asset_inventory_is_essential}` — and this is the lesson of the entire room. Without asset inventory, the analyst cannot distinguish a corporate vulnerability scanner from a compromised host doing internal reconnaissance. The exact same SIEM signal means two completely different things depending on what the lookup says.

---

## Cross-Workbook Lessons Carried Forward

1. **Every workbook is built around a single, well-phrased decision question.** Designing that question is the most important step. Bad question = bad workbook.
2. **Context-gathering comes before analysis.** Every workbook started with "assign + look up context" — because without context, the analyst can't interpret the technical findings correctly.
3. **The False Positive branch is not a dead end.** Every workbook routes FP findings back into detection engineering (rule tuning) and/or user verification. A SOC that doesn't close the FP loop drowns in repeat noise.
4. **Severity adjustments live inside workbooks.** Workbook 02 explicitly raised severity to Critical inside the YES branch. Adjusting severity is part of the procedure, not an afterthought.
5. **Workbooks are how a SOC scales.** One senior analyst's good investigation becomes every L1's standard process. This is how SOCs achieve consistency under load.

---

## The Role of Lookups — A Closer Look

The room repeatedly returned to two key lookups: **asset inventory** and **network diagrams**. They appear so often because they answer the two questions every alert silently asks:

| Lookup | Question Answered |
| --- | --- |
| Asset inventory | *"What is this IP/hostname and who owns it?"* |
| Network diagram | *"Where in our network is it, and what category of traffic should it generate?"* |

Without these, an alert is a string of bytes. With them, it's a story:

> Alert: "10.41.12.7 ran `whoami /priv` followed by `net group Domain Admins`."
>
> Asset inventory: 10.41.12.7 is Sarah Chen's marketing laptop.
> Network diagram: It sits in the corporate user subnet — no admin privileges, no AD reconnaissance use case.
>
> Verdict: This is not Sarah doing routine admin work. This is a compromise. Escalate.

The narrative is created entirely by the lookups. The SIEM only provided the trigger.

---

## Tools & Frameworks Referenced

- **SOC Workbooks Builder** — interactive workflow design platform
- **EML Analyser** — email forensics (SPF, DKIM, sender reputation, content scoring)
- **Sandbox** — dynamic analysis of attachments and downloaded binaries
- **Threat intelligence feeds** — URL/hash/domain reputation
- **EDR** — endpoint detection and response for post-compromise indicators
- **Asset inventory** — primary lookup for resolving IPs/hostnames to owners and roles
- **Network diagram** — primary lookup for resolving location and category of traffic
- **Identity directory** — primary lookup for resolving email recipients to roles
- **SIEM searches** — process tree and login timeline as evidence sources

## Summary of Workbooks & Flags

| # | Workbook | Decision Question | Flag |
| --- | --- | --- | --- |
| 01 | External Email with Script or Binary Attachment | Is the attachment malicious, origin faked, or email unexpected for the recipients' roles? | `THM{the_most_common_soc_workbook}` |
| 02 | Executable File Download using PowerShell | Is the file malicious, downloaded from untrusted resource, or originating from a suspicious process? | `THM{be_vigilant_with_powershell}` |
| 03 | Port Scanning from Internal IP | Is the source IP unlisted or does the scanning pattern remind a malicious Discovery attempt? | `THM{asset_inventory_is_essential}` |

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*