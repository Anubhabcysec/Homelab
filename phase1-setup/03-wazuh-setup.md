# Setting Up Wazuh SIEM — The Monitoring Server

## What is a SIEM, and what is Wazuh?

SIEM stands for Security Information and Event Management. It's software that collects logs from multiple machines in one place, applies detection rules against them, and raises alerts when something suspicious happens — failed logins, privilege escalation, unusual outbound connections, and so on. This is the core tool a SOC (Security Operations Center) analyst works in every day.

Wazuh is a free, open-source SIEM. In this lab, it plays the role of the defender — watching what happens on Kali in real time while attacks are carried out against Metasploitable.

---

## Setup steps

### 1. Download the OVA

Wazuh distributes a ready-to-import virtual appliance (an `.ova` file) from their official site. This is the simplest way to get a fully configured Wazuh server running, without manually installing its several components by hand.

### 2. Import it

In VirtualBox: File → Import Appliance → select the downloaded `.ova`.

During import, increase the defaults to:
- RAM: at least 4096 MB (6GB recommended — Wazuh runs several services internally and is prone to timing out on startup if underpowered)
- CPUs: 2 or more

### 3. Set up two network adapters

Wazuh needs two separate network connections for two different purposes, which is different from Kali and Metasploitable:

- **Adapter 1 — NAT Network, Cyberlab.** This is how Wazuh talks to Kali and Metasploitable, and how it receives agent logs.
- **Adapter 2 — Host-only Adapter.** This gives Wazuh a second IP that the Windows host machine itself can reach. Without this, the Wazuh web dashboard is only accessible from inside the `10.0.2.0/24` lab network, which the host's own browser isn't on.

### 4. First boot

The boot screen displays default login credentials directly:

Username: wazuh-user
Password: wazuh


Log in and check both network interfaces:

ip a


You should see two addresses — one in the `10.0.2.x` range (Cyberlab) and one in the `192.168.56.x` range (host-only).

### 5. Open the dashboard

From the Windows host browser, go to:

https://192.168.56.x


using whatever address the host-only adapter actually got. The browser will show a certificate warning because Wazuh uses a self-signed certificate — this is expected for a local lab and safe to click through.

Dashboard login:

Username: admin
Password: admin


### 6. Connect Kali as a monitored agent

From the dashboard: Agents management → Summary → Deploy new agent. Choose Linux, DEB amd64 (Kali is Debian-based), enter the Wazuh server's Cyberlab IP (`10.0.2.4`) as the server address, and give it a name like `kali-attacker`. This generates an install command to run on the Kali terminal.

After running it on Kali:

sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent


The agent should appear as Active in the dashboard within a minute.

---

## Problems actually hit during this setup

**Dashboard unreachable from the host browser.** The dashboard only listened on the internal `10.0.2.x` Cyberlab network, which the Windows host isn't part of. Solved by adding the second, host-only network adapter described above.

**API Connections showing Offline.** This happened whenever the Wazuh VM was restarted — the internal manager, indexer, and dashboard services sometimes don't all come up cleanly together. Fixed by manually restarting each in order:

`sudo systemctl restart wazuh-manager`

`sudo systemctl restart wazuh-indexer`

`sudo systemctl restart wazuh-dashboard`


**wazuh-manager failing with a timeout.** Checked with `sudo systemctl status wazuh-manager`, which showed `failed (Result: timeout)`. The cause was the VM being RAM-starved (`free -h` showed under 1GB free) with all three lab VMs running at once. Fixed by increasing Wazuh's allocated RAM from the original 5GB to 6GB.

**Kali agent going inactive after a Wazuh restart.** The agent service on Kali needs a manual restart any time the Wazuh server itself restarts:

sudo systemctl restart wazuh-agent


---

## Result

Wazuh v4.14.5 running with 6GB RAM, dual-homed on both the Cyberlab network and a host-only adapter, dashboard reachable from the Windows host, actively monitoring Kali as a connected agent named `kali-attacker`.
