# Introduction to SOAR — Portfolio Write-up

**Room:** Introduction to SOAR
**Path:** SOC Level 1 · Core SOC Solutions
**Date completed:** 15 June 2026 (Day 13)
**Completion:** 100% — flag `THM{AUT0M@T1N6_S3CUR1T¥}` earned

---

## 1. Overview

This room introduced **Security Orchestration, Automation, and Response (SOAR)** — the layer of the SOC technology stack that sits *above* the SIEM and EDR, connecting tools and executing playbook-driven responses. Where EDR provides endpoint visibility (Day 9) and SIEM aggregates and correlates logs across the estate (Days 10–12), SOAR is what turns those signals into action without requiring an analyst to click through five consoles to do it.

The practical exercise was a simulated **Threat Intelligence integration playbook** with five configurable stages. The objective was to decide, for each setting, whether it should run automatically or require manual analyst intervention — and to keep iterating until the workflow ran cleanly end-to-end.

---

## 2. The Pain Points SOAR Exists to Solve

Every interview I'll do for a SOC role will include some version of *"what problems does SOAR solve?"*. The honest answer is that the SOC has four chronic problems that compound each other:

| Pain point | What it actually looks like | How SOAR addresses it |
|------------|---------------------------|----------------------|
| **Alert fatigue** | Analysts triaging hundreds of low-priority alerts a day, most of which are false positives or already-known indicators | Auto-enrich and auto-close low-fidelity alerts; surface only what needs a human |
| **Manual processes** | Tier 1 analyst opens VirusTotal, AbuseIPDB, the EDR, the SIEM, the ticketing system, AD, and the firewall in seven tabs to triage one alert | One playbook calls all those tools via API and assembles the result |
| **Tool sprawl** | A mature SOC runs 20–40 security products; getting them to talk to each other natively is rarely possible | Orchestration layer with pre-built integrations replaces glue code and human copy-paste |
| **Communication gaps** | Findings get lost in handoffs between SOC, IT, and management; tickets get stale, escalations get missed | Automated case creation, assignment, communication, and status updates via TheHive, ServiceNow, Slack, email |

These are not theoretical problems. They are the reason the average enterprise SOC has high analyst burnout and turnover — and they are the reason **MTTR (Mean Time to Respond)** is one of the headline SOC metrics. SOAR exists to move that number down.

---

## 3. What SOAR Actually Does — The Three Letters

| Letter | What it means | Concrete example |
|--------|---------------|------------------|
| **S — Orchestration** | Wiring disparate security tools together so they participate in a single workflow | A single playbook step pulls EDR telemetry, queries the SIEM for related events, looks up the IP on VirusTotal, and writes findings back to the case ticket — all via API |
| **A — Automation** | Executing repetitive analyst actions without keyboard input | When an alert fires, automatically enrich the source IP, hash, and domain against threat intelligence feeds; if all three are flagged malicious, auto-create a P2 ticket |
| **R — Response** | Executing containment, eradication, and recovery actions — usually with an approval gate for irreversible steps | Block a malicious domain at the firewall; isolate a compromised host at the EDR; disable an account in AD |

The shorthand I'll carry into interviews: **"SOAR is what turns a SIEM detection into a closed ticket without anyone having to open five other tools."**

---

## 4. Lab Walkthrough — The Threat Intel Workflow

### 4.1 Scenario

A SOC team had recently faced a large breach investigation that took far too long because of a lack of automation. A colleague (McSkidy) advised adopting SOAR and setting up automation playbooks. My task was to work through a five-stage Threat Intelligence integration playbook and decide which settings should be automated and which should remain manual.

### 4.2 The Five Stages and Their Correct Configurations

#### Stage 1 — Case Management Settings

The first stage of any SOAR playbook: when a new alert arrives, what happens to the ticket?

| Setting | Configuration | Reasoning |
|---------|---------------|-----------|
| Create Case Ticket | **Automated** | Every alert needs a ticket; no judgement required |
| Assign Case Ticket | **Automated** | Routing rules are deterministic (by tier, by region, by playbook) |
| Communicate Case Ticket | **Automated** | Email / Slack notifications are templated |
| Update Case Ticket | **Automated** | Status changes triggered by playbook progression |
| **Delete Case Ticket** | **MANUAL** | Destructive and final — needs human accountability |

**Fails before getting this right: 3.**

The lesson hidden in the failures: I'd initially over-automated by toggling Delete to automated. SOAR platforms in the real world will let you do that, but it's almost universally a bad idea — deleting a case ticket erases the audit trail, which is exactly what attackers (or careless analysts) might do to cover their tracks. **Destructive actions belong to humans.**

#### Stage 2 — Threat Intelligence Feeds

How does the playbook get its threat intel data?

| Setting | Configuration | Reasoning |
|---------|---------------|-----------|
| Fetch New Incident Alerts | **Automated** | The whole point of TI integration is continuous polling |
| Set Fetch Intervals | **Automated** | Configured once; runs forever |
| Failed Fetch Notifications | **Automated** | Operational alerting is fire-and-forget |
| **Discard Old Alerts** | **MANUAL** | Same principle as Delete Case Ticket — destroying data needs human accountability |

**Fails before getting this right: 3.**

Same lesson, second time. Auto-deletion of stale alerts is dangerous because "stale" is a judgement call. An old alert about a dormant C2 domain might be exactly what you need three weeks later when that domain suddenly comes back to life.

#### Stage 3 — Incident Data Extraction

When an alert arrives, what IOCs does the playbook pull out of it for further enrichment?

| Setting | Configuration | Reasoning |
|---------|---------------|-----------|
| Extract Domains | **Automated** | Regex extraction — deterministic |
| Extract URLs | **Automated** | Regex extraction — deterministic |
| Extract IPs | **Automated** | Regex extraction — deterministic |
| **Analyst Extraction** | **MANUAL** | This one is for **unknown or novel threats** — anything that doesn't fit a pattern needs human eyes |

**Fails before getting this right: 0.**

I got this on the first try because the pattern was now obvious: automate the rule-based extraction, leave the pattern-breaking cases for humans.

#### Stage 4 — Reputation Checks

This stage is the interesting one — it inverts the pattern.

| Setting | Configuration | Reasoning |
|---------|---------------|-----------|
| **Reputation Results Output** | **AUTOMATED** | Compiling and publishing the result is mechanical |
| Sandbox Testing | **MANUAL** | Choosing whether to detonate a sample in Any.Run or Hybrid Analysis is a judgement call (cost, time, sensitivity) |
| Analyst Validation | **MANUAL** | Confirming reputation results need analyst sign-off because false positives at this stage cascade into bad containment decisions downstream |

**Fails before getting this right: 2.**

The lesson: not every stage of a playbook is automation-heavy. **Reputation is where judgement matters most**, because a wrong call here directly determines whether you block a legitimate service or let a malicious one through. This is why most production SOAR playbooks have human approval gates around reputation conclusions, especially for high-profile blocks.

#### Stage 5 — Course of Action

The containment stage — what the playbook actually does to the threat.

| Setting | Configuration | Reasoning |
|---------|---------------|-----------|
| Block Domains | **Automated** | Reversible; firewall rule change |
| Block IPs | **Automated** | Reversible; firewall rule change |
| Block URLs | **Automated** | Reversible; firewall/proxy rule change |
| Update Case Tickets | **Automated** | Mechanical status update |
| **Analyst Approve COA** | **MANUAL** | Final sign-off before execution — accountability gate |

**Fails before getting this right: 1.**

Note something important here: the **blocks** are automated, but they're gated by **Analyst Approve COA** being manual. This is the canonical SOAR pattern: the heavy lifting is automated, but the human stays in the loop for the moment when "are we really doing this?" needs to be answered. In a real environment, this is the difference between a SOAR platform that helps the SOC and one that takes down production by accident.

### 4.3 Final Result

After 9 total failures across the 5 stages, the workflow ran cleanly end-to-end and the flag was released: `THM{AUT0M@T1N6_S3CUR1T¥}`.

---

## 5. The Pattern Behind the Decisions — The Real Lesson

The lab teaches one principle, and it's the one I'll carry into every SOC interview:

> **Automate deterministic, reversible, low-judgement actions.
> Keep humans in the loop for irreversible, destructive, or judgement-based decisions.**

This sorts cleanly into a 2x2:

|  | **Reversible** | **Irreversible** |
|---|---|---|
| **Deterministic** | ✅ Automate (block IP, create ticket, fetch IOC) | ⚠️ Automate with approval gate (delete records, mass actions) |
| **Judgement-based** | ⚠️ Automate enrichment, manual validation (reputation checks, novel threats) | ❌ Manual (delete ticket, approve COA, discard alerts) |

Mapped to the actual lab settings:

- **Top-left (automate freely):** Create/Assign/Communicate/Update Case Ticket, Fetch/Set/Notify TI Feeds, Extract Domains/URLs/IPs, Block Domains/IPs/URLs
- **Top-right & bottom-left (mixed):** Reputation Results Output (automated) but Sandbox/Validation (manual)
- **Bottom-right (always manual):** Delete Case Ticket, Discard Old Alerts, Analyst Extraction, Analyst Approve COA, Analyst Validation

In production SOAR platforms (Cortex XSOAR, Splunk SOAR, Tines, Microsoft Sentinel Playbooks), this principle is baked into the default playbook templates. The lab forced me to derive it from first principles, which is more valuable than memorising it.

---

## 6. SOAR Tools & Vendors — What I'll See in UK Job Adverts

| Vendor | Product | Notes |
|--------|---------|-------|
| Palo Alto | **Cortex XSOAR** (formerly Demisto) | Market leader; the one to know by name |
| Splunk | **Splunk SOAR** (formerly Phantom) | Tight integration with Splunk SIEM |
| Microsoft | **Sentinel Playbooks** | Built on Azure Logic Apps; common in Microsoft-stack SOCs |
| IBM | **QRadar SOAR** (formerly Resilient) | Common in enterprise QRadar deployments |
| Tines | **Tines** | No-code playbook builder; popular in mid-market |
| Swimlane | **Swimlane Turbine** | Low-code automation platform |

---

## 7. Cross-Module Connections

This room ties directly back to earlier work in the SOC Level 1 path:

- **Day 7 — SOC Workbooks and Lookups:** SOAR playbooks are workbooks executed by software instead of by humans. The decision diamonds I drew by hand in the workbook builder are the same logical structures that exist as branch nodes in a SOAR playbook.
- **Day 8 — SOC Metrics and Objectives:** SOAR is the principal lever for reducing **MTTR (Mean Time to Respond)** and **MTTD (Mean Time to Detect)**. The room makes that link explicit.
- **Day 9 — Introduction to EDR:** SOAR's Response actions (host isolation, process kill, file quarantine) are executed via EDR APIs. SOAR doesn't replace EDR; it operates it.
- **Days 10–12 — SIEM, Splunk, Elastic:** SOAR ingests SIEM detections as its trigger events. The KQL and SPL queries I learned would, in production, populate SOAR playbook inputs.
- **Module on Eviction (APT28):** The containment actions I documented there — blocking C2 domains, disabling compromised accounts, isolating endpoints — are exactly the kinds of actions a real SOAR playbook would orchestrate.

The whole module is now coherent: EDR sees the host, SIEM correlates across the estate, SOAR coordinates the response.

---

## 8. Skills Demonstrated

| Skill | Evidence |
|-------|----------|
| Understanding SOAR's place in the SOC stack | Mapped it to EDR/SIEM hierarchy and to MTTR reduction |
| Playbook design principles | Derived the automate-vs-manual decision framework from first principles via 9 iterations |
| Tool ecosystem awareness | Identified the major SOAR vendors and their market positioning |
| Cross-stack thinking | Connected SOAR to SOC workbooks (Day 7), metrics (Day 8), EDR (Day 9), SIEM tools (Days 10–12), and incident response (Eviction) |
| Honest reflection on failure | Documented 9 failed attempts and what each taught me, rather than pretending I got it right first time |

---

## 9. Honest Reflection

The lab is a simulation, not a real SOAR platform. I configured toggles on a fake control panel, not a real Cortex XSOAR or Splunk SOAR playbook. That's an important caveat for any hiring manager reading this: **I can articulate the principles of SOAR, but I have not yet built a real playbook in a real product.** That's the next gap to close.

My priority before claiming SOAR-adjacent skills in interviews:
1. Build a real playbook in **Tines Community Edition** (free) — a phishing triage flow that extracts the sender domain, queries VirusTotal, and creates a Slack alert.
2. Walk through the **Cortex XSOAR free playbook examples** on the Palo Alto site to understand how toggles I clicked here translate into YAML and integration calls.
3. Map the lab's stages to an actual Cortex XSOAR or Splunk SOAR demo to see the production version of what the room simulated.

The 9 failures during the lab are worth keeping in this write-up rather than hiding. They show the actual learning process — each one corrected a wrong assumption about where to put a human in the loop. Anyone reading this should see that this principle wasn't obvious to me at first and that I had to fail my way into it.

---

**Tags:** `#soar` `#security-orchestration` `#automation` `#playbook` `#threat-intelligence` `#cortex-xsoar` `#splunk-soar` `#tines` `#mttr` `#mttd` `#soc-tier-1` `#incident-response`