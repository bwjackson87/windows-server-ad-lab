---
> **Security & Sanitization Notice:** This repository contains sanitized, lab-safe code and documentation. It does not include proprietary, classified, sensitive, or employer-owned data. Hostnames, domains, usernames, IP addresses, and operational details are fictionalized or generalized. See [SECURITY_NOTICE.md](SECURITY_NOTICE.md) for full details.
---

# Windows Server & Active Directory Homelab

## Overview
A hands-on homelab that simulates an enterprise Windows Server environment. Built on Windows Server 2022 with Active Directory Domain Services, this lab covers the full Systems Administrator lifecycle: domain controller deployment, organizational structure design, user and group provisioning, Group Policy management, DNS/DHCP configuration, patch management via WSUS, backup and restore procedures, and CIS/STIG security hardening.

## Problem It Solves
Hands-on experience with enterprise Windows infrastructure is difficult to demonstrate without access to a production environment. This lab creates a fully functional, isolated Windows domain that replicates the configuration patterns found in enterprise environments — providing a practical foundation for Systems Administrator and IT operations roles that require proven AD, GPO, and Windows Server skills.

## Key Features
- Full Active Directory forest deployment from scratch on Windows Server 2022
- Hierarchical OU structure mirroring real-world department/function groupings
- Bulk user and group provisioning via PowerShell (CSV-driven, idempotent scripts)
- Group Policy Objects covering security baseline, USB blocking, WSUS targeting, and drive mapping
- AD-integrated DNS with forward and reverse lookup zones, scavenging, and forwarders
- DHCP scope with reservations and exclusions; DHCP export/import for backup
- WSUS deployment with staged approval rings (IT → Users → Servers) and Patch Tuesday runbook
- Windows Server Backup for system state and full server; GPO and DHCP backup scripts
- Authoritative AD restore and bare-metal recovery procedures documented step-by-step
- CIS Benchmark / DISA STIG security checklist with 50+ controls tracked and verified

## Technologies Used
- Windows Server 2022 Standard
- Active Directory Domain Services (AD DS)
- Group Policy Management Console (GPMC) + PowerShell `GroupPolicy` module
- Windows Server Update Services (WSUS)
- Windows Server Backup
- PowerShell 5.1+ (all provisioning and configuration scripts)
- VMware Workstation / Hyper-V (hypervisor)

## Example Use Case
A new hire joins the IT department. Their account is created via the bulk provisioning script, placed in the correct departmental OU, and automatically inherits the appropriate GPOs (mapped drives, wallpaper, software restrictions). Their workstation joins the domain and begins receiving approved patches from WSUS within the next scheduled scan window — the entire onboarding sequence from account creation to a fully configured, policy-compliant workstation requires no manual GUI steps.

## How to Run

This is a lab environment, not a deployable application. To replicate it:

1. Create VMs matching the [lab topology](diagrams/lab-topology.md) (1 server + 2 clients, internal virtual switch)
2. Follow [domain-controller/dc-setup.md](domain-controller/dc-setup.md) to promote the DC and configure DNS/DHCP
3. Run the OU creation script in [ou-structure/ou-design.md](ou-structure/ou-design.md)
4. Populate users from a CSV using [user-group-provisioning/provisioning.md](user-group-provisioning/provisioning.md)
5. Apply GPOs documented in [gpo-examples/gpo-overview.md](gpo-examples/gpo-overview.md)
6. Configure WSUS and backup following the respective guides

All scripts are ready to run with PowerShell 5.1+ on Windows Server 2022.

## Example Output

**OU and user verification:**
```powershell
Get-ADUser -Filter * -Properties Department | Select Name,SamAccountName,Department | Format-Table
```
```
Name           SamAccountName  Department
----           --------------  ----------
John Smith     jsmith          IT Department
Jane Doe       jdoe            HR Department
Bob Johnson    bjohnson        Finance Department
```

**GPO application report:**
```
gpresult /R
...
Applied Group Policy Objects:
    CORP - Security Baseline - v1
    CORP - User Workstation Policy - v1
    CORP - WSUS Policy - v1
```

**WSUS compliance summary:**
```
Computer      NeedsUpdate  Installed  Failed  LastSync
--------      -----------  ---------  ------  --------
WIN10-1       0            142        0       6/21/2026
WIN10-2       2            140        0       6/21/2026
```

## Security Notes
- All lab activity is performed in isolated virtual machines with no connection to production systems or the public internet
- The domain `corp.homelab.local` uses a non-routable `.local` TLD and is not exposed outside the host machine
- Credentials shown in documentation are lab-only examples; never use example passwords in production environments
- The security baseline checklist follows CIS Microsoft Windows Server 2022 Benchmark v1.0 and DISA STIG guidelines — see [security-baseline/security-checklist.md](security-baseline/security-checklist.md)

## Lessons Learned
- The FSMO role holder must be reachable at domain join time — a static IP on the DC must be set before promoting, or clients will fail to locate the PDC emulator even with DNS working correctly
- GPO filtering via OU placement is far more maintainable than WMI filters for most use cases; WMI filters are appropriate only for hardware-specific or OS-version-specific targeting
- The AD Recycle Bin must be enabled before an accidental deletion occurs — enabling it retroactively does not recover already-deleted objects
- WSUS requires the "Update Services" feature plus a separate `postinstall` command to configure the content directory; skipping the postinstall step causes the WSUS console to open but fail silently on first sync

---

| Section | Document |
|---|---|
| Lab Diagram | [diagrams/lab-topology.md](diagrams/lab-topology.md) |
| Domain Controller Setup | [domain-controller/dc-setup.md](domain-controller/dc-setup.md) |
| OU Structure | [ou-structure/ou-design.md](ou-structure/ou-design.md) |
| User & Group Provisioning | [user-group-provisioning/provisioning.md](user-group-provisioning/provisioning.md) |
| GPO Examples | [gpo-examples/gpo-overview.md](gpo-examples/gpo-overview.md) |
| DNS & DHCP | [dns-dhcp/dns-dhcp-notes.md](dns-dhcp/dns-dhcp-notes.md) |
| Patch Management | [patch-management/patch-management.md](patch-management/patch-management.md) |
| Backup & Restore | [backup-restore/backup-restore.md](backup-restore/backup-restore.md) |
| Security Baseline | [security-baseline/security-checklist.md](security-baseline/security-checklist.md) |
