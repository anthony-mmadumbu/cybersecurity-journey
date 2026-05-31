# SOC L1 Alert Triage — Three Live Triages Write-up

**Lab Context:** TryHackMe — SOC Level 1 · SOC Team Internals
**Role Played:** Tier 1 SOC Analyst
**Focus:** Triaging three live SOC alerts of varying severities, assigning verdicts (True Positive / False Positive), adjusting severity where appropriate, and producing analyst documentation suitable for handover

---

## Executive Summary

This write-up documents three back-to-back live alert triages performed in the role of a Tier 1 SOC analyst. The three alerts spanned the full spectrum of outcomes — a confirmed malware delivery (True Positive, severity raised), a benign business activity flagged by an overly broad rule (False Positive, severity reduced), and a legitimate developer activity from a trusted user (False Positive). All three verdicts were correct on first attempt and the lab flag was earned.

The core analytical lesson reinforced across these three triages is that **the alert title and pre-assigned severity are starting hypotheses, not conclusions**. Every triage required reading the supporting evidence and reaching an independent verdict.

---

## Triage 1 — Double-Extension File Creation

### Alert Summary

| Field | Value |
| --- | --- |
| Alert Name | Double-Extension File Creation |
| Time | Mar 21st 2025 at 13:58 |
| Pre-Assigned Severity | Low |
| Host | LPT-HR-009 (HR department endpoint) |
| Process | chrome.exe |
| Process User | S.Conway |
| Target File | `C:\Users\S.Conway\Downloads\cats2025.mp4.exe` |
| Source URL (MotW) | `https://freecatvideoshd.monster/cats2025.mp4.exe` |
| File MD5 | `14d8486f3f63875ef93cfd240c5dc10b` |

### Indicators Observed

| Indicator | Assessment |
| --- | --- |
| Filename `cats2025.mp4.exe` | Classic double-extension — user sees `.mp4` (video), reality is `.exe` (executable) |
| Source domain `.monster` TLD | TLD frequently abused for malware distribution; not a reputable video host |
| Source `freecatvideoshd.monster` | Untrusted "free content" lure site — matches known social engineering patterns |
| Process: chrome.exe | Confirms browser-initiated download, ruling out lateral movement or system process |
| User: S.Conway (HR) | Non-technical user; typical phishing target |
| Mark-of-the-Web (MotW) preserved | Full provenance chain captured — supports forensic analysis |
| File MD5 captured | Can be enriched via threat intelligence (VirusTotal etc.) |

### Verdict — True Positive · Severity raised Low → High

### Analyst Comment

> *EDR alert on the creation of a Double-Extension File should be investigated, as Double-Extension Files are often used to make users run malicious payloads, thinking they're legitimate files or documents. Also, a download from GitHub occurred at 13:02, which likely introduced the malicious code to the endpoint.*

### Reasoning

The combination of double-extension filename, suspicious TLD, and a non-technical user being targeted by a "free content" lure is a textbook malware delivery pattern. The pre-assigned severity of "Low" did not reflect the realistic impact — execution of an unknown binary on an HR endpoint warrants High severity given the access HR systems typically have to sensitive personal data. The severity was raised accordingly. The MD5 hash and source URL provide the necessary indicators for L2 to enrich, contain, and hunt for further activity across the estate.

---

## Triage 2 — Potential Data Exfiltration

### Alert Summary

| Field | Value |
| --- | --- |
| Alert Name | Potential Data Exfiltration |
| Time | Mar 21st 2025 at 13:30 |
| Pre-Assigned Severity | Critical |
| Detection Rule | 5 GB or more sent from a single device to a single destination within a day |
| Destination | `*.zoom.us` |
| Source IP | 192.168.45.66 |
| Source Network | UK04/MEETINGROOM |
| Sent Data | 5.8 GB |
| Received Data | 5.2 GB |

### Indicators Observed

| Indicator | Assessment |
| --- | --- |
| Destination `*.zoom.us` | Legitimate enterprise SaaS for video conferencing — not a known exfiltration destination |
| Source network: `UK04/MEETINGROOM` | Source is a meeting room device, not a user endpoint — strongly suggests a meeting in progress |
| Symmetric traffic pattern (5.8 GB out / 5.2 GB in) | Real exfiltration is typically asymmetric (heavy outbound, minimal inbound). Symmetric traffic is consistent with a video call |
| Trigger threshold (5 GB/day) | The detection rule fires on volume alone, without considering destination reputation or source context |

### Verdict — False Positive · Severity reduced Critical → Low

### Analyst Comment

> *Based on the detection rule, the EDR thinks this is a potential Data Exfiltration because, during a Zoom meeting and possibly file sharing and video conferencing, the data being sent is likely to exceed the 5 Gigabyte limit set by the detection rule, which then triggered a false potential exfiltration alert.*

### Reasoning

This alert is the kind of false positive that a hurried analyst can easily escalate due to the alarming alert title and Critical severity tag. However, three independent indicators all point to legitimate business use: the destination is a known enterprise SaaS, the source is a meeting room, and the traffic shape (symmetric, sustained, bidirectional) matches video conferencing — not data theft.

The deeper takeaway is that the **detection rule itself is poorly tuned** — it triggers on volume to a single destination without considering destination category or source context. In a real SOC, this triage would generate a recommendation to the detection engineering team: exclude traffic to known sanctioned SaaS (Zoom, Microsoft Teams, Google Workspace) from the rule, or add a separate baseline for meeting room subnets.

---

## Triage 3 — Download from GitHub Repository

### Alert Summary

| Field | Value |
| --- | --- |
| Alert Name | Download from GitHub Repository |
| Time | Mar 21st 2025 at 13:02 |
| Pre-Assigned Severity | Low |
| Accessed URL | `https://github.com/facebook/react` |
| Source User | G.Chandler |
| Source Host | LPT-IT-063 (IT department endpoint) |
| Source Network | VPN/DEVELOPERS |

### Indicators Observed

| Indicator | Assessment |
| --- | --- |
| URL: `github.com/facebook/react` | The official Facebook/Meta React repository — one of the most widely used open-source projects in the world |
| Source user: G.Chandler (IT) | Developer in the IT team — has a legitimate need to access open-source libraries |
| Source network: `VPN/DEVELOPERS` | Connected via the corporate developer VPN, an explicitly sanctioned access path |
| Detection rule scope | The rule flags *any* GitHub download — broad by design, requires analyst interpretation |

### Verdict — False Positive · Severity remains Low

### Analyst Comment

> *Nothing to worry about here. This was from the IT team endpoint and is a safe download from a Facebook repository on GitHub, not a malicious one.*

### Reasoning

This is a deliberately broad detection rule — GitHub does host both legitimate open-source code and malicious tooling, so the rule alerts on all downloads and relies on the analyst to apply judgment. In this case, every contextual indicator (well-known legitimate repo, developer on the developer VPN, IT department user) confirms benign business use.

**Important contextual link:** The first triage referenced "a download from GitHub at 13:02" as a possible introduction vector for the malicious double-extension file. That timing matches *this* alert exactly — and the temptation, having flagged that earlier observation, is to treat this download as suspicious by association. However, careful evidence review shows they are unrelated:

- The double-extension file was downloaded by **Chrome** from **`freecatvideoshd.monster`**, not from GitHub
- This GitHub download was by a **different user** (`G.Chandler` in IT, not `S.Conway` in HR)
- This download was from a **different host** (`LPT-IT-063` vs `LPT-HR-009`)

Two unrelated events happened to occur within the same time window. This triage demonstrates the importance of **letting evidence — not coincidence — drive the verdict**.

---

## Cross-Triage Lessons Carried Forward

1. **The alert title and pre-assigned severity are hypotheses, not conclusions.** Each of the three alerts had its severity adjusted by the analyst (raised once, dropped once, unchanged once) — proving that initial classifications are starting points only.
2. **Context beats pattern matching.** Triage 1 and Triage 3 both involved downloads, but only one was malicious. The difference was every contextual indicator — user, source, destination reputation.
3. **Traffic shape tells a story.** Symmetric vs asymmetric, sustained vs burst — these patterns distinguish video calls from exfiltration without ever needing to inspect content.
4. **A False Positive is also a finding.** Triage 2 highlighted a poorly tuned detection rule, which is a valuable input to the detection engineering function — not just noise.
5. **Coincidence isn't causation.** Two events in the same time window aren't related unless the evidence connects them.

---

## Documentation Standard Demonstrated

Every analyst comment recorded in this exercise included:

- The technique or rule that triggered the alert
- The specific evidence relied upon for the verdict
- The link between the evidence and the conclusion

This is the documentation standard that allows L2 / IR teams to pick up an investigation without re-reading every log, and that supports compliance and audit reviews after the fact.

---

## Tools & Frameworks Referenced

- SIEM alert triage workflow (Status, Severity, Verdict, Assignee, Analyst Comment fields)
- True Positive / False Positive classification
- Mark-of-the-Web (MotW) provenance tracking
- Detection rule tuning principles
- Indicator of Compromise (IOC) extraction (URL, MD5)

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*