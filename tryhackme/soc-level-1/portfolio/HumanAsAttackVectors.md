# Humans as Attack Vectors — SOC Analyst Triage Write-up

**Lab Context:** TryHackMe — SOC Level 1 · Blue Team Introduction
**Role Played:** Tier 1 SOC Analyst
**Focus:** Triaging four real-world social engineering and phishing scenarios, then proposing a defensive security policy

## Executive Summary

This write-up documents my work through four progressive SOC analyst triage scenarios targeting the human attack surface, covering Trojanised freeware, business email compromise (BEC), executive impersonation (vishing), and credential phishing. Each scenario required reading beyond the alert title to interpret context, weigh indicators, and produce a defensible verdict. The exercise concluded with the design of a four-policy corporate security framework directly informed by the threats observed.

All four verdicts were correct on first attempt; the policy framework was validated through colleague feedback and TryHackMe system confirm all verdicts and policies are correct.

## Scenario 1 — Suspicious Software Download

**Alert Context**

Lucas Martinez, a new software engineer, urgently needed 7-Zip but the installer wouldn't launch. Initial investigation revealed Setup.exe downloaded from best-freeapps-2025.top, blocked by antivirus six times.

**Indicators Observed**

| Indicator | Assessment |
| --- | --- |
| Source domain: best-freeapps-2025.top | Not the official 7-Zip distribution channel (7-zip.org) |
| TLD: .top | Frequently abused for malware distribution and freeware-bait sites |
| AV blocked the file 6 times | Strong signal, repeated blocks indicate a deterministic detection, not a one-off false positive |
| User context | New employee, urgency, willing to bypass security controls |

**Verdict**

**Quarantine the Setup.exe and instruct Lucas to use the official 7-Zip installer from 7-zip.org.**

**Reasoning**

The AV's repeated blocking of the same file is the most decisive indicator, modern endpoint detection rarely flags the same legitimate file six times without cause. Combined with the unofficial source, this is consistent with malware bundled into a cracked or trojanised version of a legitimate utility, a well-documented delivery pattern.

The correct user-education response is to redirect Lucas to the official source rather than create an AV exclusion, which would have introduced persistent risk to the endpoint.

## Scenario 2 — Business Email Compromise with Whaling characteristics

**Alert Context**

SIEM flagged a "Stripe Invoice" email to Mark Phillips (Finance Director) confirming a $23,650 payment. Sender: noreply@stripe-payments.xyz. Recipients were directed to contact support@stripe.com for queries. The email included a password-protected Invoice.rar attachment with the password (1111) supplied in plain text within the email body.

**Indicators Observed**

| Indicator | Assessment |
| --- | --- |
| Domain mismatch | Sender stripe-payments.xyz does not match the contact address stripe.com |
| Suspicious TLD | .xyz is not used by Stripe; legitimate communications come from stripe.com |
| Password-protected archive | Common technique to evade automated email scanners that cannot inspect encrypted contents |
| Password in plain text within email body | The "encryption" is purely for evasion, not confidentiality — a hallmark of malicious payload delivery |
| Social engineering trigger | Large dollar amount targeted at a Finance Director to manufacture urgency |
| Generic phrasing and unprofessional structure | Unusual for a regulated payment processor |

**Verdict**

**Block the email at the gateway and initiate malware analysis on the attachment in an isolated environment.**

**Reasoning**

This email exhibits multiple high-confidence phishing indicators that align with documented BEC tradecraft. The password-protected archive is particularly telling, legitimate businesses do not deliver invoices this way, and the inclusion of the password in plain text confirms the intent is to bypass automated scanning rather than protect data. Quarantining without analysis would lose valuable threat intelligence; blocking with analysis preserves the indicators for detection engineering.

## Scenario 3 — Vishing & Executive Impersonation

**Alert Context**

IT Support (Isabella) received a phone call at 9 PM from a hidden number claiming to be CEO Ben, requesting a Gmail password reset. The reset was approved. Login logs subsequently showed activity from the USA (Ben's country of residence). Ben did not respond to follow-up calls or messages.

**Indicators Observed**

| Indicator | Assessment |
| --- | --- |
| Hidden caller ID | Inconsistent with normal executive communication |
| Out-of-hours timing | 9 PM contact for a password reset is anomalous |
| No response to verification attempts | Eliminates ability to confirm legitimacy through a second channel |
| Successful login from expected geography | **Distractor** — attackers commonly route through VPNs or compromised proxies in the victim's country to defeat geolocation-based detection |
| Pressure on a junior IT staff member | Classic social engineering target selection |

**Verdict**

**Disable Ben's Gmail account immediately. Maintain the lockout until Ben confirms his identity in person or via a verified out-of-band channel.**

**Reasoning**

Account compromise via help desk social engineering sometimes called "vishing" is one of the most cost-effective attack methods against enterprises, repeatedly demonstrated in real-world incidents (e.g., MGM Resorts, 2023). The matching geographic login is the most dangerous indicator here because it tempts the analyst to dismiss the alert, however, geolocation alone is not a reliable trust signal in 2026, given the widespread use of consumer VPN services and residential proxy networks.

The cost of disabling a real CEO's account for a few hours is minimal compared to the cost of an executive email compromise.

## Scenario 4 — Credential Phishing via Typosquatting

**Alert Context**

SIEM raised "Anomalous Login Location" for Rose Lewis (HR Assistant). Login from London, UK; typical from Oxford, UK. URLs visited immediately prior to login included:

- http://login[.]micrsoft365-online[.]ru
- https://hroyhiqtspqgkp[.]info
- https://mail[.]tryhackme[.]thm

**Indicators Observed**

| Indicator | Assessment |
| --- | --- |
| Geographic anomaly | London vs Oxford — minor distance; not a strong indicator on its own |
| Typosquatted domain | micrsoft365-online[.]ru — missing "o" in "microsoft," hosted on a Russian TLD, classic phishing infrastructure |
| Random/obfuscated domain | hroyhiqtspqgkp[.]info — algorithmically generated, consistent with redirector or payload delivery |
| Sequence of activity | Suspicious URL visits **immediately preceded** the successful login, this establishes the kill chain |

**Verdict**

**Disable Rose Lewis' account, initiate password reset through verified channels, and review recent activity for signs of post-compromise behaviour (mailbox rules, MFA changes, data access).**

**Reasoning**

The SIEM titled this alert "Anomalous Login Location," which is misleading. The location anomaly is minor and easily explained by routine travel, but the visited URLs before login tell the real story. Rose almost certainly entered her credentials into the typosquatted Microsoft login page, and the attacker is now authenticating with stolen credentials from a London-based session.

The alert title in this scenario is more a hypothesis than an actual conclusion. Context-rich data in the alert body, in this case browser history, will often contains the actual story.

## Final Task — Corporate Security Policy Design

**Brief**

As acting CISO of TrySecureMe, select four corporate security policies that would meaningfully reduce the organisation's exposure to the threats observed.

**Policies Selected**

| Priority | Policy | Maps To Threat |
| --- | --- | --- |
| 1 | **Access Management Policy** — IT support must verify all sensitive requests (password resets, access approvals) through a defined out-of-band procedure | Scenario 3 (CEO impersonation) |
| 2 | **Anti-Phishing Solution** — Deploy an email security gateway capable of inspecting password-protected archives and analysing sender reputation | Scenario 2 (Stripe phishing email) |
| 3 | **Security Awareness Program** — Quarterly training covering modern phishing, typosquatting, and reporting workflows | Scenario 4 (Rose's credential phishing) |
| 4 | **Antivirus Solution** — Maintain a reliable endpoint AV deployment across all workstations | Scenario 1 (Lucas's malware download) |

**Design Rationale**

The policy mix deliberately covers the four layers a defence-in-depth model recommends:

- **Process control** (Access Management) — defends against social engineering
- **Technical prevention** (Anti-Phishing) — catches threats at the gateway
- **Human layer** (Awareness Training) — addresses what no tool can fully prevent
- **Last-line defence** (Antivirus) — assumes earlier layers may fail

Feedback received from colleagues validated all four selections, with the antivirus and anti-phishing investments specifically called out as strong responses to email-borne and downloaded malware respectively.

## Key Lessons Carried Forward

- **The alert title is a hypothesis, not a conclusion.** Always read the body and the surrounding context before deciding.
- **AV blocks are evidence, not noise.** Repeated detections on the same file are strong signals.
- **Geolocation is not a trust signal in 2026.** VPNs and residential proxies defeat it routinely.
- **Verification through a second channel is non-negotiable for sensitive requests.** This single control would have stopped Scenario 3.
- **Defence is layered.** Each of the four policies addresses a different stage of a real attack chain.

## Tools & Frameworks Referenced

- SIEM alerting and triage workflow
- Endpoint Detection & Response (EDR) behaviour
- Email security gateway concepts
- NIST CSF "Identify → Protect → Detect → Respond" model

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*