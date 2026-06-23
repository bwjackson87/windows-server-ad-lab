# Backup & Restore

## Backup Strategy

The lab uses a layered backup approach combining Windows Server Backup (built-in), AD-specific backups, and VM-level snapshots.

| Layer | Tool | Target | Frequency | Retention |
|---|---|---|---|---|
| VM Snapshot | Hypervisor | Entire VM state | Weekly (pre-patching) | 2 snapshots |
| System State | Windows Server Backup | AD DS, Registry, SYSVOL, Boot files | Daily 1 AM | 7 days |
| Full Server | Windows Server Backup | All volumes | Weekly Sunday 12 AM | 4 weeks |
| DHCP Export | PowerShell | DHCP database | Weekly | 4 weeks |
| GPO Backup | PowerShell | All GPOs | Weekly | 4 weeks |

---

## 1. Install Windows Server Backup

```powershell
Install-WindowsFeature Windows-Server-Backup -IncludeManagementTools
```

---

## 2. Configure Daily System State Backup

```powershell
# Create backup policy
$policy = New-WBPolicy

# Set backup target (external drive or network share)
$backupTarget = New-WBBackupTarget -NetworkPath "\\192.168.10.1\Backups\DC01" `
    -Credential (Get-Credential)
Add-WBBackupTarget -Policy $policy -Target $backupTarget

# Add system state
Add-WBSystemState -Policy $policy

# Set schedule (1:00 AM daily)
Set-WBSchedule -Policy $policy -Schedule 01:00

# Enable VSS full backup
Set-WBVssBackupOption -Policy $policy -VssFullBackup

# Apply policy
Set-WBPolicy -Policy $policy -Force

Write-Host "Backup policy applied."
```

---

## 3. Backup All GPOs

```powershell
$gpoBackupPath = "C:\Backup\GPOs\$(Get-Date -Format yyyyMMdd)"
New-Item -ItemType Directory -Path $gpoBackupPath -Force | Out-Null

Backup-GPO -All -Path $gpoBackupPath
Write-Host "All GPOs backed up to $gpoBackupPath"
```

---

## 4. Backup DHCP Configuration

```powershell
$dhcpBackupPath = "C:\Backup\DHCP\$(Get-Date -Format yyyyMMdd)"
New-Item -ItemType Directory -Path $dhcpBackupPath -Force | Out-Null

Export-DhcpServer -ComputerName DC01 `
    -File "$dhcpBackupPath\dhcp-export.xml" `
    -Leases -Force
```

---

## 5. AD-Aware Restore Procedures

### Restore Individual AD Object (AD Recycle Bin)

```powershell
# Find deleted object
Get-ADObject -Filter {DisplayName -eq "John Smith"} `
    -IncludeDeletedObjects -Properties *

# Restore object
Restore-ADObject -Identity "<ObjectGUID>"
```

### Authoritative Restore (for replicated deletions)

> Use only when an object was deleted and replicated across all DCs.

1. Boot DC into **Directory Services Restore Mode (DSRM)**
   - Restart → F8 → Directory Services Restore Mode
2. Restore system state from backup:
   ```cmd
   wbadmin get versions -backuptarget:\\192.168.10.1\Backups\DC01
   wbadmin start systemstaterecovery -version:<ID> -backuptarget:\\192.168.10.1\Backups\DC01
   ```
3. Mark objects as authoritative:
   ```cmd
   ntdsutil
   activate instance ntds
   authoritative restore
   restore subtree "OU=IT Department,OU=Users,DC=corp,DC=homelab,DC=local"
   quit
   quit
   ```
4. Reboot normally — restored objects replicate outward.

---

## 6. Bare-Metal Restore Procedure

> Used when DC01 OS is unrecoverable. Requires Windows Server Backup WinRE media.

1. Boot from Windows Server 2022 installation media
2. Select **Repair your computer → Troubleshoot → System Image Recovery**
3. Point to backup location: `\\192.168.10.1\Backups\DC01`
4. Select most recent backup version
5. Restore — server reboots into restored state (~30 min for 60 GB)

### Post-Restore Verification

```powershell
# Verify AD DS is healthy
dcdiag /v

# Verify SYSVOL is shared
net share | findstr SYSVOL

# Verify DNS is responding
Resolve-DnsName DC01.corp.homelab.local

# Check replication (if additional DCs exist)
repadmin /replsummary

# Verify DHCP is authorized
Get-DhcpServerInDC
```

---

## 7. Restore Specific GPO

```powershell
# List available GPO backups
Get-ChildItem "C:\Backup\GPOs\20241101" | Select-Object Name

# Restore a single GPO from backup
Restore-GPO -Name "CORP - Security Baseline - v1" `
    -Path "C:\Backup\GPOs\20241101"
```

---

## Backup Monitoring

```powershell
# Check last backup status
Get-WBSummary

# Check backup job history
Get-WBJob -Previous 10

# Alert if last backup is older than 24 hours
$lastBackup = (Get-WBSummary).LastSuccessfulBackupTime
if ($lastBackup -lt (Get-Date).AddHours(-24)) {
    Write-Warning "ALERT: Last successful backup was $lastBackup — over 24 hours ago!"
}
```
