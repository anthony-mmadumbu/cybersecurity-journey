# 🛡️ SOC Metrics and Objectives – TryHackMe

**Path:** SOC Level 1 · SOC Team Internals

## 📅 Date

Day 8

## 📚 What I Learned

- The four core SOC performance metrics — **SLA** (Service Level Agreement), **MTTD** (Mean Time to Detect), **MTTA** (Mean Time to Acknowledge), and **MTTR** (Mean Time to Respond)
- The critical importance of the **False Positive (FP) Rate** — a high FP rate is the single biggest cause of L1 analyst burnout and missed real incidents
- How an L1 analyst contributes to improving these metrics — by feeding observations back into detection engineering, rule tuning, and process refinement
- That metrics tracking is usually the SOC manager's job, but the L1 analyst is **the first person to notice** excessive alert noise or slow response times — so knowing how to communicate the issue is essential
- The distinction between fixing **symptoms** (e.g. "ignore low severity alerts") vs fixing **root causes** (e.g. "tune detection rules to remove system noise")

## 🛠️ Lab — Managing SOC Team Performance Metrics

Imagined as the SOC manager receiving three complaints. For each, I had to identify the **problematic metric**, choose the right **improvement task**, and assign it to the **right person/team**. All three scenarios completed correctly on first attempt with the flag earned for each.

### 1️⃣ Scenario 01 — Unhappy Customer (MTTR Problem)

**Complaint:** OpenDoor Inc.'s CFO had their email + Entra ID account breached. It took the SOC almost **6 hours** to evict the attacker. Most of that — **5 hours** — was spent figuring out how to reset Entra ID password and MFA. Customer is dissatisfied.

**My diagnosis:**

| Category | Selection |
| --- | --- |
| Problematic Metric | Time to Respond was too high, too much time spent to contain the attack |
| Improvement Task | Create a workbook explaining credential rotation steps, and present it to the team |
| Assign Task To | The L2 that handled the incident (research and workbook creation task) |

**Why this works:** The team improvised the credential rotation process during the incident. A workbook built *by the L2 who lived through it* turns that hard-won knowledge into a repeatable procedure for the whole team.

Flag: `THM{mttr:quick_start_but_slow_response}`

### 2️⃣ Scenario 02 — Delayed Alert (MTTD Problem)

**Complaint:** The team contained a ransomware simulation in 40 minutes — impressive — but for the **first 20 minutes**, everyone was watching the screen waiting for alerts to appear. Detection was too slow.

**My diagnosis:**

| Category | Selection |
| --- | --- |
| Problematic Metric | Time to Detect of 20 minutes led to a delayed alert triage |
| Improvement Task | Tune the SIEM and the detection rules to run more often, every 5 minutes |
| Assign Task To | The dedicated SOC engineer (detection rules' schedule review) |

**Why this works:** The detection rule cadence was the bottleneck — the SIEM simply wasn't checking often enough. Tuning the schedule to every 5 minutes shrinks the MTTD window dramatically. Ownership belongs to the SOC engineer because detection rule scheduling is their domain, not L1's.

**Distractors I correctly rejected:**

- ❌ "Ask L1 analysts to periodically review raw logs instead of relying on alerts" — this is a regression, not an improvement. Humans cannot watch raw logs at scale; that's the SIEM's job
- ❌ "Assign log review rule implementation to senior L2/L3" — wrong responsibility; rule schedule review and rule implementation are different tasks

Flag: `THM{mttd:time_between_attack_and_alert}`

### 3️⃣ Scenario 03 — Tired Analysts (FP Rate Problem)

**Complaint:** L1 analysts close **760 alerts per 8-hour shift**, of which **95% are system noise** from IT and automation scripts. They're burning out, and the alert volume keeps growing.

**My diagnosis:**

| Category | Selection |
| --- | --- |
| Problematic Metric | False Positive Rate is the core of the problem |
| Improvement Task | Schedule a call with the team to implement the False Positive remediation process |
| Assign Task To | SOC engineers to exclude the system and IT noise from the rules |

**Why this works:** 95% FP rate is the textbook definition of detection rule noise. The fix is to remove the noise *at the rule level* — not at the analyst level. Rule tuning is SOC engineers' job, not L1's, especially when L1 is already burning out.

**Distractors I correctly rejected:**

- ❌ "Mean Time to Acknowledge is the main issue" — wrong diagnosis; the complaint was about *volume*, not speed
- ❌ "Change triage rules to triage only critical alerts and ignore low severity" — dangerous symptom-hiding. Low severity alerts can chain into major incidents; ignoring them blindly is how attacks slip through
- ❌ "Assign task to the L1 analyst to control and implement the proposed changes" — the L1 analysts are the ones *burning out*; adding work to their plate makes it worse

Flag: `THM{fpr:the_main_cause_of_l1_burnout}`

## 🔐 Why It Matters

- SOC metrics aren't just for management dashboards — they're how a SOC knows whether it's actually working. Bad metrics mean missed incidents and burnt-out analysts
- A high False Positive Rate is the single biggest cause of L1 analyst attrition. Recognising it early and feeding it back to detection engineering is one of the most valuable things an L1 can do
- The L1 analyst is the first to feel the pain of bad metrics, but rarely the one who fixes them. Knowing how to *communicate the issue* — naming the right metric, proposing the right fix, suggesting the right owner — is what gets things actually improved

## ❓ One Thing I Didn't Fully Understand

All three scenarios completed correctly on first attempt. The pattern across them — *diagnose the right metric, propose a root-cause fix, assign to the right owner* — is now a repeatable mental model.

The most useful lesson was that the distractors are designed to look reasonable. "Just triage critical alerts" sounds efficient until you realise you'd be deliberately ignoring early-stage attack indicators. SOC metrics need root-cause thinking, not surface-level workarounds.

---

**Next up:** Continuing through SOC Team Internals