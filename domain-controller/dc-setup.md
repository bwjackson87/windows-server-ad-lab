# Domain Controller Setup

## Overview

This document covers the full process of deploying a Windows Server 2022 domain controller, from OS installation through AD DS promotion.

---

## 1. Pre-Installation Checklist

- [ ] VM created with ≥ 4 GB RAM, 2 vCPUs, 60 GB disk
- [ ] Network adapter set to internal/host-only virtual switch
- [ ] Static IP assigned before promotion
- [ ] Windows Server 2022 ISO mounted and installed
- [ ] Windows activated (KMS/MAK or evaluation)
- [ ] All pending Windows Updates installed
- [ ] Computer name set to `DC01` before promotion

---

## 2. Set Static IP (PowerShell)

```powershell
# Set static IP on primary NIC
$NIC = Get-NetAdapter | Where-Object {$_.Status -eq "Up"}
New-NetIPAddress -InterfaceIndex $NIC.ifIndex `
    -IPAddress 192.168.10.10 `
    -PrefixLength 24 `
    -DefaultGateway 192.168.10.1

Set-DnsClientServerAddress -InterfaceIndex $NIC.ifIndex `
    -ServerAddresses ("192.168.10.10","8.8.8.8")
```

---

## 3. Rename the Server

```powershell
Rename-Computer -NewName "DC01" -Restart
```

---

## 4. Install AD DS Role

```powershell
Install-WindowsFeature -Name AD-Domain-Services `
    -IncludeManagementTools
```

---

## 5. Promote to Domain Controller (New Forest)

```powershell
Import-Module ADDSDeployment

Install-ADDSForest `
    -DomainName "corp.homelab.local" `
    -DomainNetbiosName "CORP" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -DatabasePath "C:\Windows\NTDS" `
    -SysvolPath "C:\Windows\SYSVOL" `
    -LogPath "C:\Windows\NTDS" `
    -SafeModeAdministratorPassword (ConvertTo-SecureString `
        "P@ssw0rd!Lab2024" -AsPlainText -Force) `
    -Force:$true
```

> Server will automatically restart. Log in as `CORP\Administrator` after reboot.

---

## 6. Verify Promotion

```powershell
# Verify AD DS is running
Get-ADDomain

# Check FSMO roles
netdom query fsmo

# Verify DNS zones
Get-DnsServerZone

# Check SYSVOL replication
dcdiag /test:sysvol
dcdiag /test:replications
```

**Expected `netdom query fsmo` output:**
```
Schema master          DC01.corp.homelab.local
Domain naming master   DC01.corp.homelab.local
PDC                    DC01.corp.homelab.local
RID pool manager       DC01.corp.homelab.local
Infrastructure master  DC01.corp.homelab.local
```

---

## 7. Configure DNS Forwarders

```powershell
Add-DnsServerForwarder -IPAddress "8.8.8.8"
Add-DnsServerForwarder -IPAddress "8.8.4.4"
```

---

## 8. Install and Authorize DHCP

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools

# Authorize DHCP in AD
Add-DhcpServerInDC -DnsName "DC01.corp.homelab.local" `
    -IPAddress 192.168.10.10

# Create DHCP scope
Add-DhcpServerv4Scope `
    -Name "LabScope" `
    -StartRange 192.168.10.100 `
    -EndRange 192.168.10.200 `
    -SubnetMask 255.255.255.0 `
    -State Active

Set-DhcpServerv4OptionValue `
    -ScopeId 192.168.10.0 `
    -Router 192.168.10.1 `
    -DnsServer 192.168.10.10 `
    -DnsDomain "corp.homelab.local"
```

---

## 9. Post-Promotion Hardening

```powershell
# Disable the built-in Administrator account after creating a named admin
Disable-ADAccount -Identity "Administrator"

# Enable AD Recycle Bin
Enable-ADOptionalFeature "Recycle Bin Feature" `
    -Scope ForestOrConfigurationSet `
    -Target "corp.homelab.local" -Confirm:$false

# Enable fine-grained audit policy
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Management" /success:enable /failure:enable
```
