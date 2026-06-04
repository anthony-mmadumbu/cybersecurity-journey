# SOC L1 Alert Reporting — Escalation Reports Write-up

**Lab Context:** TryHackMe — SOC Level 1 · SOC Team Internals
**Role Played:** Tier 1 SOC Analyst (with escalation to Tier 2)
**Focus:** Writing structured analyst reports for two high-impact alerts, raising severity where evidence justified it, and escalating to L2 with sufficient detail for handover

---

## Executive Summary

This write-up documents two live alert investigations performed end-to-end: from initial triage, through evidence interpretation, to a formal analyst report and escalation to Tier 2. Both alerts arrived at Medium severity and were correctly raised to Critical after investigation; both were verdicted True Positive; both were escalated to L2 with handover-ready reports. The SOC platform validated both reports at **100% Authenticity**, meaning the reports were assessed as genuine analyst work rather than templated or auto-generated text.

The two alerts span complementary attack surfaces — one targets a human (phishing), one targets a system (server-side compromise) — together exemplifying the breadth of an L1 SOC analyst's daily reality.

---

## Report Structure Used

For both alerts, the analyst comment followed a deliberate **5W structure** that ensures L2 can take over the investigation without re-reading the raw evidence:

1. **WHEN** — exact timestamp in UTC
2. **WHO** — affected user / system / privileged context
3. **WHAT** — observed event and supporting evidence
4. **WHY suspicious** — the indicators that drive the verdict
5. **WHY escalating** — what L1 cannot resolve and what L2 should do next

This is the structure the platform's Authenticity validator appears to reward, and — more importantly — it is the structure a real SOC manager would expect to read.

---

## Report 1 — Email Marked as Phishing after Delivery

### Alert Summary

| Field | Value |
| --- | --- |
| Alert Name | Email Marked as Phishing after Delivery |
| Time | Mar 27th 2025 at 19:25 UTC |
| Pre-Assigned Severity | Medium |
| Recipient | Eddie Huffman, IT Manager (`e.huffman@tryhackme.thm`) |
| Sender (claimed) | `Microsoft Support <support@microsoft.com>` |
| Subject | Important Update: Microsoft Teams Pricing Increase |
| Body Keywords | "600% price increase"; "urgent notice"; "download the report"; "read the details" |
| Security Checks | SPF/Fail; DKIM/Fail |
| Attached URLs | None |
| Attached Files | `REPORT.rar` |

### Indicators Observed

| Indicator | Assessment |
| --- | --- |
| SPF/Fail | The sending server is not authorised to send mail on behalf of `microsoft.com` — strong spoofing indicator |
| DKIM/Fail | The message signature does not validate against `microsoft.com` — confirms the sender identity has been forged |
| Sender mismatch with authentication | Email claims to be from Microsoft Support, yet fails both Microsoft's published authentication standards |
| Body language ("urgent", "600% increase") | Classic phishing pressure techniques designed to short-circuit critical thinking |
| `.rar` attachment | Compressed archive — a known evasion technique to bypass automated email scanning |
| Recipient: IT Manager | Privileged-access target. Compromise here cascades to systems and credentials |

### Verdict — True Positive · Severity raised Medium → Critical · Escalated to L2 (E.Fleming)

### Analyst Report (verbatim)

> *At 19:25 UTC, Eddie Huffman, IT Manager, received an email from support@microsoft.com about Microsoft Teams pricing increases, with a compressed attachment with a .rar extension. This email contains common phishing keywords and failed SPF and DKIM security checks, even though the source email address appears to be from a Microsoft domain. The attached file, REPORT.rar, needs to be analysed, hence the escalation to L2.*

**Authenticity:** 100% · **Length:** 421 characters (well above the 50-character minimum)

### Reasoning

The combination of SPF + DKIM failures together with a sender address claiming to be Microsoft is conclusive evidence of spoofing — legitimate Microsoft communications pass both standards by design. The escalation is justified not because the verdict was uncertain (it wasn't), but because the **attached file requires deeper analysis** (static and dynamic) that falls outside L1 scope. L2 is the right tier to detonate the archive in a sandbox, extract IOCs, and hunt for similar emails across the estate.

Severity was raised from Medium to Critical for three independent reasons:

- The target is a **privileged user** (IT Manager) — compromise has cascading impact
- The message was **post-delivery** (already in the inbox, so the user could have already engaged)
- The payload (`REPORT.rar`) is **active** — an executable archive, not just suspicious text

---

## Report 2 — Spike of Domain Discovery Commands

### Alert Summary

| Field | Value |
| --- | --- |
| Alert Name | Spike of Domain Discovery Commands |
| Time | Mar 27th 2025 at 19:56 UTC |
| Pre-Assigned Severity | Medium |
| Host | DMZ-MSEXCHANGE-2013 |
| Host OS | Windows Server 2012 R2 |
| User | NT AUTHORITY\SYSTEM |
| Source Process | `C:\Windows\System32\cmd.exe` |
| Parent Process | `C:\Users\Public\revshell.exe` |
| Grandparent Process | `C:\Windows\System32\inetsrv\w3wp.exe` |
| Invoked Commands | `dir`, `hostname`, `whoami /priv`, `net group "Domain Admins" /domain`, `nltest /dclist:tryhackme.thm` |

### Indicators Observed

| Indicator | Assessment |
| --- | --- |
| Host: DMZ-MSEXCHANGE-2013 | Exchange server in a DMZ — high-value target with known historical exploits (ProxyLogon, ProxyShell). Internet-exposed by design |
| User: `NT AUTHORITY\SYSTEM` | Highest privilege account in Windows. Commands running under SYSTEM are running at maximum impact |
| Grandparent: `w3wp.exe` | IIS worker process — the web-facing component. It should serve HTTP, not spawn arbitrary binaries |
| Parent: `revshell.exe` in `C:\Users\Public\` | Binary name self-describes intent. Located in a world-writable directory commonly used by attackers for staging |
| Process tree: `w3wp.exe → revshell.exe → cmd.exe` | Classic web shell / reverse shell pattern — IIS exploited, payload dropped, interactive shell obtained |
| Commands: AD reconnaissance | `net group "Domain Admins"` and `nltest /dclist` identify high-value accounts and domain controllers — the **pre-lateral-movement phase** of an intrusion |

### Verdict — True Positive · Severity raised Medium → Critical · Escalated to L2 (E.Fleming)

### Analyst Report (verbatim)

> *An alert was triggered at 19:56 UTC by the user "NT AUTHORITY\SYSTEM" for running some command-line domain discovery commands. I am not sure why 3 different processes are starting from a Grandparents process that launched a w3wp.exe process, then a revshell.exe parent process and a cmd.exe process. Preliminary research I conducted indicated that this is a Spawning Child Process and is a major red flag. I am not completely sure what that means, hence the escalation to L2 for help and deeper analysis, as this is likely a malicious actor attempting to gain admin privileges.*

**Authenticity:** 100% · **Length:** 577 characters

### Reasoning

The process tree alone tells a textbook story of an Exchange / IIS compromise:

```
w3wp.exe          (IIS worker — internet-exposed web service)
  └── revshell.exe   (dropped payload in C:\Users\Public)
        └── cmd.exe      (interactive shell)
              ↓
              whoami /priv
              net group "Domain Admins" /domain
              nltest /dclist:tryhackme.thm
```

This sequence is consistent with **post-exploitation reconnaissance** — the attacker has a foothold and is now mapping the domain to identify privileged accounts and domain controllers for the next phase (lateral movement and privilege escalation).

The escalation to L2 was correct for two reasons:

1. **Containment urgency** — a running compromise on an internet-exposed Exchange server requires immediate isolation, which is an L2 / IR action
2. **Knowledge gap** — I correctly identified the process tree as a major red flag from preliminary research, but had not yet studied Windows process spawning in depth. Escalating with specific questions for L2 is the right call when out of depth, rather than guessing

The honest acknowledgement of the knowledge gap was a deliberate choice. In a real SOC, an L1 analyst who says *"I know this is bad and here's why I think so — but I need L2 to confirm"* is more valuable than one who pretends to certainty they don't have. The escalation handed L2 a clear hypothesis to validate, not a vague worry to investigate from scratch.

---

## Cross-Report Lessons Carried Forward

1. **Severity adjustments must be defensible.** Both alerts arrived at Medium and were raised to Critical — each on independent grounds (privileged target, active payload, post-delivery exposure; running compromise, SYSTEM-level access, AD reconnaissance). The reasoning was documented in the report, not just applied silently.
2. **Escalation is a deliberate decision, not a default.** Triage 1 (phishing) was escalated because the attachment requires deeper analysis. Triage 2 (process tree) was escalated because containment is urgent and L1 knowledge was insufficient. Different reasons, same verdict.
3. **Honesty beats false certainty.** Acknowledging "I am not completely sure what that means" in Report 2 made the escalation *more* effective — L2 received specific questions to answer rather than a vague hand-wave.
4. **Process tree is its own evidence.** The relationship between processes (`w3wp.exe → revshell.exe → cmd.exe`) is often more revealing than any individual command. Learning to read process trees is essential for Windows-focused SOC work.
5. **Authentication failures are conclusive.** SPF and DKIM are designed to be unforgeable. When both fail on a domain like `microsoft.com`, the verdict is settled — only the impact and response need to be determined.

---

## Documentation Standard Demonstrated

Every report in this exercise included:

- A specific UTC timestamp anchoring the event in time
- Named users and hosts (allowing the report to be searched and correlated later)
- The evidence relied upon for the verdict (not just the verdict itself)
- The reason for escalation (what L1 cannot resolve, what L2 should do next)
- Honest acknowledgement of knowledge limits where relevant

Both reports scored 100% Authenticity from the SOC platform — confirmation that the writing was assessed as genuine analyst work.

---

## Tools & Frameworks Referenced

- SOC alert lifecycle (Status, Severity, Verdict, Assignee, Analyst Comment)
- SPF (Sender Policy Framework) and DKIM (DomainKeys Identified Mail) email authentication
- Windows process tree analysis
- IIS / Exchange compromise patterns (ProxyLogon / ProxyShell context)
- MITRE ATT&CK Discovery tactic (T1087 — Account Discovery; T1018 — Remote System Discovery)
- L1 → L2 escalation workflow

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*