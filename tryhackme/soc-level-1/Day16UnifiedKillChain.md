# 🛡️ Unified Kill Chain – TryHackMe

## 📅 Date
Day 16 · 19 June 2026

## 📚 What I Learned (3–5 lines max)
- The **Unified Kill Chain (UKC)** — developed by **Paul Pols in 2017** as part of his MSc thesis at Leiden University, in collaboration with Fox-IT. A deliberate fix for the limitations of Lockheed Martin's traditional Kill Chain.
- Structurally bigger and richer: **18 phases organised into 3 macro-stages — In (Initial Foothold), Through (Network Propagation), Out (Action on Objectives).** The traditional Kill Chain had no equivalent of the Through stage at all.
- UKC is essentially the bridge between Lockheed Martin's Kill Chain and **MITRE ATT&CK** — it borrows ATT&CK-style tactic names (Persistence, Pivoting, Lateral Movement, Privilege Escalation, Execution) and makes them explicit lifecycle phases.
- The model handles three things the traditional Kill Chain couldn't: **lateral movement**, **insider threats**, and **non-linear attack paths** (attackers looping back through recon/weaponization from inside the network).

## 🛠️ What I Did
- Worked through the theory of all 18 UKC phases and the rationale for each of the three macro-stages.
- Completed **Task 8 — Practical** — matched attacker actions to the correct UKC phase across six scenarios:
  - *"The Attacker uses tools to gather information about a system"* → **Reconnaissance** (In)
  - *"The Attacker installs a malicious script to allow them remote access at a later date"* → **Persistence** (In)
  - *"The hacked machine is being controlled from an Attacker's own server"* → **Command and Control** (In)
  - *"The Attacker uses the hacked machine to access other servers on the same network"* → **Pivoting** (Through)
  - *"The Attacker steals a database and sells this to a 3rd party"* → **Action and Objectives** (Out)
  - Plus one final scenario also resolving to **Action and Objectives**.
- All matched correctly. Earned flag: `THM{UKC_SCENARIO}`.

## 🔐 Why It Matters
- The UKC is the **bridge framework** between intrusion-analysis frameworks (Kill Chain) and technique-cataloguing frameworks (MITRE ATT&CK). Knowing it shows depth — most candidates only know the traditional Kill Chain by name.
- The explicit **Pivoting** and **Lateral Movement** phases are how modern SOCs actually think about intrusion progression. Network segmentation, micro-segmentation, and east-west traffic monitoring all map directly onto the Through macro-stage — concepts that have no home in the traditional Kill Chain.

## ❓ One Thing I Didn't Fully Understand
- The exact boundary between **Persistence** (an In-stage phase) and **Lateral Movement** (a Through-stage phase) when an attacker installs persistence on multiple hosts as part of moving laterally. Real attacks blur these phases — an attacker who pivots, escalates privileges, and installs a backdoor on the new host is technically performing Persistence, Pivoting, Privilege Escalation, and Lateral Movement simultaneously. Worth working through with a real incident write-up to see how analysts handle this in practice.