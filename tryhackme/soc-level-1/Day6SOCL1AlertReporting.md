# 🛡️ SOC L1 Alert Reporting – TryHackMe

**Path:** SOC Level 1 · SOC Team Internals

## 📅 Date

Day 6

## 📚 What I Learned

- The need for **SOC alert reporting and escalation** — why a report is more than just an analyst comment, and when to involve L2
- How to write **alert comments and case reports properly** — using a 5W structure (When, Who, What, Why suspicious, Why escalating)
- **Escalation methods and communication best practices** — assigning to the right tier, justifying the handover, and giving L2 everything they need to continue without re-investigating
- The difference between **closing an alert yourself** (L1 scope) versus **escalating to L2** (deeper analysis required, e.g. malware detonation, threat hunting)
- The role of **Authenticity validation** in modern SOC tooling — reports are now scored on whether they look like genuine analyst work

## 🛠️ Lab — Two Live Alert Reports & Escalations

Both alerts triaged, reported, and escalated correctly to L2 (E.Fleming). Both reports scored **100% Authenticity** by the system. Earned the flag on both.

### 1️⃣ Email Marked as Phishing after Delivery → True Positive · Escalated to L2

Phishing email to IT Manager Eddie Huffman, sent from a spoofed `support@microsoft.com` (failed SPF and DKIM), containing a `REPORT.rar` attachment with classic phishing keywords (600% price increase, urgent notice). Severity raised Medium → Critical, escalated to L2 for attachment detonation analysis.

### 2️⃣ Spike of Domain Discovery Commands → True Positive · Escalated to L2

Suspicious process tree on `DMZ-MSEXCHANGE-2013` running as `NT AUTHORITY\SYSTEM`: `w3wp.exe` (IIS) → `revshell.exe` → `cmd.exe` executing Active Directory reconnaissance (`whoami`, `net group "Domain Admins"`, `nltest`). Severity raised Medium → Critical, escalated to L2 for deeper analysis of likely web shell / reverse shell compromise.

## 🔐 Why It Matters

- Escalation reports are the deliverable a real SOC manager reads to evaluate L1 analysts — the quality of the report determines the speed of the response
- Knowing when to escalate (and writing the report well) is what differentiates L1 analysts who get promoted from those who don't

## ❓ One Thing I Didn't Fully Understand

Both alerts triaged and escalated correctly with 100% Authenticity reports — earned the flag 🚩

The second alert introduced me to the **process tree / spawning child process** concept. I correctly identified it as a major red flag based on preliminary research, but escalated to L2 to learn the deeper technical detail. Researched after: an IIS worker process (`w3wp.exe`) spawning a binary from `C:\Users\Public\` and then `cmd.exe` running domain reconnaissance is a classic web shell → reverse shell → privilege escalation chain. Will study Windows process tree analysis further.

---

**Next up:** SOC Workbooks and Lookups