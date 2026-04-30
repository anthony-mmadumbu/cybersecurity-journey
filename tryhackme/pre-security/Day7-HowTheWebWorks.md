🛡️ DNS in Detail – TryHackMe


Module 3 · How the Web Works
📅 DATE
31 March 2026 · Day 7


📚 WHAT I LEARNED

DNS translates human-readable domain names into IP addresses that computers use to communicate

Different record types serve different purposes — A records for IPv4, AAAA for IPv6, MX for 
mail, CNAME for aliases

DNS queries go through a hierarchy: recursive resolver → root server → TLD server → authoritative nameserver

TTL (Time to Live) controls how long a DNS record is cached before being refreshed
DNS operates over UDP port 53 by default, making it fast but also a target for attacks like DNS spoofing


🛠️ WHAT I DID

Completed the DNS in Detail room in full

Answered all in-room questions on DNS record types and query resolution

Traced how a domain name resolves step by step through the DNS hierarchy


🔐 WHY IT MATTERS

DNS is a common attack surface — DNS spoofing and cache poisoning can redirect users to malicious sites

Understanding DNS is essential for investigating phishing, domain hijacking, and network reconnaissance


❓ ONE THING I DIDN'T FULLY UNDERSTAND

Everything was clear — Security+ prep paid off! 🎉
