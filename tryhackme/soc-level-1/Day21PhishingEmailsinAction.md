# 🛡️ Phishing Emails in Action – TryHackMe

## 📅 Date
Day 21 · 29 June 2026

## 📚 What I Learned (3–5 lines max)
- **Social engineering tactics in phishing** — authority, urgency, fear, scarcity, reciprocity, and social proof (Cialdini's six principles of influence, applied to attacker pretexts).
- **Link manipulation techniques** — display-text vs `href` mismatch, URL shorteners (bit.ly, t.co), typosquatting (`paypa1.com`), homoglyph/Punycode attacks (Cyrillic 'а' as Latin 'a'), and subdomain spoofing (`paypal.security-update.com`).
- **Tracking pixels** — 1×1 invisible images embedded in HTML email that report back the recipient's IP, user-agent, and approximate geolocation when the email is opened. The defensive control is **blocking remote image loading by default** for unknown senders.
- **Credential harvesting and attachment manipulation** — fake login pages cloned from real ones, HTML attachments containing local login forms, ISO/IMG containers that bypass Mark of the Web (MOTW), HTML smuggling, double extensions, and macro-enabled documents.

## 🛠️ What I Did
- Worked through theory tasks examining real phishing email samples and the attacker tactics they demonstrate.
- No lab in this room — applied theory built on top of the fundamentals from Day 20, preparing for the practical case investigations in Day 22.

## 🔐 Why It Matters
- These are **the attacker techniques you'll see in production every day**. The difference between a SOC analyst who triages quickly and one who fumbles is recognition speed — and recognition speed comes from having seen these tactics named and decomposed.
- Understanding *why* a phishing email is effective (which Cialdini principle it leverages, which technical evasion it uses) makes you faster at spotting them and **better at writing user-awareness training that actually changes behaviour** rather than just ticking a compliance box.

## ❓ One Thing I Didn't Fully Understand
- **HTML smuggling** specifically — the JavaScript-reconstructs-binary-client-side technique. The concept is clear (the malicious file is built in the browser, not transmitted as an attachment, bypassing email gateway scanning) but I'd want to see a worked example with the actual JavaScript and the binary reconstruction to understand the bypass in detail. This is a technique that has surged in real-world attacks since 2022.