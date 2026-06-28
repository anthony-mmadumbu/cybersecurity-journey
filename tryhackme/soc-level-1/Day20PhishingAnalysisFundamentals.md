# 🛡️ Phishing Analysis Fundamentals – TryHackMe

## 📅 Date
Day 20 · 28 June 2026

## 📚 What I Learned (3–5 lines max)
- **Email delivery infrastructure** — SMTP (port 25/587), POP3 (110/995), IMAP (143/993), the MUA → MTA → MX → receiving MTA → mailbox flow, and the SPF/DKIM/DMARC authentication trio.
- **Header analysis fundamentals** — reading the `Received:` chain bottom-up (oldest hop first), and spotting mismatches between `From:`, `Return-Path:`, and `Reply-To:` as the primary phishing tell.
- **The phishing family taxonomy** — phishing, spearphishing, whaling, smishing, vishing, BEC, clone phishing, angler phishing. Each variant has a specific channel and target profile.
- **Email body investigation** — embedded links (display text vs `href` value), attachments (file type, hash, double extensions), and the social-engineering pretexts that wrap them.

## 🛠️ What I Did
- Worked through the theory tasks covering email delivery, header analysis, body analysis, and the phishing taxonomy.
- No lab in this room — pure foundational theory to support the applied rooms ahead in the Phishing Analysis module.

## 🔐 Why It Matters
- Phishing analysis is **the single most common day-one task** for a Tier 1 SOC analyst at any MSSP. Around 90%+ of breaches start with phishing, making it the highest-volume incident category by a wide margin.
- **Headers tell the truth that the body can lie about.** Knowing how to read them — particularly the `Received:` chain and `Authentication-Results:` — is non-negotiable for any phishing investigation.
- **Business Email Compromise (BEC) alone accounted for $2.9 billion in reported losses** (FBI IC3, 2023) — making it the highest-impact phishing variant by financial cost, despite being lower-volume than mass phishing.

## ❓ One Thing I Didn't Fully Understand
- The exact policy hierarchy between SPF, DKIM, and DMARC — particularly the alignment rules (relaxed vs strict) and what happens when DMARC `p=none` is set but SPF or DKIM still produce verdicts. The room covered the basics but the interaction between the three authentication mechanisms needs hands-on practice with real header samples to fully internalise.