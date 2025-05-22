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

<!-- Maintenance: Start Splunk -->
{{< note title="Maintenance: Start Splunk" >}}

```bash
/opt/splunk/bin/splunk start
```
{{< /note >}}

<!-- Maintenance: Stop Splunk -->
{{< note title="Maintenance: Stop Splunk" >}}

```bash
/opt/splunk/bin/splunk stop
```
{{< /note >}}

<!-- Maintenance: restart Splunk -->
{{< note title="Maintenance: Restart Splunk" >}}

```bash
/opt/splunk/bin/splunk restart
```
{{< /note >}}

<!-- Maintenance: check Splunk status -->
{{< note title="Maintenance: Check if Splunk is running" >}}

```bash
/opt/splunk/bin/splunk status
```
{{< /note >}}

<!-- Maintenance: Reload serverclass -->
{{< note title="Maintenance: reloading a serverclass to push deployment apps without restarting Splunk" >}}

```bash
/opt/splunk/bin/splunk reload deploy-server -class [serverclass-name]
```
{{< /note >}}

<!-- Maintenance: Check License expiration date -->
{{< note title="Maintenance: Check license expiration date" >}}

```bash
sudo /opt/splunk/bin/splunk list licenses | grep "expiration_time" | awk -F':' '{print $2}' | xargs -I{} date -d @{} +"%Y-%m-%d %H:%M:%S"
```
{{< /note >}}

<!-- Maintenance: List apps -->
{{< note title="Maintenance: List installed apps and their status" >}}

```bash
sudo /opt/splunk/bin/splunk list app
```
{{< /note >}}

<!-- Maintenance: List app versions -->
{{< note title="Maintenance: List installed apps and their version (if found)" >}}

```bash
sudo /opt/splunk/bin/splunk list app | grep version /opt/splunk/etc/apps/*/default/app.conf
```
{{< /note >}}

<!-- Maintenance: Install an app -->
{{< note title="Maintenance: Update an app" >}}

```bash
/splunk install app <path to app.package>
```
{{< /note >}}

<!-- Maintenance: Update an app -->
{{< note title="Maintenance: Update an app" >}}

```bash
/splunk install app <path to app.package> -update 1
```
{{< /note >}}

<!-- Maintenance: Remove an app -->
{{< note title="Maintenance: Remove an app" >}}

```bash
/opt/splunk/bin/splunk remove app [appname]
```
{{< /note >}}

<!-- Maintenance: Check Splunk admins -->
{{< note title="Maintenance: Check Splunk admins" >}}

```bash
/opt/splunk/bin/splunk list user | grep admin -B2
```
{{< /note >}}

<!-- Basic config: Enable Splunk service -->
{{< note title="Basic config: Enable Splunk service to start when the host boots up" >}}

```bash
/opt/splunk/bin/splunk enable
```
{{< /note >}}

<!-- Basic config: Disable Splunk service -->
{{< note title="Basic config: Disable Splunk service so it doesn't start when the host boots up" >}}

```bash
/opt/splunk/bin/splunk disable
```
{{< /note >}}

<!-- Extra: Find the startup message -->
{{< note title="Extra: Find the startup message" >}}

```bash
cat /opt/splunk/var/log/splunk/splunkd_stdout.log | grep "Splunk>" | tail -n 1
```
{{< /note >}}

</div>