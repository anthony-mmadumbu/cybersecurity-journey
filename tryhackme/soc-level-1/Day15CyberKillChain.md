# 🛡️ Cyber Kill Chain – TryHackMe

## 📅 Date
Day 15 · 18 June 2026

## 📚 What I Learned (3–5 lines max)
- The **Cyber Kill Chain** — Lockheed Martin's 2011 adaptation of the military "kill chain" concept to cyberspace. It models a successful intrusion as a sequence of seven phases an adversary must complete to achieve their objective.
- The seven phases: **Reconnaissance → Weaponization → Delivery → Exploitation → Installation → Command & Control → Actions on Objectives.**
- The defender's strategic advantage: **break the chain at any phase and the attack fails.** Defenders only need to win once per attack; attackers need to win every phase.
- The model's limitations: too perimeter-focused, too linear, weak on insider threats, and predates cloud/SaaS realities — which is why the **Unified Kill Chain** (Pols, 2017) was developed.

## 🛠️ What I Did
- Worked through the theory of all seven Kill Chain phases with worked examples for each.
- Completed **Task 9 — Practice Analysis** — a real-world scenario based on the **2013 Target data breach** (40 million payment card records stolen, $18.5M settlement). Had to map six attacker techniques to the correct Kill Chain phases:
  - *Powershell* → **Weaponization**
  - *Spearphishing attachment* → **Delivery**
  - *Exploit public-facing application* → **Exploitation**
  - *Dynamic linker hijacking* → **Installation**
  - *Fallback channels* → **Command & Control**
  - *Data from local system* → **Actions on Objectives**
- All six mapped correctly. Earned flag: `THM{7HR347_1N73L_12_4w35om3}`.

## 🔐 Why It Matters
- The Kill Chain is the foundational mental model for **intrusion analysis** — it's how SOC analysts, threat hunters, and incident responders think about where an attack is in its lifecycle and which defensive controls apply at each stage.
- It maps cleanly onto the **Pyramid of Pain** I covered yesterday — IOCs (hashes, IPs, domains) cluster at Delivery and C2 phases, while TTPs span Installation, C2, and Actions on Objectives. The two frameworks together give you both *where* in the attack lifecycle and *how durable* your detection will be.

## ❓ One Thing I Didn't Fully Understand
- The mapping of "PowerShell" to **Weaponization** in the Target breach practical felt forced. In real MITRE ATT&CK terms, PowerShell (T1059.001) sits in the **Execution** tactic, not Weaponization. The room appears to be teaching that an attacker can *weaponize* PowerShell as their attack tool, but the cleaner mapping would have been to Installation or Execution. Worth flagging as a vocabulary inconsistency between the Kill Chain and MITRE ATT&CK that I'll need to navigate carefully in interviews.