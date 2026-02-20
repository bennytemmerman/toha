---
title: "OpenTofu Project: Part 2 - VM lifecycle & OS baselines"
date: 2026-02-14T10:17:23+02:00
hero: /images/posts/opentofu.png
description: A multiphase project to learn about OpenTofu, an orchestrator to automate and enhance deployments.
theme: Toha
draft: false
menu:
  sidebar:
    name: OpenTofu 2. VM lifecycle & OS baselines
    identifier: opentofu2
    parent: cat-orchestration
    weight: 602
---

# Building secure VM baselines

## Why manual VM builds matter

Infrastructure as Code is only as good as the images it deploys. Automating insecure or poorly understood baselines simply scales risk.
Before introducing OpenTofu or CI/CD pipelines, it is critical to manually build and understand virtual machine baselines.

## Operating systems used

| # | OS | vCPU | RAM | Disk | Notes |
| --- | --- | --- | --- | --- | --- |
| 1 | Ubuntu Server | 2 | 2GB | Lightweight |
| 2 | Rocky Linux | 2 | 2GB | SELinux active |
| 3 | Windows 11 | 2 | 4GB | GUI environment |
| 4 | Windows Server 2022 | 2 | 4GB | DC, Fileserver |

### 1. Ubuntu server baseline
Required installation components:
- OpenSSH server
- Minimal install
- Non-root admin user

Post-install checklist
- [] Update system
```bash
sudo apt update && sudo apt upgrade -y
```
> Document date of patch - kernel version

- [] Install core admin utilities
```bash
sudo apt install -y curl wget vim net-tools htop
```
Why: Operational debugging, network inspection, log review

- [] Logging verification
```bash
journalctl -xe
```
Ensure no obvious boot errors and SSH logs are present

- [] Time synchronization
```bash
timedatectl status
```
Time drift kills logs, Kerberos, forensics

### 2. Rocky Linux baseline
Required validation:
- [] SELinux
```bash
getenforce
```
Should be "Enforcing"
> Document policy mode

- [] Firewall
```bash
firewall-cmd --list-all
```
> Document allowed services.

- [] Sudo config
```bash
visudo
```
Verify wheel group controls privilege escalation

### 3. Windows 11 baseline

Required steps

- [] Local admin created
- [] Windows updated
- [] Defender status verified
- [] Firewall enabled

Security Baseline Checks
```bash
secpol.msc
```
Review: Password policy, audit policy, user rights assignment
> Document settings

### 4. Windows Server 2022 Baseline
Required Setup
- [] Static IP
- [] Rename computer
- [] Disable IE Enhanced Security (lab only)
- [] Windows updates installed

Security Checks
- [] SMBv1 disabled
- [] Event logs accessible
- [] Defender enabled

## Template creation
Checklist before converting to template:
- [] All updates applied
- [] No ISOs mounted
- [] No temporary files
- [] Clear bash history (optional but good hygiene)

Linux:
```bash
history -c
```
Then shutdown VM + convert to template

## Documentation
### Baseline Report (per OS)
Example structure:
```bash
ubuntu-22.04-baseline.md
rocky-9-baseline.md
win11-baseline.md
win2022-baseline.md
```
Include:
- Installed packages
- Security defaults
- Intentional misconfigs
- Patch date
- Snapshot ID
- Template name

### Risk Register Entry

| Risk | Impact | Mitigation |
| --- | --- | --- |
| SSH password enabled | Brute force | Disable before creating template |

> This is how compliance frameworks think.

### Recovery Procedure

Document:
- How to restore from snapshot
- How to re-clone template
- How to full rebuild
> This proves: Operational maturity, disaster readiness

## Observing default security posture

Manual installation reveals important differences:

- Ubuntu enables password-based SSH by default
- Rocky Linux enforces SELinux and firewalld
- Windows systems default to overly permissive local admin access

Understanding these defaults is essential for effective hardening.

## Snapshots vs templates

- Snapshots are short-lived and ideal for experimentation
- Templates are immutable golden images used for consistent deployments

Templates ensure reproducibility and reduce configuration drift.

## Conclusion

Manual VM lifecycle management builds intuition that automation cannot replace. By understanding how systems are installed, configured, and broken, engineers can later automate with confidence.
