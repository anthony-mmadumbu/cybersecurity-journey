# SOC Metrics and Objectives — Diagnosing and Improving SOC Performance Write-up

**Lab Context:** TryHackMe — SOC Level 1 · SOC Team Internals
**Role Played:** SOC Manager (diagnosing complaints from customers, leadership, and the team)
**Focus:** Translating three real-world complaints into the correct metric diagnosis, root-cause improvement, and ownership assignment

---

## Executive Summary

This write-up documents three back-to-back SOC performance improvement scenarios — each one structured around a complaint that, on the surface, looked simple but actually tested whether the analyst could (1) identify the correct underlying metric, (2) propose a root-cause fix rather than a surface workaround, and (3) assign the fix to the right team.

All three scenarios were diagnosed correctly on first attempt, with three flags earned. Each flag is also the lesson title — `mttr:quick_start_but_slow_response`, `mttd:time_between_attack_and_alert`, and `fpr:the_main_cause_of_l1_burnout` — naming the metric and the lesson together.

The deeper exercise was learning to **resist plausible-but-wrong fixes** that hide symptoms rather than address root causes. Each of the three scenarios had at least one distractor designed to look reasonable, and recognising why each was wrong is the actual skill the room develops.

---

## Background — The Four Core SOC Metrics

| Metric | Stands For | What It Measures | Owner |
| --- | --- | --- | --- |
| **SLA** | Service Level Agreement | Contractual response and resolution times the SOC commits to (e.g. critical alerts acknowledged within 15 min) | SOC manager + customer |
| **MTTD** | Mean Time to Detect | Time between attack onset and the alert firing | Detection engineering + SIEM tuning |
| **MTTA** | Mean Time to Acknowledge | Time between alert firing and an analyst picking it up | L1 workflow + notification systems |
| **MTTR** | Mean Time to Respond | Time between acknowledgement and containment/resolution | Workbooks + L2/IR procedures |
| **FP Rate** | False Positive Rate | Fraction of alerts that turn out to be benign | Detection engineering + rule tuning |

Each scenario in the lab is engineered to fail on exactly one of these metrics, and the analyst's job is to spot which.

---

## Scenario 01 — Unhappy Customer (MTTR Failure)

### The Complaint

> *"Dear SOC manager, our biggest customer, OpenDoor Inc., was dissatisfied with how we handle breaches. When their CFO's email and Entra ID account were breached, it took us almost 6 hours to kick out the hacker from the mailbox, and threat actors had enough time to dump all emails and leak them on Darknet. Looking at the report, looks like we had a critical alert and spent 5 hours trying to properly reset the victim's Entra ID password and MFA. How could it happen, and what would be your actions?"*

### Reading the Numbers

| Metric | Observation | Verdict |
| --- | --- | --- |
| MTTD | Critical alert *did* fire | ✅ Not the problem |
| MTTA | Implicit — alert was picked up promptly | ✅ Not the problem |
| MTTR | 5 hours on credential reset + 6 hours total for eviction | ❌ The problem |

The detection layer worked. The response layer failed. The team didn't know *how* to perform Entra ID credential rotation under pressure, so they invented the process during the incident — which is the slowest possible way to do anything.

### My Diagnosis

| Category | Selection | Reasoning |
| --- | --- | --- |
| Problematic Metric | Time to Respond was too high, too much time spent to contain the attack | Names the right metric (MTTR) and links it to the observed behaviour |
| Improvement Task | Create a workbook explaining credential rotation steps, and present it to the team | Root-cause fix — captures the missing procedure as a repeatable artefact |
| Assign Task To | The L2 that handled the incident (research and workbook creation task) | The person who learned the lesson the hard way is best placed to document it |

### Flag

`THM{mttr:quick_start_but_slow_response}` — the flag is also a one-line summary of the OpenDoor incident: detection was fine, response was the failure.

### Lesson

When a metric fails, **document the procedure that should have existed** — and have it written by the person who lived through the failure. That captures hard-won context that a generic playbook can't.

---

## Scenario 02 — Delayed Alert (MTTD Failure)

### The Complaint

> *"Hey, thanks for the SOC demo for our top management. They loved your ransomware simulation and were shocked at how your team managed to stop the attack in 40 minutes. However, for the first 20 minutes, everyone was just looking at the screen, waiting for some alerts to appear. It would be nice to somehow reduce this huge delay, what do you think?"*

### Reading the Numbers

| Metric | Observation | Verdict |
| --- | --- | --- |
| MTTD | 20-minute gap between attack onset and the first alert | ❌ The problem |
| MTTA | Once alerts fired, analysts engaged immediately | ✅ Not the problem |
| MTTR | Containment in 20 minutes after detection (40 total) | ✅ Strong |

The team's response was excellent *once they had a signal*. The signal arrived too late.

### My Diagnosis

| Category | Selection | Reasoning |
| --- | --- | --- |
| Problematic Metric | Time to Detect of 20 minutes led to a delayed alert triage | Correctly names MTTD as the bottleneck |
| Improvement Task | Tune the SIEM and the detection rules to run more often, every 5 minutes | Root-cause fix — the rule cadence was too slow |
| Assign Task To | The dedicated SOC engineer (detection rules' schedule review) | Detection rule scheduling is the SOC engineer's domain, not L1's |

### Distractors I Rejected

| Distractor | Why It's Wrong |
| --- | --- |
| Ask L1 analysts to periodically review raw logs instead of relying on alerts | This is a regression. Humans cannot watch raw logs at scale — that's the SIEM's purpose. Asking L1 to do this is admitting the detection layer doesn't work, and burning out the analysts in the process |
| Assign log review rule implementation to senior L2/L3 | Wrong responsibility. Rule *implementation* and rule *schedule review* are different tasks. The schedule fix doesn't require L2/L3 development effort |

### Flag

`THM{mttd:time_between_attack_and_alert}`

### Lesson

When detection is too slow, **fix the detection layer's cadence**, don't shift the work to humans. The right fix lives at the rule layer, not the analyst layer.

---

## Scenario 03 — Tired Analysts (FP Rate Failure)

### The Complaint

> *"Dear SOC manager, on behalf of all L1 analysts, I want to raise an issue that may require your help. On average, during an 8-hour shift, our L1 analysts close 760 alerts, 95% of which is system noise from our IT team or automation scripts. It is impossible to perform a vigilant triage with such a big load, and analysts are starting to get exhausted. Moreover, as the company grows, we receive more and more alerts. Can you help us with it, please?"*

### Reading the Numbers

| Observation | Implication |
| --- | --- |
| 760 alerts per 8-hour shift | ~95 alerts per analyst per hour — one every 38 seconds |
| 95% are system noise | Roughly 722 of 760 are false positives |
| Trend: growing with company size | The problem compounds linearly without intervention |

This is the textbook definition of **alert fatigue driven by an unsustainable False Positive Rate**.

### My Diagnosis

| Category | Selection | Reasoning |
| --- | --- | --- |
| Problematic Metric | False Positive Rate is the core of the problem | Names the actual cause — 95% FP rate is not an MTTA issue, it's an FPR issue |
| Improvement Task | Schedule a call with the team to implement the False Positive remediation process | Root-cause fix — remediate the noise sources, not the symptoms |
| Assign Task To | SOC engineers to exclude the system and IT noise from the rules | Rule tuning is SOC engineers' work. The L1 analysts cannot be the ones to fix what's drowning them |

### Distractors I Rejected (the most important section of this scenario)

| Distractor | Why It's Wrong |
| --- | --- |
| Mean Time to Acknowledge is the main issue | Wrong diagnosis. The complaint described *volume*, not *speed*. Analysts aren't slow — they're overwhelmed by noise. MTTA may even *look fine* in this scenario because analysts are speed-running through dismissals to clear the queue |
| Change the triage rules to triage only critical alerts and ignore low-severity ones | **Dangerously plausible.** This hides the symptom (high alert volume) while making the underlying problem catastrophic. Low-severity alerts are often the *first signal* of a larger attack chain — port scans precede exploits, beaconing precedes data theft. Ignoring them blindly is how breaches happen |
| Assign the task to the L1 analyst to control and implement the proposed changes | The L1 analysts are the people *burning out*. Adding "build and implement the FP remediation process" to their workload accelerates the very problem the email is describing. Rule tuning belongs to SOC engineers, not the people drowning in the rules' output |

### Flag

`THM{fpr:the_main_cause_of_l1_burnout}` — and this flag deserves emphasis: industry research consistently shows that **high FP rate is the #1 cause of SOC analyst attrition**, ahead of pay, hours, and management issues. Recognising and fighting alert fatigue is one of the most career-impactful things a SOC leader can do.

### Lesson

When analysts are drowning, **the answer is to fix the rules, not the analysts**. Asking the burnt-out team to fix the cause of their burnout is the worst possible response.

---

## Cross-Scenario Lessons Carried Forward

1. **Diagnose the metric first, then the fix.** Each scenario tested whether the analyst could *name the right metric* before proposing a solution. Get the diagnosis wrong, and every "fix" downstream is also wrong.
2. **Distractors are designed to look reasonable.** Every scenario had at least one plausible-sounding wrong answer. "Ignore low severity alerts", "ask L1s to review raw logs", "let L1 implement the fix" — all sound reasonable; all are catastrophic in practice. Recognising the pattern of *plausible-but-wrong* is a transferable skill.
3. **Symptom vs root-cause thinking is the SOC analyst's superpower.** A SOC that fixes symptoms gets buried in the same incidents repeatedly. A SOC that fixes root causes gets quieter over time.
4. **Ownership matters as much as the action.** The right fix assigned to the wrong owner often doesn't get done — or gets done badly. Workbooks belong with the L2 who learned the lesson; rule tuning belongs with SOC engineers; rule cadence reviews belong with the dedicated engineer. Detail of ownership is part of the design.
5. **Metrics drive culture.** A team measured on MTTA alone will close alerts fast and shallow. A team measured on FPR alone will under-report alerts to keep the rate down. The right metrics suite — SLA, MTTD, MTTA, MTTR, and FPR balanced together — drives the right behaviour.

---

## The L1 Analyst's Role in Metrics Improvement

Even though metrics tracking is the SOC manager's job, **the L1 is the early warning system**:

- An L1 closing alerts at unusual volume → potential FP problem
- An L1 picking up alerts late after hours → potential MTTA / notification gap
- An L1 finding the same kind of FP repeatedly → tunable detection rule
- An L1 escalating but never seeing resolution → potential MTTR / handover gap

The L1's *job* is the alerts. The L1's *contribution to the metrics* is communicating what's not working. This room's biggest takeaway is that **knowing how to phrase that communication** — naming the right metric, proposing the right fix, suggesting the right owner — is what gets things actually improved.

---

## Tools & Frameworks Referenced

- **SOC performance metrics**: SLA, MTTD, MTTA, MTTR, FP Rate
- **NIST SP 800-61** (Incident Response Lifecycle — the source of detect/respond timing terminology)
- **Root-cause analysis** as a SOC management discipline
- **Detection engineering / rule tuning** as the SOC's quality control function

## Summary of Scenarios & Flags

| # | Scenario | Failing Metric | Fix Owner | Flag |
| --- | --- | --- | --- | --- |
| 01 | Unhappy Customer | MTTR | L2 who handled the incident | `THM{mttr:quick_start_but_slow_response}` |
| 02 | Delayed Alert | MTTD | Dedicated SOC engineer | `THM{mttd:time_between_attack_and_alert}` |
| 03 | Tired Analysts | FP Rate | SOC engineers | `THM{fpr:the_main_cause_of_l1_burnout}` |

All three scenarios completed correctly on first attempt.

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*