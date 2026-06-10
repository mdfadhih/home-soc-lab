# Incident Report — IR-001
## Nmap Port Scan + Hydra Brute Force Attack

| Field | Details |
|-------|---------|
| **Date** | 2026-06-09 |
| **Severity** | High |
| **Status** | Resolved |
| **Analyst** | Mohamed Fadhih |
| **MITRE ATT&CK** | T1046 — Network Service Discovery, T1110 — Brute Force |
| **Tactic** | Reconnaissance, Credential Access |
| **Detection Tool** | Wazuh 4.7 |
| **Source IP** | Kali Linux VM (soc-lab-net) |
| **Target** | windows-target (10.0.2.15) |

---

## 1. Preparation

- **Detection tool:** Wazuh SIEM with windows-target agent
- **Log source:** Windows Security Event Log via Wazuh agent
- **Lab environment:** VirtualBox NAT network (soc-lab-net)
- **Attacker:** Kali Linux 2026.1

---

## 2. Identification

**Phase 1 — Reconnaissance:**
Nmap SYN scan detected against windows-target. Open ports discovered:
- Port 135/tcp — msrpc
- Port 139/tcp — netbios-ssn
- Port 445/tcp — microsoft-ds (SMB)

**Phase 2 — Brute Force:**
Hydra v9.6 launched credential stuffing attack using rockyou.txt wordlist:
- Target: rdp://10.0.2.15:3389 → SMB://10.0.2.15:445
- Attempts: 14,344,399 login combinations
- Wazuh detected 112 authentication failures
- MITRE techniques: T1078 (Valid Accounts), T1531 (Account Access Removal)

**Wazuh Alert Evidence:**
- Rule 60122: Logon failure — Unknown user or bad password (Level 5)
- Rule 60204: Multiple Windows logon failures (Level 10)
- T1110 Credential Access — detected

---

## 3. Containment

**Immediate actions:**
- Block source IP at Windows Firewall level
- Enable account lockout policy (5 attempts, 30-minute lockout)
- Disable RDP exposure — restrict to management network only
- Review NSG/firewall rules to close unnecessary ports

```powershell
# Block attacker IP
New-NetFirewallRule -DisplayName "Block Attacker" -Direction Inbound -RemoteAddress 10.0.2.14 -Action Block
# Enable account lockout
net accounts /lockoutthreshold:5 /lockoutduration:30
```

---

## 4. Eradication

- No successful authentication confirmed
- No malware or persistence mechanisms installed
- All failed login attempts logged and attributed to Kali VM
- Port 445 (SMB) restricted to internal management access only

---

## 5. Recovery

- Monitor for repeat scan activity from same subnet
- Implement network segmentation between attacker and target subnets
- Enable Windows Defender Firewall with enhanced logging
- Deploy honeypot credentials to detect future credential stuffing

---

## 6. Lessons Learned

- Wazuh successfully detected both reconnaissance and brute force phases
- RDP should never be exposed without JIT access or VPN requirement
- Account lockout policies are critical — 14 million attempts would eventually succeed without them
- Network segmentation between attacker/target VMs should be enforced in production

---

## Timeline

| Time | Event |
|------|-------|
| 21:49 | Kali nmap scan initiated against 10.0.2.15 |
| 21:49 | Ports 135, 139, 445 discovered open |
| 18:30 | Hydra brute force started (RDP then SMB) |
| 18:30 | Wazuh starts logging authentication failures |
| 18:38 | 112 auth failures accumulated — T1110 alert triggered |
| 18:42 | Attack ended — session file saved |
| 18:45 | Analyst review — no successful auth confirmed |
