# Phishing Analysis Tools — Portfolio Write-up

**Room:** Phishing Analysis Tools
**Path:** SOC Level 1 · Phishing Analysis
**Date completed:** 30 June 2026 (Day 22)
**Format:** Three real-world phishing case investigations using ANY.RUN, VirusTotal, and Talos Reputation Center

---

## 1. Overview

This room is the first **applied analyst work** in the Phishing Analysis module, transitioning from the theory-only foundation built in Phishing Analysis Fundamentals (Day 20) and Phishing Emails in Action (Day 21). It introduces the production toolkit a Tier 1 SOC analyst uses for phishing triage — sandboxes, reputation services, hash lookups, and header analysers — and applies them to three real phishing email samples.

All three cases are based on actual malicious emails captured in real environments. The note throughout the room — *"The domains, URLs, and IP addresses referenced in this task have hosted real malicious content and may still pose a risk"* — applies because the artefacts are not synthetic.

The room reflects the daily reality of MSSP-style SOC work: an end-user reports a suspicious email, the ticket lands in your queue, and your job is to extract IOCs, run them through tools, and produce a verdict.

---

## 2. The Phishing Analyst's Toolkit

Each tool category answers a specific question during phishing triage:

| Tool category | Question it answers | Tools used in this room |
|---|---|---|
| **Header analyser** | What does the email actually claim about itself? | Thunderbird "View Message Source" |
| **Reputation lookup** | Is this domain/IP known to be malicious? | Talos Reputation Center, VirusTotal |
| **Hash lookup** | Has this file been seen before, and what's its verdict? | VirusTotal |
| **Sandbox** | What does this file *do* when executed? | ANY.RUN |
| **Network analysis** | Where does this file communicate, and what does it download? | ANY.RUN (HTTP Requests, DNS Requests, Network Threats tabs) |

The key principle the room teaches: **no single tool gives the verdict.** A mature triage workflow correlates across at least two tools (reputation + behavioural, hash + URL, etc.) before escalating.

---

## 3. The Email Analysis Workflow — The Artefact List

Every phishing analysis produces an "artefact list" that gets logged in the ticket. This is the workflow the room trains you on:

| # | Artefact | Where it comes from |
|---|---|---|
| 1 | Sender email address | `From:` header (and compare against `Return-Path:` / `Reply-To:`) |
| 2 | Sender IP | First `Received:` line at the bottom of the header chain |
| 3 | Sending mail server hostname | `Received: from <hostname>` |
| 4 | Recipient email | `To:` header |
| 5 | Subject line | `Subject:` header |
| 6 | Date/time sent | `Date:` header |
| 7 | Authentication verdicts | `Authentication-Results:` (SPF/DKIM/DMARC) |
| 8 | URLs in body | All `<a href>` values + plain-text URLs |
| 9 | Attachments | Filenames, file types, SHA256 hashes |
| 10 | Embedded images / tracking pixels | `<img src>` values |

This list goes into the ticket so the next analyst (or detection engineer) has everything they need to write detections, block IOCs, or extend the investigation.

---

## 4. Pre-Case Warm-up: Talos Reputation Center

The room opened with a quick Talos Intelligence exercise to introduce the reputation-checking workflow.

**Domain queried:** `malware-test.com`

**Question:** What content category is listed for the domain?

**Answer:** ✅ **Computer Security**

**What the lookup showed:**
- **Web Reputation:** Neutral
- **Content Category:** Computer Security
- **Spamhaus PBL / SBL:** Not Listed
- **Talos Security Intelligence Block List:** Not added

**Why this matters in practice:** A reputation lookup is the cheapest, fastest first move in any phishing triage. Talos categorises domains into hundreds of content categories — "Computer Security", "Business and Industry", "Suspicious", "Malware Sites", etc. A category mismatch between what a domain *claims* to be (e.g. a "Banking" domain that's actually categorised as "Suspicious") is an immediate red flag. The lookup also surfaces blocklist membership, which lets you skip deeper investigation if a domain is already known-bad.

---

## 5. Case 1 — Phish3Case1.eml: Netflix Credential Harvesting

**Scenario:** A coworker reported a suspicious email impersonating Netflix. As Tier 1 analyst, my job was to extract IOCs and identify the malicious indicators.

**Email artefact summary:**

| Field | Value |
|---|---|
| Subject | `Y o u r N e t f l i x A c c o u n t i s o n H o l d` (spaced letters — classic filter-evasion) |
| Sent | 7 July 2021, 02:14 |
| Claimed sender | `Netflix` (display name with spacing) |
| Actual `From:` | `<JGQ47wazXe1xYVBrkeDg-JOg7ODDQwWdR@JOg7ODDQwWdR-yVkCaBkTNp.gogolecloud.com>` |
| Recipient | `redacted@yahoo.com` |
| Pretext | "Your account is on hold — Please Update Your Payment Details" |

### The Five Investigation Questions

| # | Question | Answer | How found |
|---|---|---|---|
| 1 | What reputable brand is this email tailored to impersonate? | **Netflix** | `From:` display name + body branding + subject |
| 2 | Who is the intended recipient based on the headers? | **redacted@yahoo.com** | `X-Apparently-To:` header + `To:` field |
| 3 | What is the `Received: from` IP address? | **10.197.37.234** | First `Received:` line at the bottom of the message source |
| 4 | What domain of interest is in the `Return-Path` field? | **etekno.xyz** | `Return-Path: <postmaster@etekno.xyz>` |
| 5 | What is the shortened URL behind the `UPDATE ACCOUNT NOW` button? | **https://t.co/yuxfZm8KPg?amp=1** | Hover-link visible in lower-left of Thunderbird window |

### Red Flags Identified

1. **`From:` display name spoofing** — "N e t f l i x" with weird spacing is a classic filter-evasion trick. Email gateways looking for the exact string "Netflix" miss this; users see it correctly because their brain pattern-matches.
2. **`From:` domain is `gogolecloud.com`, not `netflix.com`** — the actual sender domain has nothing to do with Netflix. This alone is a verdict-level finding.
3. **`From:` and `Return-Path:` mismatch** — `From:` shows `gogolecloud.com`, `Return-Path:` shows `etekno.xyz`. Two completely different domains.
4. **SPF failure** — `Received-SPF: none (domain of etekno.xyz does not designate permitted sender hosts)`. The `etekno.xyz` domain has no SPF policy, so any IP can claim to send on its behalf.
5. **DKIM unknown, DMARC unknown** — `dkim=unknown; spf=none smtp.mailfrom=etekno.xyz; dmarc=unknown header.from=JOg7ODDQwWdR-yVkCaBkTNp.gogolecloud.com`. All three authentication mechanisms failed or produced no verdict.
6. **URL shortener obfuscation** — `t.co` (Twitter's shortener) hides the actual destination of the call-to-action button. Legitimate Netflix communications would link directly to `netflix.com`, not through a shortener.
7. **Urgency pretext** — "Your account is on hold" + red "UPDATE ACCOUNT NOW" button. Classic Cialdini urgency + fear stacking.
8. **Spaced letters in subject and headings** — `Y o u r N e t f l i x A c c o u n t i s o n H o l d` and similar in the body. Designed to bypass keyword filters that scan for "Netflix" or "Update Your Payment Details" as literal strings.

### Verdict: ✅ Phishing — Credential harvesting via `t.co` shortened URL

The shortened URL almost certainly led to a fake Netflix login page designed to capture credentials. The attacker's goal was account takeover, likely for payment-method abuse or to harvest credentials for credential-stuffing attacks against other services.

---

## 6. Case 2 — Payment-updateid.pdf: Weaponised PDF via ANY.RUN

**Scenario:** A second Netflix-themed phishing email, but this one carried a malicious PDF attachment instead of a credential-harvesting URL. Investigation moved to **ANY.RUN sandbox** for dynamic analysis.

### The Six Investigation Questions

| # | Question | Answer | How found |
|---|---|---|---|
| 1 | How does ANY.RUN classify this suspected phishing email? | **Suspicious activity** | ANY.RUN sandbox banner |
| 2 | What is the name of the PDF attachment? | **Payment-updateid.pdf** | ANY.RUN file panel |
| 3 | What is the SHA256 hash of the PDF file? | **cc6f1a04b10bcb168aeec8d870b97bd7c20fc161e8310b5bce1af8ed420e2c24** | MD5 from ANY.RUN → VirusTotal lookup → Basic Properties |
| 4 | Which IP associated with `AcroRd32.exe` is flagged as malicious? | **2.16.107.24** | ANY.RUN text report → Network connections table |
| 5 | Which Windows process is classed as `Potentially Bad Traffic`? | **svchost.exe** | ANY.RUN → Network Threats tab (PID 1776) |

### The Hash Pivot Workflow

The Case 2 SHA256 question demonstrates a real production workflow:

1. ANY.RUN displayed the file's **MD5** (`4A2775EAE2EBEF41901A3F08D3B857C8`)
2. I copied the MD5 and pasted it into **VirusTotal**
3. VirusTotal returned the file's full Basic Properties — including the **SHA256** I needed for the answer
4. Same file, different hash representations — the analyst's job is to know which to use where

This is a daily-reality pattern: tools show different hashes depending on their default display, and SOC tickets standardise on SHA256 (because MD5 is cryptographically broken). Pivoting MD5 → SHA256 is a routine micro-task.

### Process Chain Analysis

The sandbox revealed the attack chain:

```
Payment-updateid.pdf (opened by user)
        │
        ▼
AcroRd32.exe (Adobe Reader process)
        │
        ├─→ Outbound connection to 2.16.107.24 (FLAGGED MALICIOUS)
        │   Multiple TCP connections, port 443
        │
        └─→ Additional connections to 93.184.221.240, 93.184.220.29
            (port 80)
                │
                ▼
        svchost.exe (PID 1776)
        Classified: Potentially Bad Traffic
        Suricata signature: ET INFO TLS Handshake Failure
```

### Why the `svchost.exe` Finding Matters Most

The most operationally important finding is that **svchost.exe** generated suspicious outbound traffic. svchost.exe is a *legitimate* Windows system process that hosts Windows services — attackers love abusing or impersonating it because:

1. It's everywhere on Windows, so anomalies hide in the noise.
2. Security analysts may dismiss svchost.exe alerts as false positives.
3. Real malware (TrickBot, Emotet, Cobalt Strike variants) routinely injects into svchost.exe to evade detection.

In production, a Sysmon Event ID 1 (Process Creation) showing svchost.exe with an unusual parent process, or making outbound connections to flagged IPs, is a **high-fidelity detection** that mature SOCs prioritise.

### Verdict: ✅ Malicious — Weaponised PDF delivering follow-on payload via process injection

The PDF didn't deliver an obvious EXE. It weaponised Adobe Reader (`AcroRd32.exe`) to make outbound connections to attacker-controlled infrastructure, and eventually injected behaviour into `svchost.exe` to mask continued network activity. A perimeter email gateway would have missed this entirely if it only scanned for known-bad hashes. **The malicious behaviour only surfaced in the sandbox dynamic analysis** — which is exactly why ANY.RUN, Joe Sandbox, Hybrid Analysis, and similar tools sit in every mature SOC's stack.

---

## 7. Case 3 — CBJ200620039539.xlsx: CVE-2017-11882 Equation Editor Exploit

**Scenario:** A weaponised Excel attachment masquerading as a business document, exploiting one of the most-abused Microsoft Office vulnerabilities of the past decade.

### The Six Investigation Questions

| # | Question | Answer | How found |
|---|---|---|---|
| 1 | How does ANY.RUN classify the `.xlsx` attachment? | **Malicious activity** | ANY.RUN red banner classification |
| 2 | What is the file name of the Excel attachment? | **CBJ200620039539.xlsx** | ANY.RUN file panel |
| 3 | What is the SHA256 hash value? | **5f94a66e0ce78d17afc2dd27fc17b44b3ffc13ac5f42d3ad6a5dcfb36715f3eb** | ANY.RUN file properties |
| 4 | What IP address did the malicious file contact? | **204.11.56.48** | ANY.RUN → Network → DNS Requests (resolved IP for `biz9holdings.com`) |
| 5 | What suspicious domain was contacted? | **findresults.site** | ANY.RUN → Network → DNS Requests |
| 6 | What CVE is exploited? | **CVE-2017-11882** | ANY.RUN classification tags |

### About CVE-2017-11882 — The Equation Editor Buffer Overflow

This is one of the most-weaponised Microsoft Office vulnerabilities in recent history. Worth knowing in detail because it appears in countless real-world attack campaigns:

- **Vulnerable component:** Microsoft Office Equation Editor (`EQNEDT32.EXE`) — a legacy component dating back to Office 2000.
- **Vulnerability type:** Buffer overflow in font name parsing.
- **Impact:** Arbitrary code execution simply by opening a maliciously-crafted Office document (`.docx`, `.xlsx`, `.rtf`). No macros, no user interaction beyond opening the file.
- **Discovery:** Disclosed by Embedi in November 2017. The vulnerability had existed for ~17 years before discovery.
- **Microsoft's response:** Initially patched in November 2017 (MS17-11882). Equation Editor was completely **deprecated and removed** from Office in January 2018 because of this and related vulnerabilities.
- **Threat actor adoption:** Used heavily by **Cobalt Group**, **APT34 (OilRig)**, **Patchwork**, and many commodity malware families (Lokibot, FormBook, AgentTesla, Hawkeye keyloggers). Still seen in real attacks years after patching because of slow Office update cycles in enterprise environments.

### Network Behaviour Observed

The sandbox showed the malicious Excel making two distinct DNS lookups:

1. **`biz9holdings.com`** → resolved to `204.11.56.48` (first call-out — initial C2 contact)
2. **`findresults.site`** → resolved to `103.224.182.251` (secondary domain — likely payload delivery or follow-on C2)

The two-stage call-out pattern (one domain for initial check-in, a second for payload retrieval) is a common attacker design — separating C2 from payload infrastructure provides operational redundancy if one is taken down.

### Why CVE-2017-11882 Was the Defining Detail

The CVE attribution is the most valuable finding in Case 3 because it tells the SOC:

1. **Patching status check** — any Office install on the network that's still vulnerable to CVE-2017-11882 is years behind on patching. This is an immediate audit finding.
2. **Threat actor profiling** — if the campaign matches Cobalt Group or APT34 TTPs, the SOC may be facing a targeted operation rather than commodity malware.
3. **Detection rule prioritisation** — Sigma rules exist for EQNEDT32.EXE spawning child processes. A SOC without that detection should write one.
4. **User awareness messaging** — the attack succeeds because users open Excel attachments from unknown senders. This shapes phishing-prevention training.

### Verdict: ✅ Malicious — Targeted exploitation of CVE-2017-11882 with two-stage C2

---

## 8. Cross-Case Patterns

Looking across all three cases, several patterns emerge that are worth holding in working memory for future triage:

| Pattern | Case 1 | Case 2 | Case 3 |
|---|---|---|---|
| Brand impersonation | Netflix | Netflix | Generic business document |
| Payload type | Credential harvesting URL | Weaponised PDF | Weaponised XLSX (CVE exploit) |
| Primary IOC type | URL (`t.co/yuxfZm8KPg`) | File hash + network IPs | File hash + CVE + network domains |
| Primary tool used | Header analysis (Thunderbird) | Sandbox (ANY.RUN) + VirusTotal | Sandbox (ANY.RUN) |
| Defensive control gap | SPF/DKIM/DMARC misconfiguration on `etekno.xyz` allowed spoofing | Static signature engines missed behavioural payload | Unpatched Office Equation Editor on victim hosts |

### Three Key Lessons

1. **Static analysis alone misses behavioural attacks.** Cases 2 and 3 both required sandbox dynamic analysis to surface the actual malicious behaviour. A SOC that relies only on hash-based blocklisting will catch the known and miss the novel.

2. **Cross-tool correlation is mandatory.** Case 2 needed MD5 (ANY.RUN) → SHA256 (VirusTotal) pivoting. Case 3 needed sandbox classification → CVE lookup. No single tool would have produced the full picture for either case.

3. **Headers are the cheapest, fastest first-pass.** Case 1 didn't need a sandbox — the `From:`/`Return-Path:` mismatch and SPF failure were sufficient for a verdict. Always read headers first; escalate to sandbox only when headers are inconclusive or the email carries an attachment.

---

## 9. Skills Demonstrated

| Skill | Evidence |
|---|---|
| Email header analysis | Read raw `.eml` message source to extract `Received:`, `Return-Path:`, `Authentication-Results:` in Case 1 |
| Authentication failure interpretation | Diagnosed SPF/DKIM/DMARC verdicts and explained `dmarc=unknown` plus `spf=none` |
| Sandbox triage (ANY.RUN) | Used ANY.RUN process trees, DNS Requests tab, Network Threats tab, and file properties across Cases 2 and 3 |
| Hash workflow | Pivoted MD5 → SHA256 via VirusTotal in Case 2; documented SHA256 hashes for all three cases |
| Reputation lookup | Used Talos Reputation Center for domain category check (warm-up exercise) |
| CVE attribution | Identified CVE-2017-11882 in Case 3 and contextualised its threat-actor relevance |
| Cross-case pattern recognition | Articulated common patterns and gaps across three different phishing scenarios |
| Production-style verdict authoring | Wrote definitive verdicts for each case with red flag enumeration |

---

## 10. Honest Reflection

This is the first room in the SOC L1 path where I genuinely felt like an analyst doing analyst work rather than a student answering questions about analyst work. A few honest takes:

- **ANY.RUN is excellent but expensive in production.** The community tier limits investigation depth and forces public submissions. Production SOCs typically pair it with internal sandboxes (Cuckoo, paid Joe Sandbox) for sensitive samples. Worth knowing the limitation when discussing tooling in interviews.

- **The hash-workflow micro-task is genuinely useful.** Pivoting between MD5, SHA1, and SHA256 across tools sounds trivial but trips up junior analysts in real triage. Documenting it explicitly in this write-up isn't padding — it's a real interview-grade detail.

- **CVE-2017-11882 is a great example because it's still relevant.** A 2017 vulnerability is still landing in 2026 phishing attacks because enterprise patching cycles are slow. The defensive lesson — "audit Office Equation Editor presence on all endpoints" — is operationally actionable.

- **One gap I'd want to close next:** I haven't yet written a SIEM detection rule against any of these IOCs. The next-step exercise would be: take the IOCs from these three cases, write Sigma rules to detect them in EDR/Sysmon telemetry, and validate the rules don't produce false positives. That's the bridge from analyst to detection engineer.

For now: the toolkit is mapped, the three cases are documented, and the workflow is internalised. On to the next room.

---

**Tags:** `#phishing-analysis` `#phishing-analysis-tools` `#any-run` `#virustotal` `#talos-intelligence` `#sandbox-analysis` `#cve-2017-11882` `#equation-editor` `#credential-harvesting` `#bec` `#netflix-phishing` `#soc-tier-1`