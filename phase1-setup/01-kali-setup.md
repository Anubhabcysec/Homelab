# Setting Up Kali Linux — The Attacker Machine

## What is Kali Linux?

Kali is a Linux operating system built specifically for security testing. It comes with hundreds of hacking and analysis tools already installed — nmap for scanning, Metasploit for exploitation, Wireshark for traffic analysis, Hydra for password attacks, and many more. In this lab, Kali is the machine doing the attacking.

---

## Why a virtual machine?

A virtual machine (VM) is a computer running inside your actual computer, completely separate from it. VirtualBox is the free software that lets you create and run these. This matters for a hacking lab because it means Kali is fully isolated — nothing it does can touch your real operating system or your real network, as long as the networking is configured correctly.

---

## Setup steps

### 1. Create the VM

In VirtualBox:
- Click New
- Name: Kali Linux
- Type: Linux
- Version: Debian (64-bit)
- RAM: 3072 MB (3GB)
- CPUs: 2

### 2. Attach the Kali disk

Download the official Kali Linux VirtualBox image from kali.org, extract it, and when creating the VM choose "Use an existing virtual hard disk file" pointing to the extracted `.vdi` or `.vmdk`.

### 3. Set the network

This is the part that caused the most confusion during setup, so it's worth explaining properly.

VirtualBox offers several network modes. Two sound almost identical but behave completely differently:

- **NAT** — gives each VM its own private, isolated internet connection. Two VMs on plain NAT cannot see or talk to each other, even if they appear to have similar IP addresses.
- **NAT Network** — puts every VM attached to it on one shared internal network. VMs on the same NAT Network can talk to each other directly.

For this lab, every VM needs to reach every other VM, so NAT Network is required, not plain NAT.

Steps:
- In VirtualBox, go to File → Tools → Network (or Host Network Manager)
- Under the NAT Networks tab, create a network named `Cyberlab` with IPv4 prefix `10.0.2.0/24` and DHCP enabled
- On the Kali VM, go to Settings → Network → Adapter 1
- Set "Attached to" to **NAT Network**
- Set "Name" to **Cyberlab**

### 4. First boot and verification

Start Kali and log in. Confirm the network is working:

ip a


You should see an IP in the `10.0.2.x` range on `eth0`. If the Name field for NAT Network doesn't appear as a selectable option when setting up the adapter, fully close and reopen VirtualBox — this is a known interface bug where the dropdown doesn't populate until restart.

### 5. Update and verify tools

sudo apt update && sudo apt full-upgrade -y
which nmap msfconsole john hydra wireshark


All of these should already be installed on Kali by default.

---

## Problems actually hit during this setup

**Both Kali and Metasploitable showing the same IP.** This happened because both were on plain NAT, where each VM's `10.0.2.x` address is actually a separate, isolated network per VM — not a real shared network. It only looked like a collision. Switching both to NAT Network fixed it.

**NAT Network option not appearing in the adapter dropdown.** VirtualBox sometimes doesn't refresh this list live. Closing and reopening VirtualBox entirely resolved it.

---

## Result

Kali Linux running with 3GB RAM, 2 CPUs, on the Cyberlab NAT Network, reachable by every other VM in the lab, with its full toolset verified and ready.
