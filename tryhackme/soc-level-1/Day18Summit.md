# 🛡️ SUMMIT – TryHackMe

**Path:** SOC Level 1 · Cyber Defence Frameworks (Pyramid of Pain Lab)

## 📅 Date

Day 18 · 20 June 2026

## 📚 What I Learned

- Practical application of the **Pyramid of Pain** — climbing every level from Hash Values up to TTPs
- How each pyramid level corresponds to a different defensive tool and rule type — and why each higher level costs the attacker more to evade
- Hands-on use of multiple SOC tools in a simulated environment: **Malware Sandbox**, **Firewall Manager**, **DNS Rule Manager**, **Sigma Rule Builder** (Registry Modification, Network Connection, Process Creation rule types)
- Mapping detection rules to **MITRE ATT&CK tactics** — e.g. TA0005 Defence Evasion, TA0010 Exfiltration, TA0011 Command and Control
- The difference between **specific indicators** (IPs, hashes — easy to rotate) and **behavioural patterns** (TTPs — expensive to change)
- The role of **detection engineering** within a purple-team engagement — writing rules that survive infrastructure changes by the attacker

## 🛠️ Lab — Chasing Sphinx Up the Pyramid of Pain

Iterative threat-detection engagement against "Sphinx", a simulated pen tester. As I detected each malware sample, Sphinx upgraded their evasion technique, forcing me one level higher on the pyramid. All 6 samples detected and blocked — earned a flag at every level.

### 1️⃣ sample1.exe → Hash Values (Trivial)

Used the Malware Sandbox to scan the file and extract its SHA256 hash. Added the hash to the block list. Limitation: a single bit change in the file produces a new hash and bypasses the rule — confirmed by Sphinx's next move.

✅ Flag: `THM{f3cbf08151a11a6a331db9c6cf5f4fe4}`

### 2️⃣ sample2.exe → IP Addresses (Easy)

Sandbox scanned `sample2.exe`, revealing the C2 IP. Created an egress firewall rule:

```
Type: Egress · Source: Any · Destination: 154.35.10.113 · Action: Deny
```

Source set to `Any` because the attacker can rotate compromised hosts; only the C2 destination matters. Sphinx evaded next by rotating to new cloud IPs.

✅ Flag: `THM{2ff48a3421a938b388418be273f4806d}`

### 3️⃣ sample3.exe → Domain Names (Simple)

Sandbox revealed the C2 domain: `emudyn.bresonicz.info`. Created a DNS rule:

```
Category: Malware · Domain: emudyn.bresonicz.info · Action: Deny
```

Sphinx evaded by buying new domains and modifying DNS records.

✅ Flag: `THM{4eca9e2f61a19ecd5df34c788e7dce16}`

### 4️⃣ sample4.exe → Host Artefacts (Annoying)

Sphinx's tooling left a distinctive footprint on the host — registry modifications associated with the malware's persistence behaviour. Built a **Sigma Registry Modification rule** to detect the artefact pattern regardless of the binary that wrote it. This is the first level where detection survived a recompile/rotate by the attacker.

✅ Flag: `THM{46b21c4410e47dc5729ceadef0fc722e}`

### 5️⃣ sample5.exe → Network Tools (Challenging)

The hardest round. Sphinx's tooling beaconed via a small custom protocol. After analysing the outgoing connections log alongside the sandbox report, the signature wasn't the IP, port, or domain — it was the **fixed payload size (97 bytes) and fixed interval (1800 seconds)**. Built a Sigma **Network Connection rule**:

```
Remote IP: Any · Remote Port: Any · Size: 97 bytes · Frequency: 1800 seconds
ATT&CK: TA0011 (Command and Control)
```

Lesson: when stuck, look for what doesn't change rather than what does.

✅ Flag: `THM{c956f455fc076aea829799c0876ee399}`

### 6️⃣ sample6.exe → TTPs (Tough!)

Spotted the recurring exfiltration pattern: `cmd.exe` writing to `%temp%\exfiltr8.log`. The filename was an **operator habit**, repeated across samples — not a tool requirement. Built a Sigma **Process Creation rule**:

```
Process Name: cmd.exe
CommandLine: Contains
String: %temp%\exfiltr8.log
ATT&CK: TA0010 (Exfiltration)
```

Sphinx surrendered: *"I have no choice but to completely retrain myself and conduct extensive research to figure out how you're catching me."*

✅ Flag: `THM{c8951b2ad24bbcbac60c16cf2c83d92c}` (final flag)

## 🔐 Why It Matters

- The Pyramid of Pain is the foundational mental model for **detection engineering**. Climbing the pyramid is how a SOC moves from reactive (blocking known IOCs) to resilient (catching behaviours regardless of infrastructure)
- This lab walked through every level with real tooling — Sigma, firewall, DNS, sandbox — which is the actual workflow of a junior detection engineer
- Reaching the TTP level meant building a rule that forced the attacker to retrain — the geometric truth behind the pyramid's shape

## ❓ One Thing I Didn't Fully Understand

`sample5.exe` (network behaviour) was the hardest — took multiple attempts to land on the right combination of wildcards because the room's validation was unforgiving. The breakthrough was recognising that the signature wasn't the IP or the port, but the **fixed payload size (97 bytes) and interval (1800 seconds)**. Lesson learned: when stuck, look for what doesn't change rather than what does.

### Mission Debriefs Received

- **sample5 (Network signature authoring):** "Excellent investigative breakthrough! Transitioned from simple indicators to protocol-level artifact detection. Shows capability to craft low-FP network signatures — useful for a Detection Engineer validating detections during purple-team validation."
- **Platform navigation:** "Impressive systematic approach! Demonstrates platform navigation discipline; that's a strong career signal for a SOC Analyst working investigations during a purple-team engagement."

---

**Next up:** Eviction — APT28 threat hunting at E-Corp