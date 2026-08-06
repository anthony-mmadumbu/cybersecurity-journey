# 🛡️ Title
Network Traffic Basics – TryHackMe (SOC Level 1 · Network Security and Traffic Analysis Module)

# 📅 Date
6 August 2026 (Day 27)

# 📚 What I Learned

* **Network Traffic Analysis (NTA) is not a synonym for Wireshark** — it's the combination of log correlation, deep packet inspection and network flow statistics, aimed at complete visibility of what is communicated inside and outside the network
* The analyst's core job is knowing what normal looks like, so deviation from the baseline stands out
* **Endpoints generate the most traffic** in a network — which is why endpoint-adjacent visibility matters
* **Kerberos is contacted before an SMB session can be established** — authentication precedes file sharing on a Windows domain, so a KDC/domain controller sees authentication activity for the whole estate
* **TLS = Transport Layer Security** — and encrypted traffic changes what a tap can actually reveal
* That **where you place a tap determines what you can see**: a tap only ever captures traffic that physically passes through that point

# 🛠️ What I Did

* Worked through the NTA theory: what it is, why it's needed, what can be observed and how
* Completed the interactive tap-placement exercise across two scenarios
* **Scenario 1 (Malicious PS Download):** placed the tap on the **Web Proxy** to capture a phishing-initiated HTTP request downloading a malicious PowerShell file, then inspected the packets to find the HTTP response containing the script — flag `THM{FoundTheMalware}`
* **Scenario 2 (DNS Infiltration):** placed the tap on the **DNS server** to capture C2 instructions smuggled through DNS TXT records — flag `THM{C2CommandFound}`
* Full write-up in the portfolio piece

# 🔐 Why It Matters

* Tap placement is a prerequisite question, not a detail. If the sensor is in the wrong place, the traffic simply never reaches it and no amount of analysis skill recovers it
* Choke points (proxy, DNS server, domain controller) concentrate traffic — one well-placed tap can cover an entire category of activity across every host
* Both scenarios show attacker traffic hiding inside protocols that are ordinary and expected: HTTP downloads and DNS lookups. Detection depends on inspecting the content, not just observing that the protocol is in use

# ❓ One Thing I Didn't Fully Understand

* How much genuine visibility a proxy tap gives once traffic is HTTPS rather than HTTP. The room taught TLS as a concept but the exercise used plaintext HTTP, so I want to understand what a proxy actually sees on encrypted traffic — metadata and SNI only, or whether TLS interception changes the picture

📌 Full exercise write-up: `Day27_Network_Traffic_Basics_Portfolio.md`