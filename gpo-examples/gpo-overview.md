# Group Policy Object (GPO) Examples

## GPO Strategy

GPOs are linked at the OU level, not the domain level (except for baseline policies). This allows targeted application and avoids unintended scope. All GPOs follow a naming convention: `[Scope] - [Function] - [Version]`.

---

## GPO Inventory

| GPO Name | Linked OU | Scope | Purpose |
|---|---|---|---|
| `Default Domain Policy` | Domain root | All | Password/account lockout policy (only use for these) |
| `CORP - Security Baseline - v1` | Domain root | All computers | Core security hardening |
| `CORP - IT Workstation Policy - v1` | Computers\Desktops\IT | Computers | IT-specific settings |
| `CORP - User Workstation Policy - v1` | Computers\Desktops\Users | Computers | Standard user workstation |
| `CORP - IT User Policy - v1` | Users\IT Department | Users | IT staff user settings |
| `CORP - HR User Policy - v1` | Users\HR Department | Users | HR folder redirection, drive maps |
| `CORP - WSUS Policy - v1` | Domain root | All computers | Windows Update targeting |
| `CORP - Admin Restrictions - v1` | _Admin | Users | Restrict admin accounts from web/email |

---

## 1. Password & Account Lockout Policy

*Linked to domain root via Default Domain Policy.*

| Setting | Value |
|---|---|
| Minimum password length | 12 characters |
| Password complexity | Enabled |
| Maximum password age | 90 days |
| Minimum password age | 1 day |
| Password history | 24 passwords |
| Account lockout threshold | 5 invalid attempts |
| Lockout duration | 30 minutes |
| Reset lockout counter after | 30 minutes |

```
GPO Path: Computer Configuration → Policies → Windows Settings →
          Security Settings → Account Policies
```

---

## 2. Security Baseline GPO

*Linked to domain root, applies to all computers.*

**Key Settings:**

```
Computer Configuration → Policies → Windows Settings → Security Settings

Interactive Logon:
  - Do not display last user name: Enabled
  - Do not require CTRL+ALT+DEL: Disabled
  - Machine inactivity limit: 900 seconds (15 min)

Network Security:
  - LAN Manager authentication level: Send NTLMv2 only, refuse LM & NTLM
  - Minimum session security (NTLM SSP): Require NTLMv2 + 128-bit encryption

User Rights Assignment:
  - Deny log on locally: Guests
  - Deny access to this computer from the network: Guests, Anonymous

Audit Policy (via Advanced Audit):
  - Logon/Logoff: Success + Failure
  - Account Logon: Success + Failure
  - Account Management: Success
  - Policy Change: Success
  - Privilege Use: Failure
  - Object Access: Failure
```

---

## 3. WSUS Update Policy GPO

*Linked to domain root, applies to all computers.*

```
Computer Configuration → Policies → Administrative Templates →
Windows Components → Windows Update

Configure Automatic Updates:
  - Setting: Enabled
  - Configure automatic updating: 4 - Auto download and schedule the install
  - Schedule install day: 0 - Every day
  - Schedule install time: 03:00

Specify intranet Microsoft update service location:
  - Setting: Enabled
  - Set the intranet update service: http://DC01:8530
  - Set the intranet statistics server: http://DC01:8530

No auto-restart with logged on users: Enabled
Re-prompt for restart: 10 minutes
```

---

## 4. User Workstation Policy GPO

*Linked to Computers\Desktops\Users.*

```
Computer Configuration:
  - Disable USB storage: Enabled
    (Administrative Templates → System → Removable Storage Access →
     All Removable Storage classes: Deny all access)
  - Disable AutoRun: Enabled
  - Enable Windows Firewall (all profiles): Enabled
  - Disable Remote Registry: Enabled

User Configuration:
  - Remove Run from Start Menu: Enabled
  - Remove Command Prompt: Enabled
  - Prevent access to Registry editing tools: Enabled
  - Disable Control Panel access: Disabled (allow for IT OU, block here)
```

---

## 5. Drive Mapping GPO (HR Department)

*Linked to Users\HR Department.*

```powershell
# GPO: CORP - HR User Policy - v1
# User Configuration → Preferences → Windows Settings → Drive Maps

# Create mapped drive via GPP Drive Maps:
# Action: Create
# Location: \\DC01\HR_Share
# Drive Letter: H:
# Label: HR Files
# Reconnect: Yes
# Item-level targeting: Group = GRP_HRAdmins
```

---

## 6. PowerShell GPO Deployment Example

```powershell
Import-Module GroupPolicy

# Create a new GPO
$gpo = New-GPO -Name "CORP - USB Block - v1" -Domain "corp.homelab.local"

# Set registry value to block USB storage
Set-GPRegistryValue -Name "CORP - USB Block - v1" `
    -Key "HKLM\SYSTEM\CurrentControlSet\Services\USBSTOR" `
    -ValueName "Start" `
    -Type DWord `
    -Value 4

# Link to OU
New-GPLink -Name "CORP - USB Block - v1" `
    -Target "OU=Desktops,OU=Computers,DC=corp,DC=homelab,DC=local" `
    -LinkEnabled Yes

# Force GPO update on a specific client
Invoke-GPUpdate -Computer "WIN10-1" -Force -RandomDelayInMinutes 0
```

---

## GPO Troubleshooting Commands

```powershell
# Show applied GPOs on a computer
gpresult /R

# Generate HTML GPO report for a specific user/computer
gpresult /H C:\Reports\gpo-report.html /USER jsmith /COMPUTER WIN10-1

# Force group policy refresh
gpupdate /force

# Check GPO application events (Event ID 4016, 5016, 7016)
Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" |
    Where-Object { $_.Id -in @(4016,5016,7016) } |
    Select-Object TimeCreated,Id,Message |
    Format-List
```
