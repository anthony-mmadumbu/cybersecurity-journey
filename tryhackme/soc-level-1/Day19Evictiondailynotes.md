# 🛡️ Eviction – TryHackMe

**Path:** SOC Level 1 · Cyber Defence Frameworks

## 📅 Date

Day 19 · 21 June 2026

## 📚 What I Learned

- How to use the **MITRE ATT&CK Navigator** to load and explore a threat actor's known TTPs (Tactics, Techniques, and Procedures)
- The structure of the **MITRE ATT&CK Matrix** — Tactics as columns (the *why*), Techniques as rows (the *how*)
- How to translate threat intelligence about an adversary (in this case **APT28 / Fancy Bear**) into a structured list of techniques to hunt for inside the network
- The concept of **threat-informed defence** — using known adversary behaviour to drive proactive hunting, rather than waiting for alerts to fire
- How a single technique can map to multiple tactics — e.g. *Spearphishing Link* is used for both **Reconnaissance** and **Initial Access**

## 🛠️ Lab — Threat Hunting APT28 in E-Corp

Played a SOC analyst (Sunny) at E-Corp, a rare earth metals manufacturer. Received a classified intelligence report that **APT28** may be targeting organisations like E-Corp. Used the MITRE ATT&CK Navigator layer for APT28 to identify TTPs to hunt for inside the network.

### Question & Answer Walkthrough

**Q1: What is a technique used by the APT to both perform recon *and* gain initial access?**

✅ **Spearphishing Link**

This technique appears under both Reconnaissance (T1598.003) and Initial Access (T1566.002) — attackers send links to identify targets, and the same vector is used to deliver the actual payload once a victim engages.

**Q2: Which accounts might the APT compromise while developing resources?**

✅ **Email accounts**

Under the Resource Development tactic, APT28 has been observed compromising email accounts (T1586.002) to use as launching points for further phishing or impersonation.

**Q3: Which two techniques of User Execution should Sunny look out for?**

✅ **Malicious file** and **malicious link**

User Execution (T1204) breaks down into sub-techniques including Malicious File (T1204.002) and Malicious Link (T1204.001). Both indicate successful social engineering — the user clicked or opened something the attacker wanted them to.

**Q4: Which scripting interpreters should Sunny search for to identify successful execution?**

✅ **PowerShell** and **Windows Command Shell**

Under the Command and Scripting Interpreter technique (T1059), APT28 commonly uses PowerShell (T1059.001) and Windows Command Shell (T1059.003) for post-exploitation activity. Spotting unusual use of these interpreters following a phishing event is a strong indicator the user execution stage succeeded.

## 🔐 Why It Matters

- MITRE ATT&CK is the **industry standard taxonomy** for adversary behaviour — every SOC, threat intel team, and security vendor uses it. Fluency here is non-negotiable
- ATT&CK Navigator turns abstract threat intel ("APT28 is targeting you") into a concrete hunt list — exactly the kind of pivot a SOC analyst makes when an IOC drops
- Knowing an adversary's preferred techniques lets a SOC **hunt proactively** for evidence of compromise, instead of waiting for the SIEM to fire

## ❓ One Thing I Didn't Fully Understand

All four questions correct on first attempt — earned room completion at 100%. The mapping from threat intelligence → ATT&CK techniques → searchable indicators is now clear and repeatable for any future APT group.

---

**Next up:** Beginning the Phishing Analysis module