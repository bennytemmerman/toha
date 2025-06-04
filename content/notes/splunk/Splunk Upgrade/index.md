---
title: Splunk Upgrade procedure
weight: 212
menu:
  notes:
    name: CLI
    identifier: notes-splunk-Upgrade
    parent: notes-splunk
    weight: 12
---

<div style="display: block; width: 100%; max-width: none;">

<!-- Upgrade procedure: -->
{{< note title="Upgrade procedure:" >}}
# Splunk Upgrade Guide
This document provides a step-by-step procedure to upgrade Splunk from an existing version to a newer release. Follow each section carefully to ensure a smooth upgrade process.
## Table of Contents
1. Preparation
2. Download and Transfer Files
3. Pre-Upgrade Checks
4. Device State Management
5. Backup Current Configuration
6. Install the New Version
7. Post-Upgrade Verification
8. Documentation and Communication
9. Additional Resources
## 1. Preparation
- Ensure you have access to the target devices and necessary permissions.
- Confirm the current Splunk version.
- Notify customer and colleagues about the upcoming upgrade.
- Review release notes for the target version.
## 2. Download and Transfer Files
### Download the Splunk Installer
```bash
sudo -i
cd /tmp
wget -O splunk-<version>-<hash>-Linux-x86_64.tgz "https://download.splunk.com/products/splunk/releases/<version>/linux/splunk-<version>-<hash>-Linux-x86_64.tgz"
```
### Transfer the Installer to Target Devices


{{< /note >}}

</div>