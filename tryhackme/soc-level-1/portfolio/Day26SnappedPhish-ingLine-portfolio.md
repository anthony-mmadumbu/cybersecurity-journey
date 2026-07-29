# 🛡️ Snapped Phish-ing Line — Phishing Campaign & Kit Analysis

**TryHackMe · SOC Level 1 · Phishing Analysis Module**
**Date:** 29 July 2026 (Day 26)
**Room:** [Snapped Phish-ing Line](https://tryhackme.com/room/snappedphishingline)
**Evidence:** `phish-emails/` folder, live adversary infrastructure at `kennaroads.buzz`

---

## The Brief

As a member of the IT department at **SwiftSpend Financial**, I handle employee technical concerns. What looked like a routine day escalated when **multiple employees across different departments** reported a suspicious email. Several noticed unusual characteristics — but some had already submitted their credentials and could no longer access their accounts.

With a wider compromise in play, the incident was escalated. My task: analyse the evidence, determine the **scope** of the attack, and uncover how the adversary operated.

This is a step up from analysing a single sample. The unit of investigation here is a **campaign**, and the questions are correspondingly broader: who was targeted, who fell for it, what infrastructure was used, and who is behind it.

---

## Investigation Methodology

### Phase 1 — Email triage

The `phish-emails` folder on the VM desktop held multiple samples. Working through them established the campaign shape: different lures aimed at different employees, all traceable to a single sending address.

One practical friction point worth recording: the attachment filenames contained spaces (`Direct Credit Advice.html`), and my initial attempts at backslash escaping failed. The reliable fix was to `cd` into the directory and quote the filename:

```bash
cd ~/Desktop/phish-emails
ls
file "Direct Credit Advice.html"
```

Small thing, but it cost time. Quoting beats escaping.

### Phase 2 — Attachment analysis with CyberChef

Rather than hand-crafting regex to pull URLs out of the HTML attachment, I ran it through **CyberChef's Extract Domains** operation. One step returned everything of interest:

```
kennaroads.buzz
zoe.duncan
swiftspend.finance
```

Three artefacts, three distinct meanings:

| Extracted | What it is |
|---|---|
| `kennaroads.buzz` | The redirect destination — attacker-controlled |
| `zoe.duncan` | The victim identifier, embedded in the URL as a tracking parameter |
| `swiftspend.finance` | The legitimate employer domain, present because the lure pre-fills the victim's real address |

The victim identifier in the URL is worth pausing on. The link is **personalised per recipient**, which means the phishing page can pre-fill the email field and look more convincing — and it means the attacker knows exactly who clicked.

The equivalent manual approach, for reference:

```bash
grep -oE 'https?://[^"'\''<>]+' "Direct Credit Advice.html"
grep -iE "http-equiv|refresh|window.location|location.href|action=" "Direct Credit Advice.html"
```

### Phase 3 — Live infrastructure reconnaissance

With `kennaroads.buzz` identified, I moved to mapping the deployed kit using `curl` from the VM.

**Discovery 1 — the host is a compromised legitimate site.** The root of `kennaroads.buzz` serves a WordPress blog called *Kenna Roads*. The phishing kit is planted underneath it while the main site continues to look entirely innocuous. This is deliberate: a compromised legitimate site brings domain age, existing reputation and SEO history, all of which help it evade the filters that catch freshly registered domains.

**Discovery 2 — verbose PHP errors leak the server path.** Requesting the kit's `index.php` returned unhandled PHP warnings:

```
Warning: Undefined array key "email" in
/var/www/html/wp/data/Update365/office365/40e7baa2f826a57fcf04e5202526f8bd/index.php on line 15
```

That single error line disclosed the full internal filesystem path, confirmed the WordPress installation at `/var/www/html/wp/`, and revealed the kit's deployment directory including its randomised hash folder. The errors appear because the page expects an `email` parameter supplied by the phishing link — requesting it directly, without that parameter, breaks it open.

**Discovery 3 — directory listing is enabled.** Browsing `/data/` returned a listing rather than a denial:

```
log.txt        2026-07-29 06:22    2.9K
office365/     2020-01-13 10:01      -
```

`log.txt` was **modified the same day I was investigating** — the credential harvest log, still actively collecting. The attacker's own operational sloppiness handed over both the kit archive and the victim list.

### Phase 4 — Phishing kit retrieval and analysis

```bash
sha256sum Update365.zip
unzip Update365.zip -d Update365_extracted
```

Hash pivoted into VirusTotal for classification and file count, then the archive extracted to read the source — specifically `Validation/submit.php`, the script that handles captured credentials, which is where a kit's operator typically hardcodes their own collection address.

### Phase 5 — The flag hunt

This took the better part of three days and is written up separately below, because the difficulty was instructive.

---

## Findings

### Campaign scope

| Question | Answer |
|---|---|
| Individual who received the "Quote for Services Rendered" email | **William McClean** |
| Adversary's sending address | **`Accounts.Payable@groupmarketingonline.icu`** |

### Attachment and redirection

| Question | Answer |
|---|---|
| Root domain of the redirection URL (Zoe Duncan's attachment) | **`kennaroads.buzz`** |
| Company the login page impersonates | **Microsoft** |

### Phishing kit

| Question | Answer |
|---|---|
| Archive file in `/data` | **`Update365.zip`** |
| SHA256 of the kit | **`ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686`** |
| Additional VirusTotal threat category (besides phishing) | **Trojan** |
| Files contained in the archive | **49** |

### Victims and adversary

| Question | Answer |
|---|---|
| User who submitted credentials more than once | **`michael.ascot@swiftspend.finance`** |
| Adversary's credential collection address (from `submit.php`) | **`m3npat@yandex.com`** |
| Decoded flag | **`THM{pL4y_w1Th_tH3_URL}`** |

---

## Analysis

### The sending domain is doing a lot of work

`Accounts.Payable@groupmarketingonline.icu` is carefully built. The local part imitates a finance department mailbox, which fits the "Quote for Services Rendered" and "Direct Credit Advice" lures — invoice and payment themes aimed at people who handle money as part of their job.

The `.icu` TLD is the tell. It's a cheap, low-friction registration that appears heavily in abuse data. TLD reputation isn't proof on its own, but for a supposed business correspondent it's a meaningful anomaly worth weighting during triage.

### Two-stage delivery: HTML attachment, then redirect

The attack does not put the phishing link directly in the email body. Instead:

1. An **HTML attachment** arrives with the message
2. Opening it **redirects** to `kennaroads.buzz`
3. The victim lands on a **Microsoft credential harvesting page**, email pre-filled from the URL parameter

This defeats email gateways that scan message bodies for known-bad URLs — the destination never appears in the email itself, only inside an attached file. The pre-filled email address then makes the fake login page more convincing, since it appears to already know who the user is.

### Compromised legitimate hosting

The kit lives on a hijacked WordPress site. This matters for defence: blocking `kennaroads.buzz` outright also blocks a real, innocent blog, and the domain carries none of the age or reputation signals that flag newly registered phishing infrastructure. Detection has to key on the deployed path and behaviour rather than the domain's surface reputation.

It also means there is a **second victim** here — the owner of Kenna Roads, whose site was breached and is now hosting a credential harvester, quite possibly without their knowledge.

### The kit itself

VirusTotal classified `Update365.zip` as both **phishing** and **Trojan**, across **49 files** — not a single fake login page but a full toolkit. Reading the source revealed the operator's collection address, **`m3npat@yandex.com`**, hardcoded in `submit.php`.

That address is the single most valuable artefact in this investigation. Phishing kits are frequently resold and reused, and the collection address is often left unchanged across deployments. It's a strong pivot for threat intelligence — searching it across kit repositories and threat feeds can link this campaign to others by the same operator.

### Scoping the compromise

The exposed `log.txt` turned a guessing exercise into a definitive one. **`michael.ascot@swiftspend.finance`** submitted credentials more than once — most likely because the kit returned a fake error on the first attempt and asked him to re-enter, a common technique to capture a correctly typed password after a possible typo.

Two submissions means two things operationally: the credentials are almost certainly valid, and this account should be treated as the highest priority in the response.

### MITRE ATT&CK mapping

| Technique | ID |
|---|---|
| Phishing: Spearphishing Attachment | [T1566.001](https://attack.mitre.org/techniques/T1566/001/) |
| Phishing: Spearphishing Link | [T1566.002](https://attack.mitre.org/techniques/T1566/002/) |
| Acquire Infrastructure: Compromise Infrastructure | [T1584.004](https://attack.mitre.org/techniques/T1584/004/) |
| Credentials from Password Stores / Valid Accounts | [T1078](https://attack.mitre.org/techniques/T1078/) |

---

## The Flag Hunt — What Took Three Days

> *Return to the phishing URL and locate the `flag.txt` file. Using CyberChef to decode the flag, what is the secret value?*

This question took me nearly three days, and it's the part of the room I learned the most from.

**What went wrong.** I guessed at paths. `curl http://kennaroads.buzz/flag.txt`, then `/office365/flag.txt`, then `/Validation/flag.txt`, then case variations, then the same set over HTTPS. All 404. I then went down a dead end assuming the flag must be inside the extracted kit rather than on the live server, and spent time grepping the archive for `THM{` patterns that were never there.

**What eventually worked.** Two things.

First, I switched from guessing filenames to **mapping the structure**. The directory listing at `/data/` and the path disclosed by the PHP error gave me the actual deployment path rather than my assumptions about it.

Second — and this is the part I would not have predicted — **the same request returned different results on different attempts**. Comparing response headers across requests showed one served by `nginx/1.23.4` and another by `Apache/2.4.56 (Debian)`. Two different backends were answering for the same URL, and the file existed on only one of them. Using `curl -I` to check status codes cheaply, and simply retrying, eventually returned the file.

That's a genuinely useful lesson. In a lab you assume a request is deterministic. Against real infrastructure sitting behind a load balancer or CDN, a 404 may mean "not there" or it may mean "not there *on this backend*". Retrying and comparing `Server:` headers is now part of how I approach web reconnaissance.

**The decode.** The retrieved file contained:

```
fUxSVV8zSHRfaFQxd195NExwe01IVAo=
```

Running From Base64 alone produced `}LRU_3Ht_hT1w_y4Lp{MHT` — recognisably a flag, but backwards. The room's CyberChef recipe had **two** operations and I had applied one. Adding **Reverse (Character)** completed it:

```
THM{pL4y_w1Th_tH3_URL}
```

The flag is `pL4y_w1Th_tH3_URL` — "play with the URL". Three days spent doing exactly that, before realising that was the intended lesson rather than an obstacle to it.

---

## IOC Summary

| Type | Indicator |
|---|---|
| Adversary sending address | `Accounts.Payable@groupmarketingonline.icu` |
| Adversary sending domain | `groupmarketingonline.icu` |
| Adversary collection address | `m3npat@yandex.com` |
| Phishing host (compromised) | `kennaroads.buzz` |
| Deployed kit path | `/data/Update365/office365/40e7baa2f826a57fcf04e5202526f8bd/` |
| Disclosed server path | `/var/www/html/wp/data/Update365/...` |
| Kit archive | `Update365.zip` |
| Kit SHA256 | `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686` |
| Malicious attachment | `Direct Credit Advice.html` |
| Confirmed compromised account | `michael.ascot@swiftspend.finance` |
| Brand impersonated | Microsoft |

---

## Verdict & Recommended Actions

**Confirmed multi-recipient credential harvesting campaign against SwiftSpend Financial, with at least one successful compromise.**

**Immediate**
- Force password reset and revoke active sessions for `michael.ascot@swiftspend.finance`; treat as compromised, not merely targeted
- Review that account for post-compromise activity: mailbox rules, forwarding, OAuth grants, unusual sign-in locations, MFA device changes
- Cross-reference the full harvest log against the employee directory to identify every submission, not just the repeat one
- Block the sending address and `groupmarketingonline.icu` at the gateway
- Block the deployed kit path; consider whether blocking `kennaroads.buzz` wholesale is proportionate given it is a compromised legitimate site
- Add the kit SHA256 to endpoint blocklists
- Search mail logs for all recipients of the campaign and purge remaining copies

**Follow-up**
- Notify the owner of `kennaroads.buzz` — their site is breached and hosting a credential harvester
- Submit `m3npat@yandex.com` and the kit hash to threat intelligence sharing, and check whether the collection address links to other known campaigns
- Review whether HTML attachments should be blocked or sandboxed by default at the gateway — this campaign's entire evasion strategy depended on the URL not appearing in the message body
- Confirm MFA coverage across the organisation; harvested credentials alone should not be sufficient for account takeover
- Targeted awareness refresh for finance-adjacent roles, using this campaign's actual lures

---

## Skills Demonstrated

- Campaign-level phishing investigation: scoping across multiple recipients and lures rather than analysing a single sample
- Efficient artefact extraction using CyberChef (Extract Domains, From Base64, Reverse) and the equivalent `grep` regex approach
- Live adversary infrastructure reconnaissance with `curl`, including HEAD requests for cheap enumeration
- Reading information disclosure as evidence: PHP path leakage and directory listing turned into a structural map of the deployment
- Phishing kit analysis: archive hashing, VirusTotal classification pivot, and source review to extract the operator's collection address
- Compromise scoping from an exposed harvest log, and correctly prioritising a repeat submitter
- Recognising infrastructure inconsistency (multi-backend routing) and adapting reconnaissance technique accordingly
- IOC compilation, MITRE ATT&CK mapping, and evidence-led containment recommendations

## Reflection

The single biggest shift from the previous room is the change in unit of analysis. Greenholt asked "is this email malicious?" This room asked "how far has this gone, and who is behind it?" — and answering that required moving beyond the samples into the attacker's own infrastructure. That's the transition from analysing an artefact to investigating an incident.

The flag hunt was frustrating but formative. Three days of 404s taught me more about web reconnaissance than a first-try success would have: map structure rather than guess filenames, read error messages as evidence rather than noise, and don't assume a negative response is deterministic. The flag literally spelling out `pL4y_w1Th_tH3_URL` was a fair joke at my expense.

The finding I'd bring to an interview is the compromised WordPress host. It reframes the defensive problem: domain reputation is a weak signal when adversaries can simply borrow someone else's, and it means every phishing takedown potentially involves a third party who is themselves a victim.

**A note on documentation:** as with the previous room, I worked this over several days without recording my method live, and reconstructed the process afterwards from my working notes. Answers and artefacts are exactly as submitted and accepted. Logging reasoning during the investigation rather than after remains the habit I'm building.

**Tags:** `#SOC` `#PhishingAnalysis` `#PhishingKit` `#CredentialHarvesting` `#CyberChef` `#VirusTotal` `#OSINT` `#IOC` `#MITREATTACK` `#IncidentResponse` `#BlueTeam` `#TryHackMe`