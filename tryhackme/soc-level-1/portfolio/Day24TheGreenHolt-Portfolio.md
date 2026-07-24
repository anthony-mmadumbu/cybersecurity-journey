# 🛡️ The Greenholt Phish — Full Email Investigation

**TryHackMe · SOC Level 1 · Phishing Analysis Module (Challenge Room)**
**Date:** 24 July 2026 (Day 25)
**Room:** [The Greenholt Phish](https://tryhackme.com/room/phishingemails5fgjlzbx)
**Evidence sample:** `challenge.eml`

---

## The Brief

A sales executive at **Greenholt PLC** reported a suspicious email that appeared to come from a known customer. Three things did not sit right with them:

- A **generic greeting**, where this customer normally addresses them by name
- An **unexpected request for a money transfer**
- An **unsolicited attachment**

The employee's judgement was that the message did not match the customer's usual communication style. The report was escalated to the SOC. My job: analyse the sample and determine whether it is legitimate correspondence or a phishing attempt.

This was a challenge room — no teaching content, no guided walkthrough. Just the sample and a set of questions.

---

## Investigation Methodology

### 1. Getting to the evidence

The email sample sat on the VM desktop as `challenge.eml`. Finding it took a moment — `/root/Desktop` returned permission denied, so I enumerated user directories instead:

```bash
ls /home/*/Desktop
find / -name "*.eml" 2>/dev/null
```

### 2. Why I abandoned the GUI

My first instinct was to read the raw source in the mail client's viewer. That proved unreliable — the visible portion showed `Received:` header IPs rendered as `x.x.x.x`, and the only genuine address on screen (`10.201.192.162`) was an internal relay hop, not the origin.

Rather than scroll through a truncated view, I moved the whole analysis to the terminal. This turned out to be the right call and became my default approach for the rest of the investigation.

```bash
grep -i "received" /home/<user>/Desktop/challenge.eml
```

`Received:` headers are prepended by each server as the message travels, so they read **bottom-to-top** — the last line printed is the earliest hop, where the message actually originated. Cross-checking against private ranges (10.x, 172.16–31.x, 192.168.x, 127.x) confirmed which address was genuinely public.

### 3. OSINT from a separate host

The VM had no outbound connectivity — `whois` failed with *Network is unreachable*. Rather than treat that as a blocker, I ran the reputation and DNS lookups from my own machine. Worth noting as a practical habit: the analysis host and the OSINT host don't have to be the same, and keeping detonation/inspection isolated from lookup activity is good practice anyway.

### 4. DNS record lookups

```bash
dig +short TXT mutawamarine.com          # SPF — published at the root domain
dig +short TXT _dmarc.mutawamarine.com   # DMARC — published at the _dmarc subdomain
```

The `_dmarc.` prefix is the detail that catches people out. Querying the root domain for DMARC returns nothing, which is easily misread as "no DMARC policy exists."

### 5. Attachment handling

The attachment was downloaded to the VM and hashed **without opening it**:

```bash
sha256sum /home/<user>/Desktop/SWT_#09674321____PDF__.CAB
```

The hash was then pivoted into VirusTotal for file size, true file type, and detection verdicts — hash lookup rather than file upload, so nothing left the environment.

---

## Findings

### Message artefacts

| Question | Answer |
|---|---|
| Transfer Reference Number in the Subject line | **09674321** |
| Display name of the sender | **Mr. James Jackson** |
| Sender's email address | **info@mutawamarine.com** |
| Address that will receive a reply | **info.mutawamarine@mail.com** |

### Message source

| Question | Answer |
|---|---|
| Originating IP address | **192.119.71.157** |
| Owner of the originating IP | **HostPapa** |

### Domain authentication

| Question | Answer |
|---|---|
| Full SPF record for the Return-Path domain | **`v=spf1 include:spf.protection.outlook.com -all`** |
| Complete DMARC record for the Return-Path domain | **`v=DMARC1; p=quarantine; fo=1`** |

### Attachment

| Question | Answer |
|---|---|
| Attachment file name | **`SWT_#09674321____PDF__.CAB`** |
| SHA256 hash | **`2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f`** |
| File size (per VirusTotal) | **400.26 KB** |
| Actual file type | **RAR** |

---

## Analysis

### The Reply-To mismatch is the tell

The `From` address is `info@mutawamarine.com` — a real domain belonging to a real company. But replies route to **`info.mutawamarine@mail.com`**, a free webmail account crafted to *look* like the corporate address by folding the domain into the local part.

This is the mechanism that makes Business Email Compromise work. The recipient sees a familiar sender, hits reply, and the conversation silently moves to an attacker-controlled inbox. Any wire transfer details negotiated from that point onward come from the attacker.

**The lesson:** checking the `From` field alone is insufficient. `From`, `Return-Path` and `Reply-To` should be compared against each other, and a mismatch between them is a strong signal on its own.

### SPF: correctly configured, and it caught the spoof

```
v=spf1 include:spf.protection.outlook.com -all
```

| Component | Meaning |
|---|---|
| `v=spf1` | SPF version 1 record |
| `include:spf.protection.outlook.com` | Only Microsoft 365 / Exchange Online infrastructure is authorised to send for this domain |
| `-all` | **Hard fail** — reject everything else |

The originating IP, 192.119.71.157, belongs to **HostPapa** — a shared web hosting provider, not Microsoft. Under this policy, the message is unambiguously unauthorised. The headers reflected this with an **SPF fail**.

This is a case of the domain owner having done their job properly. `mutawamarine.com` published a strict SPF policy, and it correctly identified the message as fraudulent. The failure was on the receiving side, where the message was delivered to the user's inbox rather than blocked.

### DMARC: policy published, evaluation unclear

```
v=DMARC1; p=quarantine; fo=1
```

| Component | Meaning |
|---|---|
| `v=DMARC1` | DMARC version 1 |
| `p=quarantine` | On failure, deliver to spam/junk rather than the inbox |
| `fo=1` | Generate forensic reports if *either* SPF or DKIM fails alignment |

**A discrepancy worth flagging:** the email headers recorded `dmarc=unknown`, yet the live lookup returns a valid, published policy. The domain owner's configuration is sound — so the gap sits with the receiving infrastructure's evaluation of it, not with the sending domain. Whatever the cause, the practical outcome is that a message which should have been quarantined under `p=quarantine` reached a user's inbox instead.

It's also worth noting that `p=quarantine` is a middle-tier policy. Under `p=reject`, the strongest setting, the message would have been refused outright rather than filed to junk.

### The attachment: masquerading, twice over

The filename is `SWT_#09674321____PDF__.CAB`, and it deserves unpacking:

1. **`SWT_#09674321`** — "SWT" reads as SWIFT (international wire transfer), and the reference number matches the subject line. This is engineered to look like a legitimate payment confirmation the recipient was expecting.
2. **`____PDF__`** — the underscore padding is designed to push the real extension out of view in clients that truncate long filenames, so the user sees something that appears to end in "PDF".
3. **`.CAB`** — the declared extension is a Windows Cabinet archive.
4. **Actually RAR** — VirusTotal's file-type identification, based on the file's magic bytes rather than its name, shows the true format is **RAR**.

So the file lies about its type at two levels: the name suggests a PDF, the extension claims CAB, and the content is RAR. Archive formats are a common phishing wrapper because they defeat naive attachment scanning and require the user to take an extra deliberate step to reach the payload inside.

**MITRE ATT&CK mapping:**

| Technique | ID |
|---|---|
| Phishing: Spearphishing Attachment | [T1566.001](https://attack.mitre.org/techniques/T1566/001/) |
| Masquerading: Double File Extension | [T1036.007](https://attack.mitre.org/techniques/T1036/007/) |

---

## IOC Summary

| Type | Indicator |
|---|---|
| Sender address (spoofed) | `info@mutawamarine.com` |
| Reply-To (attacker-controlled) | `info.mutawamarine@mail.com` |
| Originating IP | `192.119.71.157` |
| IP owner / hosting provider | HostPapa |
| Attachment filename | `SWT_#09674321____PDF__.CAB` |
| Attachment SHA256 | `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f` |
| True file type | RAR archive (declared as CAB, named as PDF) |
| Subject reference | Transfer Reference Number 09674321 |

---

## Verdict

**Malicious — confirmed phishing, Business Email Compromise pattern with a weaponised attachment.**

The evidence chain:

1. The sending IP is not authorised by the spoofed domain's SPF record, which specifies a hard fail
2. The originating IP belongs to a shared hosting provider, not the customer's actual mail infrastructure
3. Replies redirect to a free webmail account impersonating the corporate address
4. The attachment misrepresents its file type at multiple levels
5. The social engineering profile — generic greeting, unexpected transfer request, unsolicited attachment — matches the reporting user's own instinct that something was wrong

### Recommended actions

**Immediate**
- Block `192.119.71.157` at the perimeter
- Block sender and reply-to addresses at the mail gateway
- Add the SHA256 to endpoint blocklists
- Search mail logs for other recipients of the same campaign and purge from mailboxes

**Follow-up**
- Confirm no user replied to or opened the attachment; if any did, treat as a potential compromise
- Contact the genuine customer out-of-band — their brand is being abused and they may not know
- Review why a message failing SPF against a `-all` policy reached an inbox, and investigate the `dmarc=unknown` evaluation
- Verify any pending payment instructions relating to reference 09674321 through a known-good phone number

**Positive note for the record:** the user reported this rather than acting on it. That is the control that worked, and it should be recognised in any awareness reporting.

---

## Skills Demonstrated

- End-to-end Tier 1 email investigation on an unguided sample, from user report to documented verdict
- Header forensics via terminal: `grep` on `Received:` chains, reading hop order correctly, distinguishing public from private addressing
- Adapting method when the initial approach failed — moving from a truncated GUI view to command-line extraction
- Working around environment constraints by relocating OSINT to a connected host
- DNS-based authentication analysis: SPF and DMARC record retrieval with `dig`, including the `_dmarc.` subdomain distinction, and interpretation of every mechanism and tag
- Safe attachment handling: hash-first, hash-lookup rather than upload, no execution
- File-type verification through magic bytes rather than trusting the declared extension
- IOC extraction and MITRE ATT&CK technique mapping
- Producing actionable containment recommendations rather than stopping at the verdict

## Reflection

This was the first room in the module with no scaffolding, and it took several sessions across multiple days. The parts that slowed me down were environmental rather than analytical — locating the file, hitting permission walls, and losing time to a VM with no network access before deciding to move the lookups elsewhere. Those detours were genuinely useful in the end. Real investigations rarely happen in a tidy environment, and being willing to change approach rather than push at a blocked one is part of the job.

The finding that will stay with me is the SPF result. `mutawamarine.com` had done everything right — a strict `-all` policy and a published DMARC record — and the phishing email still landed in a user's inbox. Sender-side authentication only helps if the receiving side acts on it. The last line of defence was the sales executive noticing that the greeting felt wrong.

**An honest note on documentation:** I worked this room across several days without recording my method as I went, and reconstructed the process afterwards from my working notes. The answers and artefacts are exactly as submitted and accepted, but the sequencing above is reconstructed rather than logged live. Documenting reasoning *during* an investigation, not after, is the habit I'm taking into the next room.

**Tags:** `#SOC` `#PhishingAnalysis` `#EmailForensics` `#BEC` `#SPF` `#DMARC` `#IOC` `#MITREATTACK` `#VirusTotal` `#BlueTeam` `#TryHackMe`