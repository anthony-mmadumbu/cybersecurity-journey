# 🛡️ SOC L1 Alert Triage – TryHackMe

**Path:** SOC Level 1 · SOC Team Internals

## 📅 Date

Day 5

## 📚 What I Learned

- The concept of a **SOC alert** — what it is, where it comes from, and how it represents the entry point to investigation
- The key **alert fields** — Status, Severity, Verdict, Assignee, and Analyst Comment — and how each plays a role in the triage workflow
- **Alert statuses** (New, In Progress, Closed) and how they reflect the lifecycle of an investigation
- **Verdict classification** — True Positive, False Positive — and how these decisions drive next steps and detection tuning
- How to perform **L1 alert triage** — reading the alert, gathering context, weighing indicators, and producing a defensible verdict with documentation
- Prepared the foundation for the **SOC Simulator** and the **SAL1 (Security Analyst Level 1)** certification

## 🛠️ Lab — Three Live Alert Triages

Triaged three real SOC alerts, assigned verdicts with supporting analyst notes, and earned the flag — all three correct on first attempt.

### 1️⃣ Double-Extension File Creation → True Positive

File `cats2025.mp4.exe` on HR endpoint, downloaded via Chrome from `freecatvideoshd.monster`. Raised severity Low → High, closed as True Positive. Classic social engineering disguise.

### 2️⃣ Potential Data Exfiltration → False Positive

5.8 GB to `*.zoom.us` from a meeting room IP. Symmetric send/receive pattern = video conference traffic, not exfiltration. Dropped severity Critical → Low, closed as False Positive.

### 3️⃣ Download from GitHub Repository → False Positive

IT team developer (`G.Chandler`) on developer VPN downloading from `facebook/react` — a well-known legitimate open-source project. Closed as False Positive with low severity.

## 🔐 Why It Matters

- Alert triage is the daily reality of L1 SOC work — and the most common interview test for SOC analyst roles
- Three alerts with three different verdict outcomes show that no single rule of thumb works — every alert needs to be read in context

## ❓ One Thing I Didn't Fully Understand

All 3 triages correct on first attempt — earned the flag 🚩

---

**Next up:** SOC L1 Alert Reporting