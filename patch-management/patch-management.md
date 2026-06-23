# Patch Management

## Strategy

Patch management in this lab uses Windows Server Update Services (WSUS) to mirror Microsoft Update, approve patches through a staged process, and push approved updates to clients via Group Policy.

---

## WSUS Installation

```powershell
# Install WSUS with local content storage
Install-WindowsFeature -Name UpdateServices, UpdateServices-UI `
    -IncludeManagementTools

# Configure WSUS post-install (run once)
& "C:\Program Files\Update Services\Tools\WsusUtil.exe" postinstall `
    CONTENT_DIR="C:\WSUS"
```

---

## WSUS Initial Configuration

After installation, complete setup via the WSUS Console or PowerShell:

```powershell
$wsus = Get-WsusServer -Name DC01 -PortNumber 8530

# Set upstream sync source
$wsus.SetUpdateSource([Microsoft.UpdateServices.Administration.UpdateSource]::MicrosoftUpdate)
$wsus.Save()

# Set product subscriptions
$wsus.GetUpdateCategories() | Where-Object {
    $_.Title -in @("Windows 10","Windows 11","Windows Server 2022")
} | ForEach-Object {
    $wsus.SetUpdateCategorySubscription($_, $true)
}

# Set classification subscriptions
$wsus.GetUpdateClassifications() | Where-Object {
    $_.Title -in @("Critical Updates","Security Updates","Definition Updates","Service Packs")
} | ForEach-Object {
    $wsus.SetUpdateClassificationSubscription($_, $true)
}

# Set sync schedule (daily at 2:00 AM)
$config = $wsus.GetConfiguration()
$config.SyncFromMicrosoftUpdate = $true
$config.HostBinariesOnMicrosoftUpdate = $false
$config.Save()
```

---

## Computer Groups in WSUS

| WSUS Group | Members | Approval Lag |
|---|---|---|
| `Lab-Servers` | DC01 | 14 days after testing |
| `Lab-Workstations-IT` | WIN10-1 | 7 days (early ring) |
| `Lab-Workstations-Users` | WIN10-2 | 14 days |

```powershell
# Create computer groups
$wsus.CreateComputerTargetGroup("Lab-Servers")
$wsus.CreateComputerTargetGroup("Lab-Workstations-IT")
$wsus.CreateComputerTargetGroup("Lab-Workstations-Users")
```

---

## Patch Approval Workflow

```
Microsoft Update (upstream)
        │
        ▼
   WSUS Syncs (daily 2 AM)
        │
        ▼
 Manual Review in WSUS Console
        │
        ├─► Critical/Security → Auto-approve for Workstations (7-day delay)
        │
        └─► All others → Manual review → Staged approval
                              │
                              ├─► Lab-Workstations-IT (test ring, day 0)
                              ├─► Lab-Workstations-Users (7 days after IT ring)
                              └─► Lab-Servers (14 days after IT ring, maintenance window)
```

---

## Auto-Approval Rules (WSUS Console)

| Rule Name | Classifications | Groups | Deadline |
|---|---|---|---|
| Critical Security Auto-Approve | Critical Updates, Security Updates | Lab-Workstations-IT | 7 days |
| Definition Updates | Definition Updates | All Computers | 1 day |

---

## GPO: Force Clients to Use WSUS

*See [GPO Examples](../gpo-examples/gpo-overview.md) — Section 3 (WSUS Update Policy).*

Client targeting is set via GPO registry key `HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate`.

---

## Patch Compliance Reporting

```powershell
# Check update status of all WSUS computers
$wsus = Get-WsusServer -Name DC01 -PortNumber 8530
$wsus.GetComputerTargets() | ForEach-Object {
    $status = $_.GetUpdateInstallationSummary()
    [PSCustomObject]@{
        Computer      = $_.FullDomainName
        NeedsUpdate   = $status.NotInstalledCount + $status.DownloadedCount
        Installed     = $status.InstalledCount
        Failed        = $status.FailedCount
        LastSync      = $_.LastSyncTime
    }
} | Format-Table -AutoSize
```

---

## Patch Tuesday Monthly Runbook

| Day | Action |
|---|---|
| Patch Tuesday | WSUS syncs overnight; review approved updates next morning |
| T+1 | Approve for IT workstation ring |
| T+7 | Verify IT ring successful; approve for user workstations |
| T+14 | Approve for servers; schedule during maintenance window (Sat 2–4 AM) |
| T+21 | Verify all systems compliant; document any failures |

---

## Maintenance Window Script (Server Patching)

```powershell
# Run on DC01 during maintenance window
$updateSession = New-Object -ComObject Microsoft.Update.Session
$searcher = $updateSession.CreateUpdateSearcher()
$result = $searcher.Search("IsInstalled=0 AND Type='Software'")

if ($result.Updates.Count -eq 0) {
    Write-Host "No pending updates." ; exit
}

$downloader = $updateSession.CreateUpdateDownloader()
$downloader.Updates = $result.Updates
$downloader.Download()

$installer = $updateSession.CreateUpdateInstaller()
$installer.Updates = $result.Updates
$installResult = $installer.Install()

Write-Host "Result: $($installResult.ResultCode)"
if ($installResult.RebootRequired) {
    Write-Host "Reboot required. Scheduling for 3:00 AM..."
    shutdown /r /t (New-TimeSpan -End (Get-Date).Date.AddDays(1).AddHours(3)).TotalSeconds
}
```
