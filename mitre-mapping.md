# MITRE ATT&CK Coverage Map — Home SOC Lab

## Tactics Covered

### Reconnaissance
| Technique | ID | Tool | Detection | Evidence |
|-----------|-----|------|-----------|---------|
| Network Service Discovery | T1046 | nmap | Wazuh port scan rule | npm-port-attack-1 screenshots |

### Credential Access
| Technique | ID | Tool | Detection | Evidence |
|-----------|-----|------|-----------|---------|
| Brute Force | T1110 | Hydra | Wazuh 112 auth failures, Rule 60204 | hydra-running screenshots |

### Lateral Movement
| Technique | ID | Tool | Detection | Evidence |
|-----------|-----|------|-----------|---------|
| Exploit Public-Facing Application | T1210 | Metasploit ms17_010 | Wazuh SMB detection | msf-attack-3 screenshot |

### Execution
| Technique | ID | Tool | Detection | Evidence |
|-----------|-----|------|-----------|---------|
| PowerShell | T1059.001 | PowerShell -bypass -hidden | Wazuh T1543.003 via Sysmon | process-attack-4 screenshots |

### Persistence / Privilege Escalation
| Technique | ID | Tool | Detection | Evidence |
|-----------|-----|------|-----------|---------|
| Create Local Account | T1136.001 | net user /add | Wazuh Rule 60109 Level 8 | user-account-creation screenshots |
| Group Policy Modification | T1484 | net localgroup administrators | Wazuh Rule 60154 Level 12 CRITICAL | user-account-creation screenshots |
| Account Manipulation | T1098 | net user /add | Wazuh Rule 60110 Level 8 | user-account-creation screenshots |

---

## Windows Event ID Reference

| EventID | Description | Wazuh Rule | Used In |
|---------|-------------|------------|---------|
| 4624 | Successful logon | 60106 | All attacks |
| 4625 | Failed logon | 60122 | Attack 2 — Brute force |
| 4688 | Process creation | Sysmon | Attack 4 — PowerShell |
| 4720 | User account created | 60109 | Attack 5 — Account creation |
| 4728 | User added to global group | 60160 | Attack 5 |
| 4732 | User added to local group | 60154 | Attack 5 — CRITICAL |

---

## Wazuh Alert Level Reference

| Level | Severity | Description | Example |
|-------|---------|-------------|---------|
| 3 | Low | Informational | Logon success |
| 5 | Medium | Notable activity | System time changed |
| 8 | High | Security concern | User account created |
| 10 | High | Threat detected | Multiple logon failures |
| 12 | Critical | Immediate response | Administrators group changed |

---

## Total Coverage Summary

- **5 MITRE ATT&CK Techniques** detected
- **7 Windows EventIDs** monitored
- **1,195 total alerts** generated
- **Level 12 Critical** alert achieved (Attack 5)
- **2 SIEM tools** demonstrating parallel detection
