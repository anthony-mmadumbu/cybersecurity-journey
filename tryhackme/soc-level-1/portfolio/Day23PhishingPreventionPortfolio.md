# 🛡️ Phishing Prevention — Email Authentication & SMTP Traffic Analysis

**TryHackMe · SOC Level 1 · Phishing Analysis Module**
**Date:** 19 July 2026 (Day 23)
**Room:** [Phishing Prevention](https://tryhackme.com/room/phishingprevention)

---

## Overview

Phishing remains one of the most common and effective ways for attackers to gain initial access — MITRE ATT&CK tracks the reconnaissance side of this as [Phishing for Information (T1598)](https://attack.mitre.org/techniques/T1598). This room covered the defensive counterpart: the controls organisations deploy to prevent, detect and mitigate phishing.

Three layers were covered:

1. **Email authentication protocols** — SPF, DKIM, DMARC and S/MIME
2. **SMTP traffic analysis** — reading server responses and message internals from a packet capture
3. **Anti-phishing protection measures** — gateway controls, sandboxing and user-focused defences

> **A note on preparation:** Task 6 recommended familiarity with Wireshark traffic analysis. Rather than fumble through it, I paused this room and completed 11 tasks of TryHackMe's Wireshark room first, then returned to the lab. That detour is written up separately (Day 24) — the analysis below is where it paid off.

---

## 1. The Email Authentication Stack (Tasks 2–5)

### SPF — Sender Policy Framework

SPF lets a domain publish (via a DNS TXT record) which mail servers are authorised to send email on its behalf. The receiving server checks the connecting IP against that record.

**Key results an analyst will see in headers:**

| Result | Meaning | Intended action |
|---|---|---|
| `Pass` | Sending IP is authorised | Deliver normally |
| `SoftFail` (`~all`) | IP not authorised, but policy is lenient | **Flag** as suspicious (accept but mark, e.g. to spam/junk) |
| `Fail` (`-all`) | IP not authorised, strict policy | Reject |

**Lab Q&A:**
- Based on TryHackMe's SPF record, domains authorised to send on its behalf: **3**
- Intended action for a `SoftFail` result: **Flag**

### DKIM — DomainKeys Identified Mail

DKIM adds a cryptographic signature to outgoing mail, signed with the domain's private key. Receivers fetch the public key from DNS (`selector._domainkey.domain`) and verify the signature — proving the message wasn't altered in transit and genuinely came from the claimed domain.

**Failure states matter in triage.** A `permerror` means the verification could not be performed at all — commonly because the public key can't be retrieved.

**Lab Q&A:**
- Reason for the `permerror` in the sample header: **no key for signature**

### DMARC — Domain-based Message Authentication, Reporting & Conformance

DMARC sits on top of SPF and DKIM. It checks *alignment* (does the `From:` domain match what SPF/DKIM validated?) and tells receivers what to do on failure via the published policy:

| Policy | Effect |
|---|---|
| `p=none` | Monitor only — deliver, but send reports |
| `p=quarantine` | Send failures to spam/junk |
| `p=reject` | **Block outright — strongest protection** |

**Lab Q&A:**
- DMARC policy providing the greatest protection: **`p=reject`**

### S/MIME — Secure/Multipurpose Internet Mail Extensions

S/MIME uses public-key cryptography and certificates to provide two guarantees:

- **Digital signature** → authenticity and integrity (proves who sent it, unaltered)
- **Encryption** → confidentiality (only the intended recipient can read it)

**Lab Q&A:**
- S/MIME component ensuring only the intended recipient can read the message: **Encryption**

---

## 2. Lab — Analysing SMTP Responses (Task 6)

**Evidence:** `traffic.pcap`, opened in Wireshark on the lab machine.

SMTP is a plaintext protocol, so server responses are directly readable in a capture. Response codes follow a clear scheme: `2xx` success (e.g. `220 Service ready`, `250 OK`), `3xx` intermediate (`354 Start mail input`), and `5xx` permanent failures — which is where blocked and rejected mail shows up.

### Findings

| # | Question | Answer |
|---|---|---|
| 1 | Wireshark filter to narrow results by SMTP response code | `smtp.response.code` |
| 2 | Packets containing `220 Service ready` | **19** |
| 3 | Response code for the email blocked by `spamhaus.org` | **553** |
| 4 | Full `Response code:` message for that packet | **Requested action not taken: mailbox name not allowed (553)** |
| 5 | Messages blocked under code `552` for potential security issues | **6** |

### Method

- Applied `smtp.response.code == 220` and read the packet count from Wireshark's status bar to answer Q2.
- For the Spamhaus block, filtered SMTP responses and inspected the response text — the rejection line referenced `spamhaus.org`, a DNS-based blocklist (DNSBL) that mail servers query in real time to refuse mail from known spam sources. The server returned **553**.
- `smtp.response.code == 552` isolated the six messages rejected for "exceeded storage allocation / security issues" — code 552 is commonly repurposed by mail gateways to refuse messages carrying suspicious content.

**Analyst takeaway:** 5xx responses in traffic are a free detection signal. A burst of 553/552 rejections against your mail gateway can indicate a spam/phishing campaign being attempted — and the response text often names the blocklist or reason outright.

---

## 3. Lab — Inspecting Emails and Attachments (Task 7)

Same capture, moving from status codes to message content using the **Internet Message Format (IMF)** dissector — Wireshark's `imf` filter exposes parsed email internals: sender/recipient fields, content type, and attachments.

### Findings

| # | Question | Answer |
|---|---|---|
| 1 | SMTP packets available for analysis | **512** |
| 2 | Attachment name in packet 270 | **`document.zip`** |
| 3 | Non-responding host IP making the message undeliverable (packet 270) | **212.253.25.152** |
| 4 | Email client that sent the message containing `attachment.scr` (via `imf` filter) | **Microsoft Outlook Express 6.00.2600.0000** |
| 5 | Encoding used for the malicious attachment | **base64** |

### Method

- The `smtp` display filter gave the packet count (512).
- Packet 270 was a bounce/non-delivery report: following the message content showed the failed attachment (`document.zip`) and the unreachable host IP quoted in the error text.
- Filtering on `imf` surfaced full parsed messages. The one carrying `attachment.scr` included an `X-Mailer` header identifying the client as Outlook Express 6.00.2600.0000, and the MIME part declared `Content-Transfer-Encoding: base64`.

**Analyst takeaway:** `.scr` (Windows screensaver) files are executables in disguise — a classic phishing payload. The `X-Mailer` header and encoding are both pivotable IOCs: an ancient client version like Outlook Express 6 on modern traffic is itself anomalous, and base64-encoded executable attachments are exactly what gateway rules and sandboxes should be catching.

---

## 4. Anti-Phishing Protection Measures (Task 8)

Beyond authentication protocols, layered defences include secure email gateways, spam filtering, URL rewriting/scanning, attachment detonation, user awareness training and report-phishing buttons.

**Scenario Q&A:**
- A security team needs to detect hidden malware in email attachments by observing file behaviour safely, without risking real systems: **Sandboxing** — detonating the file in an isolated environment (as done with ANY.RUN in the Phishing Analysis Tools lab, Day 22).

---

## Skills Demonstrated

- Interpreting SPF/DKIM/DMARC verification results and failure states from email headers
- Understanding DMARC policy enforcement levels and S/MIME's signature vs encryption roles
- Wireshark SMTP analysis: display filters (`smtp`, `smtp.response.code`, `imf`), reading response codes, and extracting message internals and attachment metadata from a PCAP
- Recognising DNSBL (Spamhaus) rejections and repurposed 5xx codes as detection signals
- Identifying malicious attachment indicators: double-purpose extensions (`.scr`), base64 transfer encoding, anomalous `X-Mailer` values
- Learning agility: identified a skills gap mid-room (traffic analysis), addressed it with a dedicated Wireshark room, and returned to complete the lab confidently

## Reflection

The deliberate pause to learn Wireshark properly was the right call — the lab questions that would have been guesswork (packet counts, filters, following message content) became routine. The biggest conceptual gain was seeing the authentication stack and the traffic view as two sides of the same investigation: headers tell you *what the receiving server decided*, and the PCAP tells you *the conversation that led there*. Next I want to trace DMARC alignment end-to-end on real headers where SPF and DKIM disagree.

**Tags:** `#SOC` `#PhishingPrevention` `#SPF` `#DKIM` `#DMARC` `#SMIME` `#SMTP` `#Wireshark` `#TrafficAnalysis` `#BlueTeam` `#TryHackMe`