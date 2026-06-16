# 🛡️ Pyramid of Pain – TryHackMe

## 📅 Date
Day 14 · 16 June 2026

## 📚 What I Learned (3–5 lines max)
- The **Pyramid of Pain** — David J. Bianco's 2013 model that ranks indicators of compromise (IOCs) by how much "pain" they cause an adversary when defenders successfully detect on them.
- Six tiers, bottom to top: **Hash Values (Trivial) → IP Addresses (Easy) → Domain Names (Simple) → Network/Host Artifacts (Annoying) → Tools (Challenging) → TTPs (Tough!)**.
- The model's central claim: detection rules higher up the pyramid survive more attacker evasion because the **cost to rotate** the indicator grows with each tier.
- Mature SOC detection engineering aims to push detection logic as high up the pyramid as the available telemetry supports — that's the difference between catching IOCs (reactive) and catching behaviours (resilient).

## 🛠️ What I Did
- Worked through the theory tasks covering each tier of the pyramid with worked examples for each indicator type.
- Completed **Task 9 — Practical: The Pyramid of Pain** — a six-item matching game pairing attacker behaviour descriptions to the correct pyramid tier:
  - *"Signatures attributing payloads to an actor"* → **Hash Values**
  - *"Addresses identifying attacker infrastructure"* → **IP Addresses**
  - *"Purchased and used in a typo-squatting campaign"* → **Domain Names**
  - *"Artefacts presenting as C2 traffic"* → **Network/Host Artifacts**
  - *"Used to accomplish their objective"* → **Tools**
  - *"Plans and objectives"* → **TTPs**
- All six mapped correctly. Earned flag: `THM{PYRAMIDS_COMPLETE}`.

## 🔐 Why It Matters
- The Pyramid of Pain is the foundational mental model for **detection engineering**. Without it, "detect IOCs" is a flat instruction — with it, you understand that not all IOCs are equal and that climbing the pyramid is a deliberate strategic objective.
- Knowing this framework by heart is interview-table-stakes for SOC Analyst roles. Any conversation about detection engineering, threat hunting, or IOC management implicitly sits on top of it.

## ❓ One Thing I Didn't Fully Understand
- How the Pyramid of Pain interacts with **MITRE ATT&CK's data sources** and **D3FEND's defensive technique catalogue**. The pyramid tells me *where* to detect; ATT&CK tells me *what behaviours* exist at the top tier; D3FEND tells me *what response actions* map to them. Worth mapping these three frameworks against each other as I get deeper into the Cyber Defence Frameworks module.