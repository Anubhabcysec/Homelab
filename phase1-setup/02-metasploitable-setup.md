# Setting Up Metasploitable 2 — The Vulnerable Target

## What is Metasploitable 2?

Metasploitable 2 is a Linux virtual machine built on purpose to be insecure. It's distributed by Rapid7 (the makers of Metasploit) specifically for people to practice exploitation against, legally and safely, without touching a real system. It runs Ubuntu 8.04 and ships with dozens of intentionally outdated, vulnerable services — old FTP daemons, weak Samba configurations, default credentials, and more.

---

## Setup steps

### 1. Download

Metasploitable 2 is distributed as a `.zip` containing a pre-built virtual disk (`.vmdk`), not an installer. Download it from the official SourceForge page, then extract the zip.

### 2. Reset the VMDK's internal ID before first use

VirtualBox tracks every virtual disk by a unique ID (UUID) in its own internal registry. If a `.vmdk` file is ever moved, copied, or re-downloaded, this ID can go out of sync with what VirtualBox expects, and the VM will refuse to boot with an error about the UUID not matching. It's worth fixing this before attaching the disk for the first time:

VBoxManage internalcommands sethduuid “path\to\Metasploitable.vmdk”


Run this from PowerShell (as Administrator), pointing at wherever the extracted `.vmdk` actually is.

### 3. Create the VM

- Name: Metasploitable2
- Type: Linux
- Version: Ubuntu (32-bit)
- RAM: 1024 MB
- CPUs: 1
- When asked for a disk, choose "Use an existing virtual hard disk file" and select the `.vmdk` from the extracted folder — do not let VirtualBox create a new blank disk

### 4. Boot order

Metasploitable has no installer or ISO — it boots directly off the existing disk. In the VM's Settings → System → Boot Order, make sure Hard Disk is the only thing checked, with Optical unchecked. If Optical is checked and no ISO is attached, the VM will fail to boot with a "no bootable medium" error.

### 5. Set the network

Same as Kali — Settings → Network → Adapter 1 → Attached to **NAT Network** → Name **Cyberlab**.

### 6. First boot

Default login:

Username: msfadmin
Password: msfadmin


Verify the IP:

ifconfig


### 7. Take a snapshot immediately

Once Metasploitable boots cleanly and is reachable from Kali, take a snapshot right away, before running any exploits against it. This machine is deliberately fragile, and several things can corrupt its filesystem.

---

## Problems actually hit during this setup

**VMDK UUID mismatch after taking a snapshot.** VirtualBox's internal registry and the disk file disagreed on the UUID, causing the VM to abort on startup with `does not match the value stored in the media registry`. Fixed by running `sethduuid` again, this time pointing it at the exact UUID the error message named, rather than generating a new random one.

**Filesystem corruption after an improper shutdown.** Metasploitable's filesystem is old and fragile (ext3 from 2008). Closing the VM window directly instead of shutting it down gracefully (`sudo poweroff` from inside the VM) corrupted it badly enough that `fsck` couldn't auto-repair it. The fix that actually worked was deleting the VM registration entirely (not the underlying `.vmdk`) and recreating it fresh from the same disk file.

**Filtered ports that were actually open.** Early `nmap` scans reported every port on Metasploitable as filtered, which looked like a firewall issue. The real cause was running `nmap` without `sudo` — unprivileged scans use a slower TCP connect method that behaves unreliably on this old target. Running with `sudo` and using `-sS` (SYN scan) resolved it.

---

## Result

Metasploitable 2 running with 1GB RAM, on the Cyberlab NAT Network at `10.0.2.3`, reachable from Kali, snapshotted in a clean state before any attacks were run against it.
