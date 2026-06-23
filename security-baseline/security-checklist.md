# Security Baseline Checklist

Reference: CIS Microsoft Windows Server 2022 Benchmark v1.0 / DISA STIG WN22

Mark each item ✅ Compliant | ⚠️ Partial | ❌ Not Compliant | N/A

---

## 1. Account Policies

| # | Control | Setting | Status |
|---|---|---|---|
| 1.1 | Minimum password length | ≥ 12 characters | ✅ |
| 1.2 | Password complexity | Enabled | ✅ |
| 1.3 | Maximum password age | ≤ 90 days | ✅ |
| 1.4 | Enforce password history | ≥ 24 passwords | ✅ |
| 1.5 | Account lockout threshold | ≤ 5 attempts | ✅ |
| 1.6 | Account lockout duration | ≥ 30 minutes | ✅ |
| 1.7 | Guest account disabled | Disabled | ✅ |
| 1.8 | Built-in Administrator account renamed | Renamed to non-obvious name | ✅ |

---

## 2. Local Policies — User Rights Assignment

| # | Control | Status |
|---|---|---|
| 2.1 | Access this computer from the network — only Administrators, Authenticated Users | ✅ |
| 2.2 | Deny access to this computer from network — Guests, Anonymous | ✅ |
| 2.3 | Allow log on locally — Administrators only (on DC) | ✅ |
| 2.4 | Deny log on locally — Guests | ✅ |
| 2.5 | Take ownership of files — Administrators only | ✅ |
| 2.6 | Manage auditing and security log — Administrators only | ✅ |
| 2.7 | Shut down the system — Administrators, Authenticated Users | ✅ |

---

## 3. Security Options

| # | Control | Setting | Status |
|---|---|---|---|
| 3.1 | Interactive logon: Do not display last username | Enabled | ✅ |
| 3.2 | Interactive logon: Do not require CTRL+ALT+DEL | Disabled (require it) | ✅ |
| 3.3 | Interactive logon: Machine inactivity limit | 900 seconds | ✅ |
| 3.4 | Interactive logon: Legal notice text | Configured | ✅ |
| 3.5 | Network security: LAN Manager authentication | NTLMv2 only, refuse LM/NTLM | ✅ |
| 3.6 | Network security: NTLM SSP minimum session | NTLMv2 + 128-bit encryption | ✅ |
| 3.7 | Network access: Anonymous SID/name translation | Disabled | ✅ |
| 3.8 | Network access: Do not allow anonymous enumeration of SAM accounts | Enabled | ✅ |
| 3.9 | Accounts: Limit local account use of blank passwords | Enabled | ✅ |
| 3.10 | Shutdown: Allow system to be shut down without logon | Disabled | ✅ |
| 3.11 | Recovery console: Allow floppy copy and access to all drives | Disabled | ✅ |
| 3.12 | System cryptography: Force strong key protection | Enabled | ✅ |

---

## 4. Advanced Audit Policy

| # | Category | Subcategory | Setting | Status |
|---|---|---|---|---|
| 4.1 | Account Logon | Credential Validation | Success + Failure | ✅ |
| 4.2 | Account Logon | Kerberos Authentication Service | Success + Failure | ✅ |
| 4.3 | Account Management | Computer Account Management | Success | ✅ |
| 4.4 | Account Management | Security Group Management | Success | ✅ |
| 4.5 | Account Management | User Account Management | Success + Failure | ✅ |
| 4.6 | Logon/Logoff | Logon | Success + Failure | ✅ |
| 4.7 | Logon/Logoff | Logoff | Success | ✅ |
| 4.8 | Object Access | File System | Failure | ✅ |
| 4.9 | Policy Change | Audit Policy Change | Success | ✅ |
| 4.10 | Privilege Use | Sensitive Privilege Use | Failure | ✅ |
| 4.11 | System | Security System Extension | Success | ✅ |
| 4.12 | System | System Integrity | Success + Failure | ✅ |

---

## 5. Windows Firewall

| # | Control | Status |
|---|---|---|
| 5.1 | Domain profile: Firewall state ON | ✅ |
| 5.2 | Private profile: Firewall state ON | ✅ |
| 5.3 | Public profile: Firewall state ON | ✅ |
| 5.4 | Inbound connections: Block (unless rule allows) | ✅ |
| 5.5 | Outbound connections: Allow (unless rule blocks) | ✅ |
| 5.6 | Log dropped packets: Enabled (all profiles) | ✅ |
| 5.7 | Log log file size: ≥ 16,384 KB | ✅ |

---

## 6. Windows Defender / Antivirus

| # | Control | Status |
|---|---|---|
| 6.1 | Microsoft Defender Antivirus enabled | ✅ |
| 6.2 | Real-time protection enabled | ✅ |
| 6.3 | Definition updates: Current (< 24 hours) | ✅ |
| 6.4 | Scheduled full scan: Weekly | ✅ |
| 6.5 | Cloud-delivered protection: Enabled | ✅ |
| 6.6 | Defender SmartScreen: Enabled | ✅ |

---

## 7. Remote Access & Services

| # | Control | Status |
|---|---|---|
| 7.1 | Remote Desktop: NLA required | ✅ |
| 7.2 | Remote Desktop: Highest encryption level | ✅ |
| 7.3 | WinRM: Disabled if not needed | ✅ |
| 7.4 | Telnet client: Not installed | ✅ |
| 7.5 | FTP server: Not installed | ✅ |
| 7.6 | Remote Registry: Disabled | ✅ |
| 7.7 | Print Spooler (on DC): Disabled | ✅ |

---

## 8. Active Directory Specific

| # | Control | Status |
|---|---|---|
| 8.1 | AD Recycle Bin enabled | ✅ |
| 8.2 | LDAP signing required | ✅ |
| 8.3 | LDAP channel binding | Enabled | ✅ |
| 8.4 | SMB signing required (DC) | ✅ |
| 8.5 | SMB v1 disabled | ✅ |
| 8.6 | SYSVOL/NETLOGON using DFS-R replication | ✅ |
| 8.7 | Protected Users group used for privileged accounts | ✅ |
| 8.8 | Admin accounts not used for email/web browsing | ✅ |
| 8.9 | Service accounts use gMSA where possible | ⚠️ Partial |
| 8.10 | Fine-grained password policy for admins | ✅ |

---

## 9. Patch & Vulnerability Management

| # | Control | Status |
|---|---|---|
| 9.1 | OS patches current (≤ 30 days) | ✅ |
| 9.2 | WSUS or equivalent deployed | ✅ |
| 9.3 | Patch compliance reports reviewed monthly | ✅ |
| 9.4 | Vulnerability scan performed quarterly | ⚠️ Lab — manual review |

---

## 10. Logging & Monitoring

| # | Control | Status |
|---|---|---|
| 10.1 | Security log size ≥ 196,608 KB | ✅ |
| 10.2 | Application log size ≥ 32,768 KB | ✅ |
| 10.3 | System log size ≥ 32,768 KB | ✅ |
| 10.4 | Logs retained ≥ 90 days | ✅ |
| 10.5 | Event log: do not overwrite (archive) | ✅ |

---

## Verification Commands

```powershell
# Check SMB v1 status (should be False)
Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

# Check LDAP signing requirement
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\NTDS\Parameters" `
    -Name "LDAPServerIntegrity"
# 2 = Require signing ✅

# Check NTLMv2 enforcement
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "LmCompatibilityLevel"
# 5 = NTLMv2 only ✅

# Verify Print Spooler is stopped on DC
Get-Service -Name Spooler | Select-Object Name,Status,StartType

# Check Defender status
Get-MpComputerStatus | Select-Object AntivirusEnabled,RealTimeProtectionEnabled,`
    AntivirusSignatureAge,FullScanAge
```
