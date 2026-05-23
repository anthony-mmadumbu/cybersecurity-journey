# Systems as Attack Vectors — SOC Analyst Hardening Write-up

**Lab Context:** TryHackMe — SOC Level 1 · Blue Team Introduction
**Role Played:** SOC Analyst / Acting System Hardening Lead
**Focus:** Understanding why systems are high-value targets, how they are attacked, and selecting a defensible remediation plan from a list containing deliberate anti-patterns

---

## Executive Summary

This write-up documents my work understanding the system attack surface and producing a corporate hardening plan. The exercise required distinguishing between four legitimate security controls and three plausible-sounding distractors that violate core security principles. All four correct remediation actions were selected, and the plan was validated through colleague feedback.

The key analytical challenge was not identifying good controls in isolation, but recognising bad advice dressed up as reasonable security practice — a skill SOC analysts use whenever they review vendor proposals, audit findings, or well-intentioned but flawed internal suggestions.

---

## Background — Why Systems Are High-Value Targets

A system is any place data lives: a physical server, a virtual machine, or a cloud platform such as Microsoft 365. The critical insight is one of **scale and blast radius**:

> Compromising one user's mailbox via phishing yields a single mailbox.
> Compromising the mail server yields every mailbox — potentially thousands.

This escalation principle is what makes servers, administrator endpoints, and infrastructure far more valuable to a threat actor than any individual user account.

### System Value by Threat Actor Goal

| Breached System | Attack Value to Adversary |
| --- | --- |
| Personal laptop of a school student | Steal Steam profile; add the host to a botnet |
| Laptop of a bank's senior IT administrator | Pivot into internal banking systems |
| Mail server of a criminal law firm | Dump all mailboxes; blackmail / extortion |
| Server at the heart of an industrial network | Encrypt the entire network with ransomware |
| Government website management panel | Defacement / activism |

The same compromise technique yields wildly different impact depending on *what* is compromised. Prioritising defence by system value is therefore a foundational SOC skill.

---

## How Systems Are Attacked

The room covered four primary attack avenues:

| Attack Avenue | Description |
| --- | --- |
| Human-led attacks | Social engineering, phishing, and direct manipulation of people with system access |
| Vulnerabilities | Exploitation of unpatched software flaws (covered in detail below) |
| Supply chain | Compromising a trusted third-party component, vendor, or dependency to reach the real target |
| Emerging supply chain threat | The growing trend of attackers targeting widely used software dependencies and update mechanisms to achieve mass compromise |

### Software Vulnerabilities & Response

Software vulnerabilities are flaws in code that an attacker can exploit. The primary response is a **patch management process** — a structured cycle of identifying, testing, and applying patches before adversaries can weaponise the flaw.

### Misconfigurations & Response

Misconfigurations are security weaknesses introduced not by flawed code but by incorrect setup — default passwords, excessive permissions, exposed services. The primary responses are **secure-by-default hardening** and **ongoing IT staff training** to prevent recurrence.

---

## Lab Task — Corporate Remediation Plan

### Brief

Acting as the SOC analyst responsible for system hardening, select **four** remediation actions from a provided list that would best protect the organisation's systems at risk. The list deliberately mixed genuine controls with plausible-sounding anti-patterns.

### Available Actions & My Assessment

| Action | Decision | Rationale |
| --- | --- | --- |
| **Antivirus Protection** | ✅ Selected | Effective baseline defence against data stealers and USB worms |
| **Patch Management Policy** | ✅ Selected | Directly reduces the risk of vulnerability exploitation |
| **Secure Password Policy** | ✅ Selected | The core defence against brute-force and credential attacks |
| **Security Training for IT** | ✅ Selected | Addresses misconfigurations at their root cause — human error |
| **Obscure Server Naming** | ❌ Rejected | Security through obscurity; renaming servers `X719I` does not reduce attack surface and gives false assurance |
| **Website Restrictions** | ❌ Rejected | Blocking public access to a public-facing website defeats its business purpose; not a proportionate control |
| **Shared Accounts** | ❌ Rejected | Destroys accountability and audit trails — a direct violation of the principle of individual attribution |

### Verdict — Selected Plan

1. **Antivirus Protection**
2. **Patch Management Policy**
3. **Secure Password Policy**
4. **Security Training for IT**

### Reasoning

The four chosen controls map cleanly onto the attack avenues taught in the room:

- **Patch Management** counters *vulnerabilities*.
- **Security Training for IT** counters *misconfigurations* and *human-led attacks*.
- **Secure Password Policy** counters *credential-based intrusion*.
- **Antivirus Protection** provides a *last-line endpoint defence* against malware delivery.

The three rejected options are instructive because each represents a recognisable security anti-pattern:

- **Obscure Server Naming** relies on *security through obscurity* — discredited because it provides no real protection once an attacker is inside, and breeds complacency.
- **Website Restrictions** is disproportionate — it "solves" a security problem by destroying business functionality, which a mature security function would never accept.
- **Shared Accounts** trades away *accountability* for convenience. Without per-user attribution, incident investigation and forensics become almost impossible — the opposite of what a SOC needs.

### Outcome

Colleague feedback validated all four selections, specifically noting that secure passwords are the only reliable defence against brute-force attacks and that a well-trained IT team is less likely to leave systems exposed.

---

## Key Lessons Carried Forward

1. **Defend by blast radius, not just by asset count.** A mail server is worth a thousand mailboxes — prioritise accordingly.
2. **Recognising bad advice is as important as knowing good controls.** Security through obscurity, shared accounts, and disproportionate lockdowns all *sound* reasonable but fail under scrutiny.
3. **Controls should map to attack avenues.** A good remediation plan deliberately covers vulnerabilities, misconfigurations, credentials, and malware delivery — not four variations of the same defence.
4. **Accountability is non-negotiable.** Any control that erases per-user attribution undermines the entire detection and response capability.
5. **Supply chain is the emerging frontier.** Increasingly, the weakest link is not the target's own systems but a trusted third party.

---

## Tools & Frameworks Referenced

- Patch management lifecycle
- Endpoint protection (antivirus / EDR)
- Defence-in-depth principle
- Principle of individual accountability / least privilege
- Supply chain risk management

---

*Author: Anthony · Documented as part of the TryHackMe SOC Level 1 learning path.*