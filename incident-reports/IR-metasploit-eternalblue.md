# Incident Report — IR-002
## Metasploit EternalBlue Reconnaissance Scan

| Field | Details |
|-------|---------|
| **Date** | 2026-06-10 |
| **Severity** | High |
| **Status** | Resolved |
| **Analyst** | Mohamed Fadhih |
| **MITRE ATT&CK** | T1210 — Exploitation of Remote Services |
| **Tactic** | Lateral Movement |
| **Detection Tool** | Wazuh 4.7 |
| **Source** | Kali Linux VM |
| **Target** | windows-target (10.0.2.15:445) |

---

## 1. Preparation

- **Detection tool:** Wazuh SIEM with windows-target agent
- **Module used:** Metasploit auxiliary/scanner/smb/smb_ms17_010
- **Vulnerability:** MS17-010 (EternalBlue — CVE-2017-0144)
- **Lab environment:** VirtualBox soc-lab-net

---

## 2. Identification

**Alert triggered:** SMB reconnaissance scan detected against port 445

**Metasploit execution:**
```
msf > use auxiliary/scanner/smb/smb_ms17_010
msf auxiliary(scanner/smb/smb_ms17_010) > set RHOSTS 10.0.2.15
msf auxiliary(scanner/smb/smb_ms17_010) > run
[-] 10.0.2.15:445 - An SMB Login Error occurred while connecting to the IPC$ tree.
[*] 10.0.2.15:445 - Scanned 1 of 1 hosts (100% complete)
```

**Result:** SMB Login Error — host responded to SMB probe but authentication required. Windows Server 2022 is NOT vulnerable to MS17-010 (patched by default). However, the scan activity itself was logged by Wazuh as suspicious SMB reconnaissance.

**Risk Assessment:** While the target is patched, the scan technique (EternalBlue) is associated with ransomware campaigns (WannaCry, NotPetya). Any EternalBlue scan warrants immediate investigation in a production environment.

---

## 3. Containment

- Block SMB (port 445) from untrusted network segments
- Verify MS17-010 patch status on all Windows systems
- Alert SOC team of active exploitation attempt

```powershell
# Verify patch status
Get-HotFix -Id KB4012212,KB4012215,KB4012216,KB4012217
# Block SMB from external
New-NetFirewallRule -DisplayName "Block SMB External" -Direction Inbound -Protocol TCP -LocalPort 445 -RemoteAddress Internet -Action Block
```

---

## 4. Eradication

- Windows Server 2022 confirmed NOT vulnerable to MS17-010
- No exploitation occurred — scan only
- SMB port 445 restricted to management VLAN
- Metasploit session terminated — no persistence

---

## 5. Recovery

- Patch verification scan completed — system up to date
- Network monitoring enhanced for SMB exploitation patterns
- Implement SMB signing to prevent relay attacks
- Deploy honeypot SMB share to detect future reconnaissance

---

## 6. Lessons Learned

- EternalBlue scans are a reliable indicator of ransomware pre-attack reconnaissance
- Windows Server 2022 is patched by default — older servers require verification
- SMB should never be exposed to untrusted networks regardless of patch status
- Wazuh detection of SMB activity demonstrates value of network-level logging

---

## Timeline

| Time | Event |
|------|-------|
| 09:14 | Metasploit msfconsole started on Kali |
| 09:14 | Module smb_ms17_010 loaded, RHOSTS set to 10.0.2.15 |
| 09:14 | Scan executed — SMB Login Error response received |
| 09:14 | Wazuh logs SMB connection attempt from Kali IP |
| 09:15 | Analyst review — host confirmed patched, no exploitation |
| 09:20 | Incident closed — monitoring enhanced |
