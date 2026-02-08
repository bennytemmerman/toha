---
title: "OpenTofu Project: Proxmox foundations"
date: 2026-02-07T10:17:23+02:00
hero: /images/posts/opentofu.png
description: A multiphase project to learn about OpenTofu, an orchestrator to automate and enhance deployments.
theme: Toha
menu:
  sidebar:
    name: OpenTofu 1. Proxmox foundations
    identifier: opentofu1
    parent: cat-orchestration
    weight: 600
---

# Securing the hypervisor before anything else

## Why hypervisor security comes first

In any virtualization environment, the hypervisor represents a high-value target. A compromised virtual machine is an incident. A compromised hypervisor is a catastrophe. Before building labs, automating infrastructure, or experimenting with offensive security techniques, it is essential to establish a hardened Proxmox VE baseline. This article walks through securing a single-node Proxmox installation as the foundation for a security-focused lab.

---

## Understanding Proxmox architecture

Proxmox VE is a type-1 hypervisor built on Debian Linux. It combines:

- KVM/QEMU for virtual machines
- LXC for containers
- A web-based management UI
- A REST API
- Native Linux storage and networking

Because everything ultimately runs on Linux, traditional Linux hardening principles apply directly to Proxmox.

---

## Threat model: Why this matters

If an attacker gains control of the Proxmox host, they can:

- Access every VM disk
- Snapshot live memory
- Inject malicious images
- Disable backups
- Persist invisibly

This makes hypervisor security non-negotiable.

---

## Creating a non-root administrative user

### Linux user creation
```bash
adduser labadmin
```

### Proxmox user and RBAC assignment
```bash
pveum user add labadmin@pam
pveum aclmod / -user labadmin@pam -role Administrator
```  
  
> Using named accounts instead of root improves accountability and auditability.

## SSH hardening with key-based authentication
### Generate an SSH Key
```bash
ssh-keygen -t ed25519 -C "labadmin@proxmox"
```
### Install the key
```bash
ssh-copy-id labadmin@<proxmox-ip>
```
### Harden SSH configuration
Edit /etc/ssh/sshd_config:
```bash
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```
### Restart SSH:
```bash
systemctl restart ssh
```
  
> This eliminates password-based attacks and disables remote root access.

## Reviewing Proxmox firewall defaults
By default, the Proxmox firewall is often disabled. This is not inherently wrong, but it must be a conscious decision.
Blindly enabling firewalls without understanding networking can be as dangerous as leaving systems exposed.
At this stage, documenting the default state is more important than enforcing rules.

## Backup awareness
Before automating backups, administrators should understand:
- What data is backed up?
- Where backups are stored?
- How restores work?

> Backup strategy becomes critical once VM templates are introduced.

## Documentation as a security control

Documenting architecture, access patterns, and policies reduces operational risk and improves incident response.

At minimum, record:
- Host IP addresses
- Storage configuration
- Network bridges
- Administrative users
- SSH policies

## Conclusion

Hardening the Proxmox host establishes trust in everything built on top of it. Automation, DevSecOps, and offensive security labs all depend on this foundation.

In the next phase, we move inside the virtual machines and begin building secure operating system baselines.

> Secure the hypervisor first — everything else depends on it.

## FAQ

### 1. Root login disabled remotely, do we still keep root?
Yes. Absolutely. Root never goes away. What we disable is remote root login, not root itself.
Root must exist, have a strong password stored securely, only be usable via:
- Physical console
- Out-of-band management (IPMI)
- Emergency recovery modes
Root is required for:
- Boot recovery
- Filesystem repair
- Broken auth recovery
- Catastrophic misconfiguration fixes

> Root over SSH has no attribution, is heavily brute-forced, turns one mistake into total compromise.

### 2. What if PermitRootLogin is set to no and I need root locally?
PermitRootLogin no only applies to SSH.
At the console, logged in as another user with sudo, in single-user mode, you can still become root.

> SSH policy ≠ local access policy.

### 3. What if I lose my private key AND password auth is disabled?
This is a real incident scenario, not a hypothetical.

Recovery paths:
- Console access
- Physical keyboard + screen
- Proxmox web console (if still logged in)
- Out-of-band management
- IPMI / iDRAC / iLO
- Single-user mode
- Rescue ISO

> It is important that there is documentation that says how to recover. If none of these exist, the system is effectively lost.

### 4. What would be an example of documentation?
#### Access Control Documentation
##### Proxmox Host Access Policy

##### Administrative Accounts
- root (local only, SSH disabled)
- labadmin (primary admin, SSH key-based)
- btg-admin (break-glass, disabled by default)

##### SSH Policy
- PasswordAuthentication: disabled
- RootLogin: disabled
- Authentication: ed25519 keys only
- SSH Port: 22 (subject to future VPN restriction)

##### Recovery Procedures
- Console login as root
- IPMI access (credentials stored in password manager)
- Single-user mode enabled via GRUB

##### Audit Notes
- All administrative actions performed via named accounts
- Root usage logged via local console access

> This is DORA / ISO / SOC2 friendly: clear ownership, clear controls, clear recovery.

### 5. What is PAM in Proxmox?

PAM: Pluggable Authentication Modules

PAM = Linux system users (Auth is delegated to /etc/passwd, /etc/shadow, PAM stack)

Other Proxmox auth realms:

- pam → local Linux users

- pve → Proxmox-only users

- ldap, ad, oidc → enterprise auth

| Realm	| Meaning |
| --- | --- |
| labadmin@pam	| Linux user |
| user@pve	| Proxmox-only user |
| user@ldap	| External directory |

### 6. What is pveum?

pveum = Proxmox User Manager
pveum ties Linux identity to Proxmox RBAC.

Authentication vs Authorization
- PAM → authentication (who are you?)
- pveum → authorization (what can you do?)

> PAM user must exist in Linux, Proxmox must know if the user exists + what permissions they have. 

### 7. What is aclmod and what are ACLs?

ACL = Access Control List

ACLs define who (user/group), what (resource path), how (role / permissions)

Example:
```bash
pveum aclmod / -user labadmin@pam -role Administrator
```
Translation:
- Apply to / (entire datacenter)
- User: labadmin
- Permissions: Administrator role

ACL hierarchy
```bash
/
├── nodes/
├── storage/
├── vms/
```

### 8. Difference between Proxmox admin and root?

| Root | Proxmox Admin |
| --- | --- |
| Linux superuser	| Proxmox RBAC role |
| Unrestricted OS access | Scoped to Proxmox objects |
| No audit trail | Fully auditable |
| Breaks everything | Can be limited |

### 9. What is ed25519?

Modern elliptic curve algorithm (faster, smaller keys, safer defaults than RSA)

#### Why not skip the passphrase?

Without a passphrase, steal the key = instant access
Malware reads ~/.ssh/id_ed25519 → done

With a passphrase key theft alone is insufficient, requires user interaction or agent compromise

> This is defense-in-depth.

#### Why not rotate keys often?

Key rotation sounds good, but in practice -> breaks automation, breaks access unexpectedly, adds operational risk

Industry practice:
- Rotate on compromise
- Rotate on role change
- Rotate periodically, but not constantly

> Passphrases reduce the need for aggressive rotation.

### 10. Break-glass accounts a good idea?

Yes. But one break-glass admin, disabled by default, password stored offline, logged usage, reviewed periodically.
No multiple active BTG accounts, that increases attack surface.

### 11. “Easy pivot into every VM”, even without VM creds?

Because Proxmox controls VM disks, VM memory, VM consoles... With Proxmox admin access you can mount disks, reset passwords, inject startup scripts, snapshot memory.

> You don’t need VM credentials if you own the hypervisor.

### 12. What is IPMI?

IPMI = Out-of-band management (Power control, console access, BIOS access, local shell = keyboard + monitor)
- iDRAC (Dell) - Integrated dell remote access controller
- iLO (HP) - Integrated lights-out
- Supermicro IPMI - Intelligent platform management interface

> Out-of-band (OOB) management means that you can manage the system even if the OS is down. it is completely separate from SSH, network config, firewall rules, Linux itself. It is like a power button, keyboard + monitor, BIOS but over the network.

### 13. What is single-user mode?
Linux recovery mode (Minimal services, root shell, no networking)
Used for password resets, filesystem repair, auth recovery.

### 14. Is console access last resort with root?
Yes and that’s by design. Remote convenience is sacrificed for security, control and recovery integrity.

### 15. Why a read-only admin?
Use cases:
- Auditors
- Monitoring systems
- Junior admins learning safely
- CI/CD validation jobs

> Read-only reduces accidental damage or malicious misuse.

### 16. What's the difference between a Linux user vs Proxmox user?

- Linux user → OS-level identity
- Proxmox user → Management-plane identity

When using @pam, they are linked, but permissions are separate, auth happens via PAM, authorization happens via Proxmox RBAC.  
That separation is what makes Proxmox enterprise-grade.

