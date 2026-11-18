# Phase 2 — Attacks

## What this phase covers

This is the offensive half of the lab. Using Kali Linux as the attacker machine, I ran real exploitation techniques against Metasploitable 2 — the same categories of attack used in actual penetration tests. Each attack below has its own folder with a full writeup and screenshots.

The point was never just to get a shell and stop. For every attack I tried to understand why the vulnerability existed, how a defender would notice it, and what should have been done to prevent it in the first place.

---

## Target

- Attacker: Kali Linux, `10.0.2.15`
- Target: Metasploitable 2, `10.0.2.3`
- Network: isolated VirtualBox NAT Network ("Cyberlab", `10.0.2.0/24`)

---

## Attacks

### 01 — Network Reconnaissance
**Folder:** `01-nmap-recon/`

Scanned the target with nmap before touching anything else, to see what ports and services were actually running. This is always step one — you can't exploit what you haven't found.

---

### 02 — Samba Exploitation
**Folder:** `02-samba-exploit/`

Exploited CVE-2007-2447, a flaw in Samba 3.0.20 where the username field during login gets passed unsanitized to the system shell. Sending a crafted username executed arbitrary commands as root — no password needed. Got full root access on the target.

---

### 03 — FTP Brute Force
**Folder:** `03-hydra-bruteforce/`

Used Hydra to automatically try passwords against the FTP service. The target had no account lockout policy and was still using its default credentials (`msfadmin:msfadmin`), so it was cracked in just four attempts.

---

### 04 — Post-Exploitation
**Folder:** `04-post-exploitation/`

Everything done after getting a root shell on the target: checking what the system actually looks like from the inside, dumping password hashes from `/etc/shadow`, creating a backdoor user to stay in even if the original vulnerability gets patched, checking the ARP cache to see what other machines the box had talked to, and finally clearing local logs to simulate covering tracks — then noting why that doesn't actually work against a SIEM that's already shipped the logs elsewhere.

---

### 05 — vsftpd 2.3.4 Backdoor
**Folder:** `05-vsftpd-exploit/`

A different kind of vulnerability than the others. This wasn't a coding mistake — someone broke into the vsftpd project's own download server in 2011 and planted a deliberate backdoor in the source code. Any server running that exact version opens a root shell the moment it sees a username ending in `:)`. Same result as the Samba exploit (instant root, no privilege escalation needed), but the cause is a supply chain compromise rather than a bug.

---

### 06 — Denial of Service
**Folder:** `06-dos-attack/`

Ran a DoS attempt against the target using hping3, flooding it with traffic and watching the effect in Wireshark. One useful mistake here — I set a spoofed source IP (SHOST) for the attack, which meant Wazuh's agent on Kali didn't log anything meaningful, since it only sees Kali's own local activity, not what IP address outbound packets claim to be from. That's a real detection gap worth understanding: host-based agents don't see spoofed traffic the way network-level tools like Wireshark or Suricata do.

---

## Key findings across all attacks

| Vulnerability | Severity | Root cause |
|---|---|---|
| CVE-2007-2447 (Samba) | Critical | Unpatched, 19-year-old software |
| Default FTP credentials | High | No password policy enforced |
| No account lockout | High | Missing basic configuration |
| CVE-2011-2523 (vsftpd) | Critical | Supply chain compromise, not a bug |
| Spoofable DoS traffic | Medium | Host-based monitoring alone can't see it |

## Biggest takeaway

Every exploit here except the DoS attempt had a known, published fix years before this lab was built. The Samba patch has existed since 2007. Default credentials and missing lockout policies are a configuration decision, not a zero-day. Most real-world breaches aren't sophisticated — they're exactly this: old software and weak defaults that nobody got around to fixing.
