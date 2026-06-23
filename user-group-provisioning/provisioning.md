# User & Group Provisioning

## Overview

All user and group provisioning is scripted via PowerShell to ensure consistency, auditability, and repeatability. Scripts are designed to be re-runnable (idempotent where possible).

---

## Group Naming Convention

| Prefix | Scope | Type | Example |
|---|---|---|---|
| `GRP_` | Global | Security | `GRP_IT_Admins` |
| `GRP_DL_` | Domain Local | Security (resource access) | `GRP_DL_FileShare_HR_RW` |
| `GRP_DIST_` | Global | Distribution | `GRP_DIST_AllStaff` |

---

## Create Security Groups

```powershell
$groupOU = "OU=Security Groups,OU=Groups,DC=corp,DC=homelab,DC=local"

$groups = @(
    @{ Name="GRP_IT_Admins";    Description="IT Department Administrators" },
    @{ Name="GRP_HelpDesk";     Description="Help Desk Technicians" },
    @{ Name="GRP_HRAdmins";     Description="HR OU Delegated Admins" },
    @{ Name="GRP_Finance";      Description="Finance Department Users" },
    @{ Name="GRP_AllUsers";     Description="All standard domain users" }
)

foreach ($g in $groups) {
    if (-not (Get-ADGroup -Filter {Name -eq $g.Name} -ErrorAction SilentlyContinue)) {
        New-ADGroup -Name $g.Name `
                    -GroupScope Global `
                    -GroupCategory Security `
                    -Description $g.Description `
                    -Path $groupOU
        Write-Host "Created group: $($g.Name)"
    }
}
```

---

## Bulk User Creation from CSV

### Sample `users.csv`

```csv
FirstName,LastName,Department,Title,OU
John,Smith,IT Department,Systems Administrator,IT Department
Jane,Doe,HR Department,HR Generalist,HR Department
Bob,Johnson,Finance Department,Accountant,Finance Department
Alice,Williams,IT Department,Help Desk Technician,IT Department
```

### Bulk Create Script

```powershell
$csvPath  = ".\users.csv"
$domain   = "corp.homelab.local"
$baseOU   = "OU=Users,DC=corp,DC=homelab,DC=local"
$defPass  = ConvertTo-SecureString "Welcome1!ChangeMe" -AsPlainText -Force

Import-Csv $csvPath | ForEach-Object {
    $username   = ($_.FirstName[0] + $_.LastName).ToLower()   # jsmith
    $upn        = "$username@$domain"
    $targetOU   = "OU=$($_.OU),$baseOU"
    $displayName = "$($_.FirstName) $($_.LastName)"

    if (Get-ADUser -Filter {SamAccountName -eq $username} -ErrorAction SilentlyContinue) {
        Write-Warning "User $username already exists — skipping."
        return
    }

    New-ADUser `
        -SamAccountName   $username `
        -UserPrincipalName $upn `
        -Name             $displayName `
        -GivenName        $_.FirstName `
        -Surname          $_.LastName `
        -DisplayName      $displayName `
        -Department       $_.Department `
        -Title            $_.Title `
        -Path             $targetOU `
        -AccountPassword  $defPass `
        -Enabled          $true `
        -ChangePasswordAtLogon $true

    # Add to GRP_AllUsers
    Add-ADGroupMember -Identity "GRP_AllUsers" -Members $username

    # Department-specific group
    switch ($_.Department) {
        "IT Department"      { Add-ADGroupMember -Identity "GRP_IT_Admins" -Members $username }
        "HR Department"      { Add-ADGroupMember -Identity "GRP_HRAdmins"  -Members $username }
        "Finance Department" { Add-ADGroupMember -Identity "GRP_Finance"   -Members $username }
    }

    Write-Host "Created user: $username in $targetOU"
}
```

---

## Offboarding Script

```powershell
param([string]$Username)

$disabledOU = "OU=Disabled Accounts,OU=Users,DC=corp,DC=homelab,DC=local"

# Disable account
Disable-ADAccount -Identity $Username

# Reset password to random value (prevents further auth)
$randomPass = ConvertTo-SecureString ([System.Web.Security.Membership]::GeneratePassword(16,4)) `
              -AsPlainText -Force
Set-ADAccountPassword -Identity $Username -NewPassword $randomPass -Reset

# Remove from all groups (except Domain Users)
$user = Get-ADUser $Username -Properties MemberOf
$user.MemberOf | ForEach-Object {
    Remove-ADGroupMember -Identity $_ -Members $Username -Confirm:$false
}

# Move to Disabled OU
Move-ADObject -Identity $user.DistinguishedName -TargetPath $disabledOU

# Stamp the description
Set-ADUser $Username -Description "DISABLED $(Get-Date -Format 'yyyy-MM-dd') — offboarded"

Write-Host "User $Username has been offboarded."
```

---

## Verify Provisioning

```powershell
# List all users and their OUs
Get-ADUser -Filter * -Properties Department,Title,Enabled |
    Select-Object Name,SamAccountName,Department,Title,Enabled |
    Sort-Object Department,Name |
    Format-Table -AutoSize

# Show group memberships
Get-ADGroup -Filter * -SearchBase "OU=Security Groups,OU=Groups,DC=corp,DC=homelab,DC=local" |
    ForEach-Object {
        Write-Host "`n=== $($_.Name) ===" -ForegroundColor Cyan
        Get-ADGroupMember $_ | Select-Object Name,SamAccountName
    }
```
