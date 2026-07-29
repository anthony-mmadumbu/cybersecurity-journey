# 🛡️ Title
Snapped Phish-ing Line – TryHackMe (SOC Level 1 · Phishing Analysis Module)

# 📅 Date
29 July 2026 (Day 26)

# 📚 What I Learned

* How a phishing **campaign** differs from a single phishing email — multiple recipients, multiple lures, one shared adversary infrastructure
* That attackers frequently host kits on **compromised legitimate websites** rather than their own domains, inheriting the site's reputation and age to avoid takedown
* How verbose PHP error messages leak internal server paths — a free reconnaissance gift to the analyst
* That directory listing left enabled on attacker infrastructure exposes the whole operation, including the live credential capture log
* Reading a **phishing kit's source** to find the adversary's own collection address — the operator's email sitting in plain sight inside `submit.php`

# 🛠️ What I Did

* Investigated a multi-recipient phishing campaign at SwiftSpend Financial, where several employees had already submitted credentials
* Reviewed the `phish-emails` folder and extracted the adversary's sending address and recipient list
* Used CyberChef's Extract Domains operation on the HTML attachment to pull out the redirect domain, victim identifier and impersonated brand in one step
* Explored the live phishing infrastructure with `curl`, mapping the deployed kit's directory structure
* Downloaded and hashed the phishing kit, pivoted the hash into VirusTotal, and extracted the archive to read `submit.php`
* Identified a repeat credential-submission victim from the exposed harvest log
* Retrieved and decoded the final flag — full detail in the portfolio piece

# 🔐 Why It Matters

* Campaign-level investigation is what a real SOC does: one reported email is never one email, and scoping the blast radius is the first job
* The exposed log gave a definitive list of who actually submitted credentials — that's the difference between "reset everyone's password" and a targeted, evidence-led response
* The adversary's collection address is a high-value pivot for threat intelligence, potentially linking this campaign to others

# ❓ One Thing I Didn't Fully Understand

* Why the same URL returned different results across requests — one response came from nginx, another from Apache, with the file only present on one. I understand the mechanism is likely load balancing or CDN routing between backends, but I want to learn how to detect and work around inconsistent infrastructure deliberately rather than by retrying until it works

📌 Full investigation write-up: `Day26_Snapped_Phishing_Line_Portfolio.md`