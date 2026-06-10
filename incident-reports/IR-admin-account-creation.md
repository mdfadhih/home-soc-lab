# Incident Report — IR-003
## Unauthorised Administrator Account Creation

| Field | Details |
|-------|---------|
| **Date** | 2026-06-10 |
| **Severity** | Critical |
| **Status** | Resolved |
| **Analyst** | Mohamed Fadhih |
| **MITRE ATT&CK** | T1136.001 — Local Account, T1484 — Group Policy Modification, T1098 — Account Manipulation |
| **Tactic** | Persistence, Privilege Escalation, Defense Evasion |
| **Detection Tool** | Wazuh 4.7 |
| **Wazuh Alert Level** | 12 (Critical) |
| **Source** | windows-target (local admin session) |

---

## 1. Preparation

- **Detection tool:** Wazuh SIEM — windows-target agent
- **Log sources:** Windows Security Event Log (EventID 4720, 4732, 4728)
- **Wazuh rules triggered:** 60154 (Level 12), 60109 (Level 8), 60110 (Level 8)

---

## 2. Identification

**Alert triggered:** Wazuh Level 12 Critical — Administrators group changed

**Commands executed on windows-target:**
```powershell
net user hacker Password123! /add
net localgroup administrators hacker /add
```

**Wazuh detections:**
| Time | Rule ID | Level | Description | MITRE |
|------|---------|-------|-------------|-------|
| 09:45:27 | 60154 | 12 | Administrators group changed | T1484 |
| 09:45:15 | 60170 | 5 | Users group changed | T1484 |
| 09:45:15 | 60110 | 8 | User account changed | T1098 |
| 09:45:15 | 60109 | 8 | User account enabled or created | T1098 |
| 09:45:15 | 60160 | 5 | Domain users group changed | T1484 |

**Scope assessment:**
- New account "hacker" created with admin privileges
- Account added to local Administrators group
- No lateral movement or data exfiltration detected
- Attack confined to single workstation

---

## 3. Containment

**Immediate actions (executed within 5 minutes of detection):**

```powershell
# Disable the malicious account immediately
net user hacker /active:no

# Remove from administrators group
net localgroup administrators hacker /delete

# Delete the account
net user hacker /delete

# Audit all recent account changes
Get-EventLog -LogName Security -InstanceId 4720,4732 -Newest 20
```

**Escalation decision:** In a production environment, this would be P1 — immediate escalation to Tier 2 and security management. Evidence preservation would be required before remediation.

---

## 4. Eradication

- Malicious account "hacker" deleted — confirmed removed
- Administrators group membership audited — no other suspicious accounts
- Registry run keys checked — no persistence mechanisms
- Scheduled tasks reviewed — no new suspicious tasks
- Active sessions terminated

---

## 5. Recovery

- Full audit of all local administrator accounts completed
- Password policy reviewed and hardened
- Account creation alerting threshold lowered to Level 10
- Privileged Account Management (PAM) solution recommended
- Just-In-Time admin access recommended for production

**Recommended hardening:**
```powershell
# Restrict who can create local accounts
secedit /export /cfg C:\security_audit.cfg
# Enable enhanced audit logging
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
```

---

## 6. Lessons Learned

- Wazuh Level 12 Critical alert fired within seconds of account creation — excellent detection speed
- Account creation (T1136) is a high-confidence indicator of compromise in production environments
- Privileged Identity Management (PIM) would prevent this attack entirely
- The attack chain (brute force → account creation) represents a realistic ransomware pre-deployment pattern
- Wazuh detected all 5 EventIDs associated with this attack automatically

---

## Timeline

| Time | Event |
|------|-------|
| 09:44 | Attacker gains admin shell on windows-target |
| 09:45:15 | `net user hacker Password123! /add` executed |
| 09:45:15 | Wazuh fires Rule 60109 — User account created (Level 8) |
| 09:45:15 | Wazuh fires Rule 60110 — User account changed (Level 8) |
| 09:45:27 | `net localgroup administrators hacker /add` executed |
| 09:45:27 | Wazuh fires Rule 60154 — **CRITICAL Level 12** — Administrators group changed |
| 09:45:30 | SOC analyst receives critical alert |
| 09:46 | Containment initiated — account disabled |
| 09:47 | Account deleted — eradication complete |
| 09:50 | Full audit complete — no other persistence found |
| 10:00 | Incident closed — hardening recommendations issued |
