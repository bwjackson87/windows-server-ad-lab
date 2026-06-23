# Windows Server & Active Directory Homelab

A hands-on homelab simulating an enterprise Windows Server environment with Active Directory, DNS/DHCP, Group Policy, patch management, backup/restore procedures, and security hardening.

Built to demonstrate Systems Administrator skills relevant to enterprise IT environments.

---

## Lab Overview

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation / Hyper-V |
| Domain Controller | Windows Server 2022 |
| Client Machines | Windows 10/11 Pro (x2) |
| Domain Name | `corp.homelab.local` |
| IP Scheme | `192.168.10.0/24` |
| DNS | AD-integrated, forwarders to `8.8.8.8` |
| DHCP | Scope `192.168.10.100–200` |

---

## Repository Structure

```
windows-server-ad-lab/
├── diagrams/               # Lab topology diagrams
├── domain-controller/      # DC setup, promotion, and config
├── ou-structure/           # OU design and documentation
├── user-group-provisioning/ # User/group creation scripts
├── gpo-examples/           # Group Policy Object examples
├── dns-dhcp/               # DNS and DHCP configuration notes
├── patch-management/       # WSUS and patching procedures
├── backup-restore/         # Backup strategy and restore procedures
└── security-baseline/      # CIS/STIG security checklist
```

---

## Quick Links

- [Lab Diagram](diagrams/lab-topology.md)
- [Domain Controller Setup](domain-controller/dc-setup.md)
- [OU Structure](ou-structure/ou-design.md)
- [User & Group Provisioning](user-group-provisioning/provisioning.md)
- [GPO Examples](gpo-examples/gpo-overview.md)
- [DNS & DHCP Notes](dns-dhcp/dns-dhcp-notes.md)
- [Patch Management](patch-management/patch-management.md)
- [Backup & Restore](backup-restore/backup-restore.md)
- [Security Baseline Checklist](security-baseline/security-checklist.md)

---

## Skills Demonstrated

- Active Directory DS installation and promotion
- Domain, forest, and trust configuration
- Organizational Unit (OU) hierarchy design
- Bulk user and group provisioning via PowerShell
- Group Policy design, linking, and troubleshooting
- AD-integrated DNS zone management
- DHCP scope and reservation management
- Windows Server Update Services (WSUS) deployment
- Windows Server Backup and bare-metal restore
- CIS Benchmark / DISA STIG security hardening

---

> **Environment:** All lab activity performed in isolated VMs. No production systems involved.
