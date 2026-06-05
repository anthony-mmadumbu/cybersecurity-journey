# 🛡️ SOC Workbooks and Lookups – TryHackMe

**Path:** SOC Level 1 · SOC Team Internals

## 📅 Date

Day 7

## 📚 What I Learned

- The role of **SOC investigation workbooks** — modular, repeatable playbooks that guide an L1 analyst through the right sequence of actions for a given alert type
- The spectrum of workbook design — from **hundreds of granular workbooks** (more like SOAR automation playbooks) to **a few high-level guides** that rely more on analyst experience and decision-making
- Where to find and how to use the **asset inventory** in a SOC investigation — for resolving IPs/hostnames to owners, asset roles, and business context
- The importance of **corporate network diagrams** — knowing which subnets are which, where the DMZ is, what's in a VPN, and which IPs map to scanners vs endpoints
- How to **divide an investigation into modular blocks** and build simple workbooks around them — branching on key decision points like "is this malicious?"
- Practical workflow building inside an interactive workbook builder — assembling drag-and-drop steps into a coherent investigation flow

## 🛠️ Lab — Workbooks Practice

Three workbooks built from scratch in the SOC Workbooks Builder, each for a different alert scenario. All three completed correctly and the corresponding flags claimed.

### 1️⃣ Workbook 01 — External Email with Script or Binary Attachment

**Scenario:** Phishing email with a suspicious attachment.

**Flow built:**

```
Start
  → Take ownership of the alert, use identity inventory to get context of the email recipients
  → Investigate the email using EML analyser: its SPF/DKIM, content, and sender domain reputation
  → Continue to the attachment analysis: use sandbox for binaries and manual code review for scripts
  → Decision: Is the attachment malicious, origin faked, or email unexpected for the recipients' roles?
       ├── YES → Gather triage evidence (recipient list, sandbox report, EML analyser results, other attack indicators)
       │         → Write an alert report for L2 explaining the findings, attach the triage evidence to the report
       │         → Escalate to L2
       └── NO  → Write a short alert comment explaining why you are confident the email is safe and expected
                 → Close as FP
```

Flag: `THM{the_most_common_soc_workbook}`

### 2️⃣ Workbook 02 — Executable File Download using PowerShell

**Scenario:** PowerShell downloading an executable from the internet.

**Flow built:**

```
Start
  → Assign the alert to yourself, get context of the affected machine using asset inventory
  → Use threat intel to analyse the URL, and perform static analysis of the downloaded executable
  → Build a process tree to find out the parent process and account used to start PowerShell
  → Decision: Is the file malicious, downloaded from untrusted resource, or originating from a suspicious process?
       ├── YES → Save the results of the "[WIN] Process Tree" and "[WIN] Login Timeline" SIEM searches
       │         → Assign the alert to the system owner (IT team) and change alert's severity to Critical
       │         → Write an alert report for L2. Attach your evidence, assumptions, and SIEM search results
       │         → Escalate to L2
       └── NO  → Contact the user in alert via email and confirm if the user approves PowerShell usage
                 → Write a brief comment explaining your verdict. Reach SOC engineers to tune the detection rule
                 → Close as FP
```

Flag: `THM{be_vigilant_with_powershell}`

### 3️⃣ Workbook 03 — Port Scanning from Internal IP

**Scenario:** An internal IP performing port scans against corporate resources.

**Flow built:**

```
Start
  → Assign the alert to yourself, get the IPs context from the network map and asset inventory
  → List ports scanned by the IP, find out which services could be running on the ports
  → If source is a corporate Nessus/Zabbix/etc, verify if the scanning pattern is expected
  → Decision: Is the source IP unlisted or does the scanning pattern remind a malicious Discovery attempt?
       ├── YES → Collect your evidence: attack time range, scanned ports, and IP context
       │         → Use EDR to verify if there are any alerts related to data exfiltration attempts
       │         → Scan for open ports on the source IP, find out services that can be running on the ports
       │         → Write an alert report, explain your verdict, and attach the evidence for L2 review
       │         → Escalate to L2
       └── NO  → Write an alert comment, explain your verdict, and reach out to SOC engineers to tune the rule
                 → Close as FP
```

Flag: `THM{asset_inventory_is_essential}`

## 🔐 Why It Matters

- Workbooks turn one good analyst's investigation method into the **whole SOC's standard playbook** — they're how SOCs scale consistency
- Asset inventory and network diagrams are the lookups that make every workbook usable. Without them, an alert is just an IP address; with them, it's "the marketing intern's laptop on the corporate VPN"
- The branching FP path in each workbook — feeding back to SOC engineers to tune the detection rule — is mature SOC thinking. Every false positive is an opportunity to improve detection, not just close a ticket

## ❓ One Thing I Didn't Fully Understand

All three workbooks completed correctly — earned a flag at each step. The pattern across all three (assign → context → analyse → branch on verdict → escalate or tune) is now clear and repeatable for future alert types.

---

**Next up:** SOC Metrics and Objectives