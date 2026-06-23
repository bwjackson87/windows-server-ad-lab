# DNS & DHCP Configuration Notes

---

## DNS

### Architecture

DNS is AD-integrated and hosted on DC01. All zone data is stored in Active Directory (replicated automatically) rather than flat files.

| Zone | Type | Replication |
|---|---|---|
| `corp.homelab.local` | Primary (AD-integrated) | Forest-wide |
| `10.168.192.in-addr.arpa` | Reverse lookup (AD-integrated) | Forest-wide |

### Key DNS Records

| Name | Type | Value | Purpose |
|---|---|---|---|
| `DC01` | A | 192.168.10.10 | Domain controller |
| `_ldap._tcp.corp.homelab.local` | SRV | DC01:389 | AD client discovery |
| `_kerberos._tcp.corp.homelab.local` | SRV | DC01:88 | Kerberos |
| `_gc._tcp.corp.homelab.local` | SRV | DC01:3268 | Global Catalog |

### DNS Forwarders

```powershell
# View current forwarders
Get-DnsServerForwarder

# Set forwarders
Set-DnsServerForwarder -IPAddress "8.8.8.8","8.8.4.4"

# Test name resolution
Resolve-DnsName google.com -Server 192.168.10.10
Resolve-DnsName DC01.corp.homelab.local -Server 192.168.10.10
```

### Create DNS Records Manually

```powershell
# Add A record
Add-DnsServerResourceRecordA `
    -ZoneName "corp.homelab.local" `
    -Name "fileserver01" `
    -IPv4Address "192.168.10.20"

# Add PTR record (reverse lookup)
Add-DnsServerResourceRecordPtr `
    -ZoneName "10.168.192.in-addr.arpa" `
    -Name "20" `
    -PtrDomainName "fileserver01.corp.homelab.local"

# Add CNAME record
Add-DnsServerResourceRecordCName `
    -ZoneName "corp.homelab.local" `
    -Name "intranet" `
    -HostNameAlias "DC01.corp.homelab.local"
```

### Enable DNS Aging & Scavenging

```powershell
# Enable scavenging on the zone (removes stale records)
Set-DnsServerZoneAging -ZoneName "corp.homelab.local" `
    -Aging $true `
    -ScavengeServers 192.168.10.10 `
    -NoRefreshInterval 7.00:00:00 `
    -RefreshInterval 7.00:00:00

# Trigger immediate scavenging
Start-DnsServerScavenging -Force
```

### DNS Troubleshooting

```powershell
# Check DNS server health
dcdiag /test:DNS /v

# View DNS server event log
Get-WinEvent -LogName "DNS Server" | Select-Object -First 20 TimeCreated,Id,Message

# Flush DNS cache on client
ipconfig /flushdns

# Test SRV records (critical for AD)
nslookup -type=srv _ldap._tcp.corp.homelab.local
nslookup -type=srv _kerberos._tcp.corp.homelab.local
```

---

## DHCP

### Scope Configuration

| Setting | Value |
|---|---|
| Scope Name | LabScope |
| Subnet | 192.168.10.0/24 |
| Start IP | 192.168.10.100 |
| End IP | 192.168.10.200 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.10.1 |
| DNS Server | 192.168.10.10 |
| DNS Domain | corp.homelab.local |
| Lease Duration | 8 hours |

### Exclusions & Reservations

```powershell
# Exclude static IP range from scope
Add-DhcpServerv4ExclusionRange `
    -ScopeId 192.168.10.0 `
    -StartRange 192.168.10.100 `
    -EndRange 192.168.10.110

# Create reservation for WIN10-1
Add-DhcpServerv4Reservation `
    -ScopeId 192.168.10.0 `
    -IPAddress 192.168.10.101 `
    -ClientId "AA-BB-CC-DD-EE-FF" `
    -Description "WIN10-1 workstation"
```

### DHCP Failover (Production Pattern — documented for reference)

```powershell
# In a multi-DC environment, configure DHCP failover for HA
Add-DhcpServerv4Failover `
    -Name "LabFailover" `
    -ScopeId 192.168.10.0 `
    -PartnerServer "DC02.corp.homelab.local" `
    -Mode LoadBalance `
    -LoadBalancePercent 50 `
    -SharedSecret "FailoverSecret2024!"
```

### DHCP Reporting & Lease Management

```powershell
# View active leases
Get-DhcpServerv4Lease -ScopeId 192.168.10.0

# View lease statistics
Get-DhcpServerv4ScopeStatistics -ScopeId 192.168.10.0

# Export DHCP config (backup)
Export-DhcpServer -File "C:\Backup\DHCP_Export_$(Get-Date -Format yyyyMMdd).xml" -Leases

# Import DHCP config (restore)
Import-DhcpServer -File "C:\Backup\DHCP_Export_20241101.xml" -BackupPath "C:\Backup\DHCP"
```

### DHCP Audit Logs

DHCP audit logs are stored at: `C:\Windows\System32\dhcp\`

Log files are named `DhcpSrvLog-XXX.log` (Mon–Sun).

Key Event IDs:

| ID | Meaning |
|---|---|
| 10 | New IP address lease |
| 11 | Lease renewed |
| 12 | Lease released |
| 13 | IP address found in use on network |
| 20 | IP address leased (authoritative) |
| 30 | DNS update request |
