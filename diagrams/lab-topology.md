# Lab Topology Diagram

## Network Architecture

```
                          INTERNET
                              │
                         [ Router ]
                       192.168.10.1
                              │
                    ┌─────────┴─────────┐
                    │   Virtual Switch   │
                    │  192.168.10.0/24   │
                    └──┬────────┬────────┘
                       │        │        │
              ┌────────┴──┐  ┌──┴──────┐  ┌──────────┐
              │    DC01    │  │  WIN10-1 │  │  WIN10-2  │
              │ WS 2022    │  │ Client 1 │  │  Client 2 │
              │.10.10      │  │ .10.101  │  │  .10.102  │
              │ DC + DNS   │  └──────────┘  └──────────┘
              │ + DHCP     │
              │ + WSUS     │
              └────────────┘
```

## VM Specifications

| VM | Role | OS | IP | RAM | vCPU |
|---|---|---|---|---|---|
| DC01 | Domain Controller, DNS, DHCP, WSUS | Windows Server 2022 Standard | 192.168.10.10 (static) | 4 GB | 2 |
| WIN10-1 | Domain-joined client (IT OU) | Windows 10 Pro 22H2 | 192.168.10.101 (DHCP) | 2 GB | 2 |
| WIN10-2 | Domain-joined client (Users OU) | Windows 10 Pro 22H2 | 192.168.10.102 (DHCP) | 2 GB | 2 |

## Domain Details

| Setting | Value |
|---|---|
| Forest/Domain Name | `corp.homelab.local` |
| NetBIOS Name | `CORP` |
| Forest Functional Level | Windows Server 2016 |
| Domain Functional Level | Windows Server 2016 |
| FSMO Roles | All on DC01 (single DC lab) |
| Global Catalog | DC01 |

## Network Services

| Service | Server | Details |
|---|---|---|
| DNS | DC01 | AD-integrated primary zone, forwarders → 8.8.8.8 |
| DHCP | DC01 | Scope: 192.168.10.100–200, lease 8h |
| WSUS | DC01 | Approves critical/security updates, GPO-pushed to clients |
| NTP | DC01 | Syncs to pool.ntp.org, clients sync to DC01 |

## Traffic Flow

```
Client Boot
    │
    ▼
DHCP Request → DC01 assigns IP + DNS + Gateway
    │
    ▼
Domain Join → Client contacts DC01 on port 389 (LDAP) / 636 (LDAPS)
    │
    ▼
User Logon → Kerberos ticket from DC01 (port 88)
    │
    ▼
GPO Application → SYSVOL share on DC01 (ports 445 / 135)
    │
    ▼
Windows Update → WSUS on DC01 (port 8530)
```
