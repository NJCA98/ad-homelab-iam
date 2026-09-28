# AD Homelab — Identity & Access Management Build

A self-hosted Active Directory domain controller built from scratch on a home network, designed to practice IAM concepts used in enterprise environments: identity lifecycle, role-based access control (RBAC), DNS architecture, and centralized security policy enforcement.

## Overview

| | |
|---|---|
| **Platform** | Windows Server 2022 (Evaluation), VirtualBox |
| **Networking** | Bridged adapter on home Wi-Fi, static IP, self-referencing DNS |
| **Domain** | `lab.local` (single forest, single domain) |
| **Host** | `DC01` — Domain Controller, DNS Server |

## Architecture

```
Wi-Fi Router (Gateway)
   |
   |  DHCP + external DNS forwarding
   v
DC01 (VM, bridged to Wi-Fi)
   - Static IP on home LAN subnet
   - AD DS + DNS Server roles
   - DNS points to itself (self-referencing)
   v
Host Laptop
   - Runs VirtualBox
   - Same Wi-Fi / same LAN subnet as DC01
```

Bridged networking (rather than NAT) puts the VM directly on the home LAN with its own real IP address, so it behaves like any other device on the network — a requirement for AD DS, since domain members need to reach the DC directly.

## What Was Built

**Network foundation**
- Configured VirtualBox bridged networking over Wi-Fi so the VM receives a real LAN-routable IP
- Assigned a static IP, subnet mask, and gateway matching the home network
- Set DC01's DNS to point to itself, required for AD DS to register and resolve its own service records
- Configured a DNS forwarder so internal (`lab.local`) names resolve locally while external names (e.g. `google.com`) forward out through the router — a split-DNS pattern used in production environments

**Active Directory**
- Installed the AD DS role and promoted DC01 to the first domain controller in a new forest (`lab.local`)
- Verified domain health end-to-end with `dcdiag`

**Identity & access (RBAC)**
- Created custom Organizational Units (`Users`, `Groups`) rather than relying on AD's default containers, to enable Group Policy targeting
- Created security groups representing roles (`HR-Team`, `IT-Helpdesk`)
- Created user identities (`jdoe`, `asmith`) and assigned them to their respective role-based groups — access is granted through group membership, not directly to individual users, following least-privilege / role-based access principles used in real IAM programs

**Security baseline**
- Enforced a domain-wide password policy via Group Policy: 12-character minimum length, complexity requirements enabled

## Verification

All configuration was validated via PowerShell rather than relying solely on the GUI:

```powershell
Import-Module ActiveDirectory

Get-ADUser jdoe
Get-ADGroupMember "HR-Team"
Get-ADGroupMember "IT-Helpdesk"
Get-DnsServerForwarder
dcdiag
```

See [`/screenshots`](./screenshots) for verification evidence, including:
- `ipconfig /all` showing static IP, gateway, and self-referencing DNS
- `dcdiag` with all tests passing
- Domain login screen (`LAB\Administrator`)
- `nslookup google.com` resolving successfully through the forwarder
- ADUC showing the `Users` and `Groups` OUs, group membership, and user objects
- Group Policy password policy settings

## Why This Matters

This build mirrors the identity lifecycle and access governance patterns used in production IAM: centralized authentication, role-based (not per-user) access grants, split DNS, and policy enforced once at the domain level rather than per-machine. It was built independently, outside of assigned work duties, to reinforce architecture-level understanding of how these systems fit together.

## Coming Next (Part 2)

- Add a Windows client VM
- Point its DNS at DC01
- Join it to the `lab.local` domain
- Log in as a domain user (`jdoe` / `asmith`) to validate the full identity chain from directory to endpoint

---

*Built as a hands-on lab to strengthen practical Active Directory / IAM skills. Sensitive details (passwords, real IP ranges) have been redacted or generalized in all screenshots.*
