# 🛡️ Title
The Greenholt Phish – TryHackMe (SOC Level 1 · Phishing Analysis Module)

# 📅 Date
24 July 2026 (Day 25)

# 📚 What I Learned

* How to run a full Tier 1 email investigation end to end on a single sample, with no walkthrough — extract artefacts, verify the source, assess the payload, reach a verdict
* Reading `Received:` headers bottom-to-top to find the true originating IP, and filtering out private ranges to isolate the public one
* That the `Reply-To` address is often the real attacker-controlled inbox, even when the `From` domain looks legitimate
* Looking up SPF at the root domain but DMARC at the `_dmarc.` subdomain — a distinction that trips people up
* That a file extension is a claim, not a fact — VirusTotal revealed a `.CAB` attachment was actually a RAR archive

# 🛠️ What I Did

* Investigated a suspicious email escalated by a Greenholt PLC sales executive — generic greeting, unexpected money-transfer request, unsolicited attachment
* Located `challenge.eml` on the VM and extracted headers using terminal commands rather than the GUI, after the visible raw source proved incomplete
* Identified the originating IP as **192.119.71.157** and attributed it to **HostPapa** via OSINT
* Ran SPF and DMARC lookups on the Return-Path domain `mutawamarine.com` using `dig`
* Hashed the attachment with `sha256sum` and pivoted the hash into VirusTotal for file size and true file type
* Full investigation write-up in the portfolio piece

# 🔐 Why It Matters

* This is the closest thing yet to an actual SOC ticket — a user report arrives, and the analyst has to decide legitimate or malicious, with evidence
* The artefacts extracted here (originating IP, spoofed domain, reply-to address, file hash) are exactly what feeds blocklists, detection rules and the incident record
* Business Email Compromise attacks like this one cost organisations more than any other phishing category, and they succeed precisely because the sender domain often *is* real

# ❓ One Thing I Didn't Fully Understand

* Why the receiving mail server logged `dmarc=unknown` in the headers when the domain does publish a valid DMARC record (`p=quarantine`). I want to understand what causes a DMARC evaluation to come back unknown rather than pass or fail — whether it's an alignment issue, a lookup failure, or something in how the receiver processed it

📌 Full investigation write-up: `Day25_The_Greenholt_Phish_Portfolio.md`