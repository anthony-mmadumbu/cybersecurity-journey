# 🛡️ Introduction to SOAR – TryHackMe

## 📅 Date
Day 13 · 15 June 2026

## 📚 What I Learned (3–5 lines max)
- **SOAR = Security Orchestration, Automation, and Response** — the layer above SIEM and EDR that connects disparate tools, automates repetitive tasks, and executes playbook-driven responses.
- The traditional SOC challenges SOAR addresses: **alert fatigue**, **manual processes**, **tool sprawl**, and **communication gaps** across IT and management teams.
- A **playbook** is a flowchart of automated steps and human approval gates triggered by an alert — the structure is identical to the SOC workbooks I built back on Day 7, just executed by software.
- The decision of what to automate vs leave manual follows a clear principle: **automate deterministic, reversible actions; keep humans in the loop for irreversible or judgment-based decisions.**

## 🛠️ What I Did
- Worked through the Threat Intel Workflow Practical — a 5-stage simulated SOAR playbook covering Case Management, Threat Intelligence Feeds, Incident Data Extraction, Reputation Checks, and Course of Action.
- Configured automation toggles across all 5 stages, identifying which settings should be automated and which should stay manual.
- Made 9 errors in total before getting the workflow right (3 on Case Management, 3 on Threat Intel Feeds, 0 on Data Extraction, 2 on Reputation Checks, 1 on Course of Action) — each error taught me something about where human judgement still belongs in the loop.
- Completed the room at 100% and earned flag `THM{AUT0M@T1N6_S3CUR1T¥}`.

## 🔐 Why It Matters
- SOAR is the layer that directly improves **MTTR (Mean Time to Respond)** — the SOC metric I documented on Day 8. Without SOAR, Tier 1 analysts spend most of their time clicking between consoles; with it, they spend their time on the decisions that actually require analysis.
- The lab made the principle of "automate enrichment, gate containment" tangible — this is exactly how real Cortex XSOAR and Splunk SOAR playbooks are designed.
- The Delete Case Ticket / Discard Old Alerts / Analyst Approve COA toggles being **manual** is a real-world safeguard: destructive or final actions need a human signature even in highly automated environments. This maps directly to NIST 800-53 controls around separation of duties and accountability.

## ❓ One Thing I Didn't Fully Understand
- The lab simulated playbook *settings* but didn't show me what a real playbook actually looks like under the hood — the YAML, the action blocks, the integration calls. I need to look at a real Cortex XSOAR or Tines playbook example next, so I understand how the toggles I clicked here translate into actual configuration in production.