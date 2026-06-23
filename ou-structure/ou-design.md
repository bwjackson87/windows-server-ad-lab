# Organizational Unit (OU) Structure

## Design Philosophy

The OU hierarchy is designed to:
1. Mirror real-world department/function groupings
2. Enable targeted GPO application without excessive WMI filtering
3. Support delegation of administrative tasks at the OU level
4. Facilitate clean reporting and bulk management via PowerShell

---

## OU Hierarchy

```
corp.homelab.local
└── CORP                          (domain root)
    ├── _Admin                    (privileged accounts — blocked from user GPOs)
    │   ├── Admin Users
    │   └── Service Accounts
    ├── Computers
    │   ├── Desktops
    │   │   ├── IT
    │   │   └── Users
    │   ├── Laptops
    │   └── Servers
    ├── Groups
    │   ├── Security Groups
    │   └── Distribution Groups
    └── Users
        ├── IT Department
        ├── HR Department
        ├── Finance Department
        └── Disabled Accounts     (tombstone for offboarded users)
```

---

## OU Descriptions

| OU | Purpose | GPO Linked |
|---|---|---|
| `_Admin\Admin Users` | Named admin accounts (no email, no browsing) | Admin Security Baseline |
| `_Admin\Service Accounts` | gMSA and standard service accounts | Service Account Policy |
| `Computers\Desktops\IT` | IT staff workstations | IT Workstation Policy |
| `Computers\Desktops\Users` | Standard user workstations | User Workstation Policy |
| `Computers\Servers` | Member servers (future expansion) | Server Baseline Policy |
| `Groups\Security Groups` | All AD security groups | (none) |
| `Users\IT Department` | IT staff user accounts | IT User Policy |
| `Users\HR Department` | HR user accounts | HR User Policy, Folder Redirection |
| `Users\Finance Department` | Finance user accounts | Finance Policy, Auditing Policy |
| `Users\Disabled Accounts` | Offboarded — disabled, stripped of groups | Block All GPOs |

---

## Create OU Structure (PowerShell)

```powershell
$domain = "DC=corp,DC=homelab,DC=local"

# Top-level OUs
foreach ($ou in @("_Admin","Computers","Groups","Users")) {
    New-ADOrganizationalUnit -Name $ou -Path $domain -ProtectedFromAccidentalDeletion $true
}

# _Admin children
New-ADOrganizationalUnit -Name "Admin Users"      -Path "OU=_Admin,$domain"
New-ADOrganizationalUnit -Name "Service Accounts" -Path "OU=_Admin,$domain"

# Computers children
New-ADOrganizationalUnit -Name "Desktops" -Path "OU=Computers,$domain"
New-ADOrganizationalUnit -Name "Laptops"  -Path "OU=Computers,$domain"
New-ADOrganizationalUnit -Name "Servers"  -Path "OU=Computers,$domain"

# Desktops sub-OUs
New-ADOrganizationalUnit -Name "IT"    -Path "OU=Desktops,OU=Computers,$domain"
New-ADOrganizationalUnit -Name "Users" -Path "OU=Desktops,OU=Computers,$domain"

# Groups children
New-ADOrganizationalUnit -Name "Security Groups"     -Path "OU=Groups,$domain"
New-ADOrganizationalUnit -Name "Distribution Groups" -Path "OU=Groups,$domain"

# Users children
foreach ($dept in @("IT Department","HR Department","Finance Department","Disabled Accounts")) {
    New-ADOrganizationalUnit -Name $dept -Path "OU=Users,$domain"
}
```

---

## Delegation of Control

| OU | Delegated To | Permissions |
|---|---|---|
| `Users\HR Department` | `GRP_HRAdmins` | Reset passwords, unlock accounts |
| `Computers\Desktops\Users` | `GRP_HelpDesk` | Join computers, manage accounts |
| `Users\Disabled Accounts` | `GRP_ITAdmins` | Full control (for offboarding) |

```powershell
# Example: delegate password reset to HR admins on the HR OU
$hrOU = "OU=HR Department,OU=Users,DC=corp,DC=homelab,DC=local"
$group = Get-ADGroup "GRP_HRAdmins"
dsacls $hrOU /G "$($group.SID):CA;Reset Password;user"
```
