---
title: "OpenTofu Project: VM Lifecycle & OS Baselines"
date: 2026-02-14T10:17:23+02:00
hero: /images/posts/opentofu.png
description: A multiphase project to learn about OpenTofu, an orchestrator to automate and enhance deployments.
theme: Toha
draft: true
menu:
  sidebar:
    name: OpenTofu 2. VM Lifecycle & OS Baselines
    identifier: opentofu2
    parent: cat-orchestration
    weight: 602
---

# Building secure VM baselines in Proxmox: Manual before automated

## Why manual VM builds matter

Infrastructure as Code is only as good as the images it deploys. Automating insecure or poorly understood baselines simply scales risk.
Before introducing OpenTofu or CI/CD pipelines, it is critical to manually build and understand virtual machine baselines.



## Operating systems used

This lab includes four operating systems, each chosen for specific security learning goals:

- Ubuntu Server: DevOps-friendly, permissive defaults
- Rocky Linux: Enterprise-focused, secure-by-default
- Windows 11: Client-side attack surface
- Windows Server 2022: Enterprise service host

Each OS exposes different risks and hardening challenges.



## Proxmox VM configuration best practices

Recommended baseline settings:

- OVMF (UEFI) firmware
- q35 machine type
- VirtIO disk and network devices
- Memory ballooning enabled

These settings improve performance, compatibility, and security posture.



## Observing default security posture

Manual installation reveals important differences:

- Ubuntu enables password-based SSH by default
- Rocky Linux enforces SELinux and firewalld
- Windows systems default to overly permissive local admin access

Understanding these defaults is essential for effective hardening.



## Intentional insecurity as a learning tool

Each baseline includes documented misconfigurations, such as:

- Weak SSH policies on Linux
- Default Administrator accounts on Windows
- Lack of auditing and credential protections

These weaknesses are intentional and provide material for later red-team and blue-team exercises.



## Snapshots vs templates

- Snapshots are short-lived and ideal for experimentation
- Templates are immutable golden images used for consistent deployments

Templates ensure reproducibility and reduce configuration drift.



## Conclusion

Manual VM lifecycle management builds intuition that automation cannot replace. By understanding how systems are installed, configured, and broken, engineers can later automate with confidence.
