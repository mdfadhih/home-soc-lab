# Home SOC Lab — Wazuh + Splunk Threat Detection

![Wazuh](https://img.shields.io/badge/Wazuh-4.7-blue)
![Splunk](https://img.shields.io/badge/Splunk-9.2.1-orange)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red)
![Kali Linux](https://img.shields.io/badge/Kali-Linux-557C94)
![NIST IR](https://img.shields.io/badge/NIST-IR-green)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

## Overview

A fully functional home SOC lab built on VirtualBox with three VMs — Ubuntu (Wazuh SIEM), Windows Server 2022 (attack target), and Kali Linux (attacker). Simulated 5 real-world MITRE ATT&CK attack scenarios, detected and documented using Wazuh and Splunk.

Built to demonstrate hands-on SOC analyst skills relevant to the Australian cybersecurity market.

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                Windows 11 Host (16GB RAM)                    │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Ubuntu 22.04 │  │  Windows     │  │   Kali Linux     │  │
│  │ Wazuh 4.7    │  │  Server 2022 │  │   Attacker VM    │  │
│  │ SIEM Manager │  │  Attack      │  │                  │  │
│  │ 10.0.2.4     │  │  Target      │  │  Tools:          │  │
│  │              │  │  10.0.2.15   │  │  • nmap          │  │
│  │ RAM: 4GB     │  │  Wazuh Agent │  │  • hydra         │  │
│  │              │  │  RAM: 4GB    │  │  • metasploit    │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────────────┘  │
│         │                 │                                  │
│         └─────────────────┘                                  │
│              NAT Network: soc-lab-net                        │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Splunk Enterprise 9.2.1 (Windows 11 host)           │   │
│  │  localhost:8000 · Windows Event Log monitoring       │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## Tools & Technologies

| Category         | Tool                | Version         |
| ---------------- | ------------------- | --------------- |
| SIEM             | Wazuh               | 4.7             |
| Log Analysis     | Splunk Enterprise   | 9.2.1           |
| Hypervisor       | Oracle VirtualBox   | 7.x             |
| Attacker OS      | Kali Linux          | 2026.1          |
| Target OS        | Windows Server 2022 | Datacenter Eval |
| SIEM OS          | Ubuntu Server       | 22.04 LTS       |
| Attack Framework | MITRE ATT&CK        | v14             |
| IR Framework     | NIST SP 800-61      | Rev 2           |
| Recon Tool       | nmap                | 7.98            |
| Brute Force      | Hydra               | v9.6            |
| Exploitation     | Metasploit          | v6.4            |

---

## MITRE ATT&CK Coverage

| #   | Attack                | Technique                 | TTP ID    | Tool       | Detection                       | Severity |
| --- | --------------------- | ------------------------- | --------- | ---------- | ------------------------------- | -------- |
| 1   | Port Scan             | Network Service Discovery | T1046     | nmap       | Wazuh rule 40101                | Medium   |
| 2   | Brute Force RDP/SMB   | Brute Force               | T1110     | Hydra      | Wazuh T1110 — 112 auth failures | High     |
| 3   | EternalBlue Scan      | Exploit Public-Facing App | T1210     | Metasploit | Wazuh SMB detection             | High     |
| 4   | Suspicious PowerShell | PowerShell                | T1059.001 | PowerShell | Wazuh T1543.003 via Sysmon      | High     |
| 5   | New Admin Account     | Create Account            | T1136     | net user   | Wazuh T1484/T1098 — Level 12    | Critical |

---

## Wazuh Detection Results

**Total alerts generated: 1,195**

| Metric                      | Value                              |
| --------------------------- | ---------------------------------- |
| Total alerts                | 1,195                              |
| Level 12+ (Critical) alerts | 1                                  |
| Authentication failures     | 112                                |
| Authentication successes    | 126                                |
| Agents monitored            | 2 (windows-target + wazuh-manager) |

### Top MITRE ATT&CK Techniques Detected

- Valid Accounts (T1078)
- Account Manipulation (T1098)
- Brute Force (T1110)
- Windows Service (T1543)
- Modify Registry
- Stored Data Manipulation
- Create Account (T1136)

---

## Attack Simulations

### Attack 1 — Nmap Port Scan (T1046)

**Tool:** Kali Linux → nmap  
**Command:** `nmap -sS -p 1-1000 10.0.2.15`  
**Result:** Discovered open ports 135 (msrpc), 139 (netbios-ssn), 445 (microsoft-ds)  
**Detection:** Wazuh network scan alert

### Attack 2 — Hydra Brute Force (T1110)

**Tool:** Kali Linux → Hydra v9.6  
**Command:** `hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://10.0.2.15`  
**Result:** 14,344,399 login attempts against RDP/SMB  
**Detection:** Wazuh — 112 authentication failures, T1110 Credential Access  
**Splunk:** Failed Login Monitor dashboard showing spikes

### Attack 3 — Metasploit EternalBlue Scan (T1210)

**Tool:** Kali Linux → Metasploit v6.4  
**Module:** `auxiliary/scanner/smb/smb_ms17_010`  
**Target:** 10.0.2.15:445  
**Result:** SMB Login Error — scan completed, host enumerated  
**Detection:** Wazuh SMB reconnaissance detection

### Attack 4 — Suspicious PowerShell (T1059.001)

**Tool:** Windows Server VM → PowerShell (Admin)  
**Commands:** `-ExecutionPolicy bypass` + `-WindowStyle Hidden`  
**Detection:** Wazuh T1543.003 — New Windows Service Created via Sysmon  
**Splunk:** Process Creation Monitor dashboard spike

### Attack 5 — New Admin Account (T1136 / T1484)

**Tool:** Windows Server VM → PowerShell (Admin)  
**Commands:**

```powershell
net user hacker Password123! /add
net localgroup administrators hacker /add
```

**Detection:** Wazuh Level 12 Critical — T1484 Administrators group changed (Rule 60154)  
T1098 — User account enabled or created (Rule 60109)  
**Evidence:** user-account-creation-attack-5 screenshots

---

## Splunk Dashboards

Three custom dashboards built in Splunk Enterprise 9.2.1:

| Dashboard                | Search Query                                                                              | Visualization |
| ------------------------ | ----------------------------------------------------------------------------------------- | ------------- |
| Failed Login Monitor     | `index=* source="WinEventLog:Security" EventCode=4625 \| timechart count by Account_Name` | Line chart    |
| Process Creation Monitor | `index=* source="WinEventLog:Security" EventCode=4688 \| timechart count by ComputerName` | Column chart  |
| Security Events Overview | `index=* source="WinEventLog:Security" \| timechart count by EventCode`                   | Pie chart     |

---

## Incident Reports

Three full NIST SP 800-61 incident reports in `/incident-reports`:

| Report | Incident                            | Severity | MITRE TTP    |
| ------ | ----------------------------------- | -------- | ------------ |
| IR-001 | Nmap Port Scan + Brute Force        | High     | T1046, T1110 |
| IR-002 | Metasploit EternalBlue Scan         | High     | T1210        |
| IR-003 | Unauthorised Admin Account Creation | Critical | T1136, T1484 |

---

## Screenshots Evidence

| File                                             | Description                                         |
| ------------------------------------------------ | --------------------------------------------------- |
| zero_Agent_Wazuh.png                             | Wazuh fresh install — 0 agents                      |
| windows-agent-active.png                         | Wazuh showing 1 active agent (windows-target)       |
| npm-port-attack-1_1.png                          | Kali nmap scan discovering open ports on 10.0.2.15  |
| npm-port-attack-1_2.png                          | Wazuh detecting nmap scan                           |
| npm-port-attack-1_1_3-kali.png                   | Nmap results — ports 135, 139, 445 open             |
| hydra-running-kali-brute-force-attack-2_1_1.png  | Hydra brute force running from Kali                 |
| hydra-running-wazuh-brute-force-attack-2_1_2.png | Wazuh — 1,102 alerts, 112 auth failures             |
| hydra-running-wazuh-brute-force-attack-2_1_3.png | Wazuh alert table — T1078 T1531 logon failures      |
| splunk-running-attack-2_1_4.png                  | Splunk Failed Login Monitor showing spikes          |
| msf-attack-3_1_1.png                             | Metasploit EternalBlue scanner running              |
| process-attack-4_1_1.png                         | PowerShell bypass execution running                 |
| suspicious-powershell-attack-4_1_1-.png          | Wazuh T1543.003 PowerShell detection                |
| user-account-creation-attack-5_1_1.png           | net user hacker /add successful                     |
| user-account-creation-attack-5_1_2.png           | Wazuh T1484 Administrators group changed — Level 12 |
| user-account-creation-attack-5_1_3.png           | Wazuh 1,195 total alerts dashboard                  |
| Dashboard_created.png                            | Splunk — 3 dashboards created                       |
| failed-login-monitor-splunk.png                  | Splunk Failed Login Monitor with data               |
| Process-creation-monitor-splunk.png              | Splunk Process Creation Monitor with data           |
| security-events-overview-splunk.png              | Splunk Security Events Overview pie chart           |
| Splunk_event-Log-arriving.png                    | Splunk WinEventLog:Security events flowing          |

---

## Key Findings

- **1,195 Wazuh alerts** generated across 5 attack simulations
- **Level 12 Critical alert** triggered by new admin account creation (T1484)
- **112 authentication failures** detected — Hydra brute force fully captured
- **Sysmon required** for PowerShell command line visibility in Wazuh
- **Two SIEM tools** (Wazuh + Splunk) demonstrated different collection methods
- **Wazuh agent** deployed via MSI on Windows Server Core (no GUI)
- **LogonType 3** used by Windows Server Core for network authentication

---

## Lab Setup

### Prerequisites

- Windows 11 host with 16GB RAM
- Oracle VirtualBox 7.x + Extension Pack
- Ubuntu Server 22.04 ISO
- Windows Server 2022 Evaluation ISO
- Kali Linux VirtualBox OVA

### VM Specifications

| VM             | OS                  | RAM | Role          |
| -------------- | ------------------- | --- | ------------- |
| wazuh-manager  | Ubuntu 22.04        | 4GB | Wazuh SIEM    |
| windows-target | Windows Server 2022 | 4GB | Attack target |
| kali           | Kali Linux 2026.1   | 2GB | Attacker      |

### Wazuh Install Command

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash wazuh-install.sh -a
```

---

## About

**Mohamed Fadhih** — Cybersecurity Analyst | Melbourne, Australia

- Master of Cybersecurity — Monash University (Group of Eight)
- AWS Certified Cloud Practitioner
- Google Cybersecurity Professional Certificate
- SOC Operations Intern — Digillium (500+ alerts triaged)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/fadhih)
[![Portfolio](https://img.shields.io/badge/Portfolio-Visit-green)](http://my-portfolio-seven-tawny-79.vercel.app/)
[![Project 1](https://img.shields.io/badge/Project_1-Azure_Sentinel_Lab-blue)](https://github.com/mdfadhih/soc-lab-threat-detection)
