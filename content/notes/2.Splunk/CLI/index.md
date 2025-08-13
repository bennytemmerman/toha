---
title: Splunk CLI commands
weight: 202
menu:
  notes:
    name: CLI
    identifier: notes-splunk-CLI
    parent: notes-splunk
    weight: 22
---

<div style="display: block; width: 100%; max-width: none;">

<!-- CLI commands: -->
{{< note title="CLI commands:" >}}
These are some commands to use in CLI, while a gui is available, I like to do most things using the command line interface.
### Basic operations
```bash
/opt/splunk/bin/splunk start
```
Start Splunk
```bash
/opt/splunk/bin/splunk stop
```
Stop Splunk
```bash
/opt/splunk/bin/splunk restart
```
Restart Splunk
```bash
/opt/splunk/bin/splunk status
```
Check if Splunk is running

### General checks
```bash
/opt/splunk/bin/splunk list user | grep admin -B2
```
Check Splunk admins

### Splunk service setup
```bash
/opt/splunk/bin/splunk enable
```
Enable Splunk service to start when the host boots up
```bash
/opt/splunk/bin/splunk disable
```
Disable Splunk service so it doesn't start when the host boots up

### Deployment server
```bash
/opt/splunk/bin/splunk reload deploy-server -class [serverclass-name]
```
Reload a serverclass to push deployment apps without restarting Splunk"

### Licenses
```bash
/opt/splunk/bin/splunk list licenses | grep "expiration_time" | awk -F':' '{print $2}' | xargs -I{} date -d @{} +"%Y-%m-%d %H:%M:%S"
```
Check license expiration date

### Apps/addons
```bash
/opt/splunk/bin/splunk list app
```
List installed apps and their status
```bash
/opt/splunk/bin/splunk list app | grep version /opt/splunk/etc/apps/*/default/app.conf
```
List installed apps and their version (if found)
```bash
/splunk install app <path to app.package>
```
Install an app
```bash
/splunk install app <path to app.package> -update 1
```
Update an app
```bash
/opt/splunk/bin/splunk remove app [appname]
```
Remove an app

### Extra
```bash
cat /opt/splunk/var/log/splunk/splunkd_stdout.log | grep "Splunk>" | tail -n 1
```
Find the startup message
{{< /note >}}
</div>