🛡️ Systems as Attack Vectors – TryHackMe

SOC Level 1 · Blue Team Introduction

📅 Date

Day 4


📚 What I learned

What a system is — a physical server, virtual machine, or cloud platform (like Microsoft 365) where data such as bank cards and emails is stored

Why systems are high-value targets — breaching one user's mailbox via phishing compromises a single mailbox, but breaching the mail server compromises thousands. Scale is everything to a threat actor

System value varies by attacker goal — a student's laptop might be added to a botnet, while a bank IT admin's laptop grants access to internal banking systems, and an industrial server can be ransomwared to encrypt a whole network

How systems are attacked — human-led attacks, vulnerabilities, supply chain attacks, and the emerging threat of supply chain compromise

Software vulnerabilities and how to respond to them — primarily through patch management

Misconfigurations and how to respond to them — through hardening, training, and secure defaults


🛠️ Lab — Remediation Plan

Tasked as a SOC analyst to assess the Systems at Risk and select the 4 best measures to protect them from a list that included deliberate distractors.

✅ My selections (all 4 correct):

Antivirus Protection — defends against data stealers and USB worms
Patch Management Policy — reduces exploitation risk from known vulnerabilities
Secure Password Policy — protects against brute-force attacks
Security Training for IT — reduces misconfigurations at the source


❌ Distractors I correctly rejected:

Obscure Server Naming — security through obscurity is not real security
Website Restrictions — blocking public access defeats the purpose of a public site
Shared Accounts — destroys accountability and audit trails


🔐 Why it matters

Systems are high-value because compromising one can cascade — the difference between one mailbox and an entire mail server is the difference between an incident and a catastrophe
A good SOC analyst must distinguish genuine security controls from plausible-sounding bad advice — recognising anti-patterns like security-through-obscurity and shared accounts is a core skill


❓ One thing I didn't fully understand

All 4 remediation actions selected correctly — feedback confirmed every choice was valid 🎯