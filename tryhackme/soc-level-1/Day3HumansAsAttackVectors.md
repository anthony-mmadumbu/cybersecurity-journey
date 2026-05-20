🛡️ Humans as Attack Vectors – TryHackMe

SOC Level 1 · Blue Team Introduction

📅 Date

Day 3

📚 What I learned

Why humans are targeted — people are often the easiest entry point into an organisation because technical defences can be bypassed by manipulating trust
The kinds of attacks targeted at humans — phishing, vishing, smishing, business email compromise, deepfake impersonation, and social engineering via phone
Defending humans — through awareness training, verification policies, technical controls (anti-phishing, AV), and clear escalation paths
🛠️ Lab Scenarios — My Verdicts

1️⃣ Lucas Martinez — Unofficial 7-Zip download

A new software engineer urgently needed 7-Zip but it wouldn't launch. Investigation revealed a Setup.exe downloaded from best-freeapps-2025.top, blocked by antivirus 6 times.

Verdict: Quarantine the Setup.exe and instruct Lucas to use the official 7-Zip installer. The download wasn't from the official source, and AV blocking it 6 times is a strong indicator, not a false positive.

2️⃣ Mark Phillips — Suspicious "Stripe" invoice email

An email claimed a $23,650 payment confirmation from Stripe, sent from noreply@stripe-payments.xyz but asking recipients to contact support@stripe.com. Included a password-protected RAR attachment with the password in plain text.

Verdict: Block the email and start analysis — it's phishing. Domain mismatch between sender and contact address, suspicious .xyz TLD for a global brand, password in plain text (unprofessional and a technique to bypass email scanners), and the urgency of a large invoice are all red flags.

3️⃣ CEO Ben — Out-of-hours password reset

IT Support received a call at 9 PM from a hidden number, claiming to be CEO Ben, requesting a Gmail password reset. Logs showed a login from the USA (Ben's country), but Ben was unreachable for verification.

Verdict: Disable Ben's Gmail account until he confirms the login or returns to the office. All behaviour is atypical of a CEO — hidden number, out-of-hours timing, and being unreachable for verification. The matching USA login doesn't rule out an attacker using a VPN to bypass geolocation checks.

4️⃣ Rose Lewis — Anomalous login location

SIEM flagged a login from London (typical: Oxford). URLs visited before login included http://login[.]micrsoft365-online[.]ru and https://hroyhiqtspqgkp[.]info.

Verdict: Disable Rose Lewis' account. All visited URLs are illegitimate — the first is a typosquatted Microsoft domain on a Russian TLD, and the second is a random obfuscated URL. Rose most likely entered her credentials on the fake login page, and the attacker is now using them. The geographic proximity is a distractor, context (visited URLs) matters more than the SIEM alert title.

5️⃣ Final Task — Corporate Security Policy Selection

Selected 4 corporate policies to defend against the kinds of attacks worked through in the room. Each choice was validated in the feedback received from colleagues and other departments.

Access Management Policy — to defend against social engineering attempts like the Ben scenario
Anti-Phishing Solution — to catch threats like the Stripe email at the gateway
Security Awareness Program — to train staff to recognise attacks like the one Rose fell for
Antivirus Solution — to act as a last line of defence, as it did with Lucas's download


🔐 Why it matters

Humans remain the most exploited attack vector, most major breaches start with a successful social engineering or phishing attempt.
SOC analysts must learn to read context beyond what an alert literally says, the visited URLs, sender domains, and behavioural anomalies tell the real story


❓ One thing I didn't fully understand

All scenarios completed with correct verdicts, the policy task feedback validated every choice 🎯