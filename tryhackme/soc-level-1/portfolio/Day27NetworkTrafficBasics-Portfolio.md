# 🛡️ Network Traffic Basics

**TryHackMe · SOC Level 1 · Network Security and Traffic Analysis Module**
**Date:** 6 August 2026 (Day 27)
**Room:** [Network Traffic Basics](https://tryhackme.com/room/networktrafficbasics)

---

## Overview

This room opens the Network Security and Traffic Analysis module, and it makes an important framing point early:

> Network Traffic Analysis (NTA) is a process that encompasses capturing, inspecting, and analyzing data as it flows in a network. Its goal is to have complete visibility and understand what is communicated inside and outside the network.

Crucially, **NTA is not a synonym for Wireshark**. Wireshark is one tool for one part of the job. NTA proper is the combination of:

- **Correlating several logs** across different sources
- **Deep packet inspection** — reading what's actually inside the traffic
- **Network flow statistics** — the shape and volume of communication over time

The skill the room emphasises is baseline awareness: an L1 analyst has to navigate a sea of network information and recognise what is normal versus what deviates. You cannot spot the anomaly if you don't know the ordinary.

---

## Theory — What Can Be Observed

### Endpoints dominate the traffic volume

**Endpoints generate the most traffic in a network.** Every workstation is browsing, syncing, authenticating, resolving names, checking for updates and talking to file shares. Servers are individually noisy but far fewer in number; endpoints are where the bulk of the volume lives — and, conveniently for an attacker, where malicious traffic is easiest to hide in the noise.

### Authentication precedes access

**Before an SMB session can be established, Kerberos must be contacted first.** On a Windows domain, a workstation reaching for a file share must first obtain a ticket from the KDC (the domain controller) before the SMB session opens.

This ordering matters for two reasons:

1. **Visibility.** The domain controller sees authentication activity for the entire estate, which makes it an extremely high-value observation point.
2. **Detection.** Attacks against Kerberos itself — ticket abuse, unusual ticket request patterns, service account anomalies — appear in that traffic before any file access occurs. The authentication step is the earlier signal.

### Encryption changes what a tap can read

**TLS stands for Transport Layer Security.** It's the protocol that encrypts traffic in transit, and it's the reason a well-placed tap doesn't automatically mean full content visibility. On encrypted traffic the analyst is often working from metadata — endpoints, timing, volume, certificate details — rather than payload. This is exactly why flow statistics and log correlation sit alongside packet inspection in the NTA definition, rather than packet inspection being the whole discipline.

---

## Lab — Network Tap Placement

The exercise is an interactive network diagram. A **TAP** is dragged onto one of several possible positions, and only the correct placement captures the relevant traffic.

> **Objective:** place a network tap in the most efficient location and inspect the network traffic. Only the most efficient location will allow you to view the network traffic; other locations won't (in real-life scenarios, every location would show network traffic).

That parenthetical is the room being honest about its own simplification, and it's worth reading carefully. In reality every tap position captures *something* — the question is whether it captures **what you need**, and whether it does so for **every host** rather than one. "Most efficient" means the choke point that concentrates the traffic you care about.

The topology contained: a workstation (WP1), Internet, a firewall (FW1), a Router, two switches (SW01, SW02), a mail server (SRV-MAIL), a DNS server (SRV-DNS), and several Windows hosts (WIN001–003).

---

### Scenario 1 — Malicious PS Download

> A user using a random workstation clicked on a phishing link and an HTTP request was initiated to download a malicious PowerShell file. If we had wanted to capture this web traffic, where would we have had to place the tap?

**Tap placement: the Web Proxy.**

The room's confirmation explains the logic precisely: all devices in the network send their HTTP(S) requests to this device, and the web proxy then initiates the connection to the destination. Placing the tap there captures all web traffic exiting and entering the network.

**Why this is the right answer and the workstation isn't.** The scenario says *a random workstation* — the analyst doesn't know which host was involved. Tapping WP1 only helps if WP1 happened to be the victim. The proxy is the point where **every** host's web traffic converges, so a single tap covers the whole estate regardless of which endpoint clicked the link.

**Findings from the captured packets:**

- TCP traffic between the internal address 192.168.0.1 and multiple external addresses (93.184.216.34, 104.16.132.229, 203.0.113.55) — bidirectional request/response pairs
- The HTTP response containing the PowerShell script yielded the flag: **`THM{FoundTheMalware}`**

**Analyst takeaway.** A malicious PowerShell download over HTTP is not exotic traffic — it's an ordinary web request carrying a hostile payload. Nothing about the connection metadata marks it out; the maliciousness lives in the response body. This is deep packet inspection earning its place in the NTA definition.

---

### Scenario 2 — DNS Infiltration

> A workstation was compromised, and malicious C2 instructions were infiltrated via DNS TXT records. If we had wanted to capture this DNS traffic, where would we have had to place the tap?

**Tap placement: the DNS server (SRV-DNS).**

The room's confirmation: this device handles all external DNS queries and replies on behalf of the host, which means all external DNS traffic passes through it.

**Findings from the captured packets:**

- UDP traffic between 192.168.0.1 and internal resolvers 10.10.10.10 and 10.10.10.11 — query/response pairs
- The DNS TXT record carrying the C2 instruction yielded the flag: **`THM{C2CommandFound}`**

**Analyst takeaway.** DNS is a superb covert channel precisely because it is universal and rarely blocked — every host must resolve names, so DNS egress is almost always permitted. **TXT records** are the natural abuse vector because they're designed to hold arbitrary text (legitimately: SPF and DMARC records, exactly as seen in the Phishing Prevention room on Day 23).

Detecting this requires content inspection, not connection blocking. The signals are behavioural: unusually long or high-entropy subdomain labels, TXT queries for domains with no business justification, abnormal query volume from a single host, and regular timing intervals suggesting automated beaconing rather than human-driven lookups.

---

## The Pattern Across Both Scenarios

Both correct answers were **service choke points**, not endpoints or perimeter devices:

| Traffic of interest | Correct tap point | Why |
|---|---|---|
| Web / HTTP(S) | Web Proxy | Every host's web traffic is funnelled through it |
| DNS | DNS Server | Every host's external name resolution passes through it |

The transferable principle: **tap where the traffic of interest converges for all hosts, not where a single suspected host sits.** Tapping an endpoint gives deep visibility into one machine; tapping the service choke point gives complete visibility of one traffic category across every machine. For hunting an unknown victim, the second is far more useful.

By extension, and drawing on the Kerberos point from the theory section, the domain controller is the equivalent choke point for authentication traffic.

**The corollary is the harder lesson:** if the sensor isn't in the right place, the traffic never reaches it and no amount of analytical skill recovers it. Visibility is an architecture decision made before the incident, not an analysis decision made during it. When an investigation stalls for lack of data, the root cause is often a tap placement choice made months earlier.

---

## MITRE ATT&CK Mapping

| Scenario | Technique | ID |
|---|---|---|
| Malicious PS Download | Ingress Tool Transfer | [T1105](https://attack.mitre.org/techniques/T1105/) |
| Malicious PS Download | Command and Scripting Interpreter: PowerShell | [T1059.001](https://attack.mitre.org/techniques/T1059/001/) |
| DNS Infiltration | Application Layer Protocol: DNS | [T1071.004](https://attack.mitre.org/techniques/T1071/004/) |

---

## Findings Summary

| Question | Answer |
|---|---|
| Category of devices generating the most traffic | **Endpoint** |
| Service contacted first before an SMB session is established | **Kerberos** |
| What TLS stands for | **Transport Layer Security** |
| Scenario 1 tap placement | **Web Proxy** |
| Scenario 1 flag (HTTP traffic) | **`THM{FoundTheMalware}`** |
| Scenario 2 tap placement | **DNS Server** |
| Scenario 2 flag (DNS traffic) | **`THM{C2CommandFound}`** |

---

## Skills Demonstrated

- Understanding NTA as a composite discipline — log correlation, deep packet inspection and flow statistics — rather than equating it with a single tool
- Reasoning about network visibility architecture: identifying service choke points that concentrate a traffic category across all hosts
- Selecting sensor placement appropriate to the traffic of interest rather than the suspected host
- Recognising protocol abuse patterns: malicious payload delivery over ordinary HTTP, and C2 instruction smuggling via DNS TXT records
- Connecting protocol behaviour to detection opportunity — Kerberos preceding SMB as an earlier authentication signal
- Mapping observed activity to MITRE ATT&CK techniques

## Reflection

The exercise is visually simple — drag a tap onto a diagram — but the underlying question is one of the more consequential decisions in defensive architecture. Every investigation I've done in this course so far has assumed the evidence exists: a PCAP was handed to me, an email sample was on the desktop. This room is about the step before that, and about why the evidence sometimes *doesn't* exist.

The framing that stuck with me is that NTA is not Wireshark. I've just spent time deliberately learning Wireshark, and it would be easy to conflate tool fluency with the discipline. Packet inspection is one input; correlation and flow analysis are equally part of the job, and on encrypted traffic they may be most of it.

The connection back to Day 23 was satisfying. TXT records were the mechanism for SPF and DMARC in the Phishing Prevention room — a legitimate, security-*enhancing* use — and here the same record type carries C2 instructions. Same protocol feature, opposite intent. That's a good illustration of why detection has to look at content and context rather than protocol alone.

**Tags:** `#SOC` `#NetworkTrafficAnalysis` `#NTA` `#PacketCapture` `#NetworkTap` `#DNS` `#C2` `#HTTP` `#Kerberos` `#TLS` `#MITREATTACK` `#BlueTeam` `#TryHackMe`