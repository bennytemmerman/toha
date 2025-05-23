---
title: Splunk CLI commands
weight: 211
menu:
  notes:
    name: CLI
    identifier: notes-splunk-CLI
    parent: notes-splunk
    weight: 11
---

<div style="display: block; width: 100%; max-width: none;">

<!-- Maintenance: -->
{{< note title="Maintenance:" >}}
Start Splunk
```bash
/opt/splunk/bin/splunk start
```
Stop Splunk
```bash
/opt/splunk/bin/splunk stop
```
Restart Splunk
```bash
/opt/splunk/bin/splunk restart
```
Check if Splunk is running
```bash
/opt/splunk/bin/splunk status
```
Reload a serverclass to push deployment apps without restarting Splunk"
```bash
/opt/splunk/bin/splunk reload deploy-server -class [serverclass-name]
```
Check license expiration date
```bash
/opt/splunk/bin/splunk list licenses | grep "expiration_time" | awk -F':' '{print $2}' | xargs -I{} date -d @{} +"%Y-%m-%d %H:%M:%S"
```
List installed apps and their status
```bash
/opt/splunk/bin/splunk list app
```
List installed apps and their version (if found)
```bash
/opt/splunk/bin/splunk list app | grep version /opt/splunk/etc/apps/*/default/app.conf
```
Install an app
```bash
/splunk install app <path to app.package>
```
Update an app
```bash
/splunk install app <path to app.package> -update 1
```
Remove an app
```bash
/opt/splunk/bin/splunk remove app [appname]
```
Check Splunk admins
```bash
/opt/splunk/bin/splunk list user | grep admin -B2
```
{{< /note >}}
<!-- Basic config: -->
{{< note title="Basic config:" >}}
Enable Splunk service to start when the host boots up
```bash
/opt/splunk/bin/splunk enable
```
Disable Splunk service so it doesn't start when the host boots up
```bash
/opt/splunk/bin/splunk disable
```
{{< /note >}}
<!-- Extra: -->
{{< note title="Extra:" >}}
Find the startup message
```bash
cat /opt/splunk/var/log/splunk/splunkd_stdout.log | grep "Splunk>" | tail -n 1
```
{{< /note >}}

</div>