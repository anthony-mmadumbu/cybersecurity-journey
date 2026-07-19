# 🛡️ Title
Phishing Prevention – TryHackMe (SOC Level 1 · Phishing Analysis Module)

# 📅 Date
19 July 2026 (Day 23)

# 📚 What I Learned

* How the email authentication stack — SPF, DKIM, DMARC and S/MIME — works together to verify senders and protect message integrity
* SPF results (Pass / Fail / SoftFail) and what each means for mail handling — SoftFail means the message is flagged as suspicious but still accepted
* DKIM failure states, including `permerror` caused by "no key for signature"
* DMARC policies (`p=none`, `p=quarantine`, `p=reject`) — with `p=reject` giving the strongest protection
* How to read SMTP response codes (220, 250, 354, 5xx rejections) directly from network traffic

# 🛠️ What I Did

* Worked through the theory tasks on SPF, DKIM, DMARC and S/MIME
* Paused mid-room at Task 6 to complete 11 tasks of the Wireshark room so I could do the traffic analysis properly — then returned and applied it
* Analysed `traffic.pcap` in Wireshark: filtered on `smtp.response.code`, traced a Spamhaus 553 block, counted 552 security rejections, and inspected email internals with the `imf` filter
* Identified a malicious `attachment.scr` (base64-encoded) sent via Microsoft Outlook Express — full detail in the portfolio write-up
* Answered the anti-phishing controls scenario (sandboxing for safe detonation of suspicious attachments)

# 🔐 Why It Matters

* SPF/DKIM/DMARC results appear in every email header a SOC analyst triages — knowing what `softfail` or `permerror` actually means turns header noise into evidence
* SMTP response codes in a PCAP show *why* mail was blocked or bounced, which is often the first clue in a phishing or spam investigation
* Defence is layered: authentication protocols + gateway filtering + sandboxing + user reporting

# ❓ One Thing I Didn't Fully Understand

* The exact interaction when SPF passes but DKIM fails (or vice versa) and how DMARC alignment decides the final verdict — I want to trace a few real headers end-to-end to cement this

📌 Full lab write-up: `Day23_Phishing_Prevention_Portfolio.md`