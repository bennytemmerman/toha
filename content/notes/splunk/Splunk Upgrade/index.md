---
title: Splunk Upgrade procedure
weight: 212
menu:
  notes:
    name: Upgrade
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
Use WinSCP or similar tools to connect to the device:
```bash
set path=C:\Program Files (x86)\WinSCP;%path%
winscp sftp://<CLI-account>@<device_NAT_IP>:/tmp
put "path\to\splunk-<version>-<hash>-Linux-x86_64.tgz" "/tmp/splunk-<version>-<hash>-Linux-x86_64.tgz"
```
Use SCP on more recent windows machines or on Linux:
```bash
scp "path\to\splunk-<version>-<hash>-Linux-x86_64.tgz" user@remote-ip:"/tmp/splunk-<version>-<hash>-Linux-x86_64.tgz"
```
## 3. Pre-Upgrade Checks
Check if Splunk has an active process:
```bash
ps -ef | grep splunk
```
Check if Splunk is running:
```bash
/opt/splunk/bin/splunk status
```
Check Splunk version:
```bash
/opt/splunk/bin/splunk --version
```
## 4. Device State Management
Check whether a monitoring needs to set the host in maintenance mode

On Splunk in environments with index clusters: Maintenance mode halts most bucket fixup activity and prevents frequent rolling of hot buckets. It is useful when performing peer upgrades and other maintenance activities on an indexer cluster.
Enable maintenance mode:
```bash
/opt/splunk/bin/splunk enable maintenance-mode
```
Disable maintenance mode:
```bash
/opt/splunk/bin/splunk disable maintenance-mode
```
Enable maintenance mode:
```bash
/opt/splunk/bin/splunk show maintenance-mode
```

## 5. Backup Current Configuration
Check KV Store status
```bash
/opt/splunk/bin/splunk show kvstore-status
```
Backup KV Store
```bash
/opt/splunk/bin/splunk backup kvstore
```
Check backup
```bash
ls -la /opt/splunk/var/lib/splunk/kvstorebackup/
```
Backup config files
```bash
sudo -i
cd /tmp
mkdir /tmp/etc_backup$(date +"%d-%m-%Y")
cp -a /opt/splunk/etc/ /tmp/etc_backup$(date +"%d-%m-%Y")/
```
Check backup
```bash
ls /tmp/etc_backup$(date +"%d-%m-%Y")/etc
```

## 6. Install the New Version
stop splunk using Splunk command
```bash
/opt/splunk/bin/splunk stop
```
stop splunk using systemctl command
```bash
systemctl stop splunk
```
Unpack the Installer
```bash
tar -xvzf /tmp/splunk-<version>-<hash>-Linux-x86_64.tgz -C /opt/
```
Change the owner of the Splunk folder recursively with the account running Splunk
```bash
chown -R splunk.splunk /opt/splunk
```
Start Splunk with License Acceptance
```bash
/opt/splunk/bin/splunk start --accept-license --answer-yes
```
## 7. Post-Upgrade Verification
Check if Splunk is running
```bash
/opt/splunk/bin/splunk status
```
Check recent logs for errors
```bash
tail -20f /opt/splunk/var/log/splunk/splunkd.log
```
check Splunk version
```bash
/opt/splunk/bin/splunk --version
```
## 8. Documentation and Communication
- Update internal documentation with the new version details.
- Notify customer and colleagues of the successful upgrade.
- Share release notes and any post-upgrade instructions.

{{< /note >}}

</div>