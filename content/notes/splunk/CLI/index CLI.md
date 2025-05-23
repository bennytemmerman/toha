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
{{< note title="Start Splunk" >}}

```bash
/opt/splunk/bin/splunk start
```
{{< /note >}}

{{< note title="Stop Splunk" >}}

```bash
/opt/splunk/bin/splunk stop
```
{{< /note >}}

{{< note title="Restart Splunk" >}}

```bash
/opt/splunk/bin/splunk restart
```
{{< /note >}}

{{< note title="Check if Splunk is running" >}}

```bash
/opt/splunk/bin/splunk status
```
{{< /note >}}

{{< note title="reloading a serverclass to push deployment apps without restarting Splunk" >}}

```bash
/opt/splunk/bin/splunk reload deploy-server -class [serverclass-name]
```
{{< /note >}}

{{< note title="Check license expiration date" >}}

```bash
/opt/splunk/bin/splunk list licenses | grep "expiration_time" | awk -F':' '{print $2}' | xargs -I{} date -d @{} +"%Y-%m-%d %H:%M:%S"
```
{{< /note >}}

{{< note title="List installed apps and their status" >}}

```bash
/opt/splunk/bin/splunk list app
```
{{< /note >}}

{{< note title="List installed apps and their version (if found)" >}}

```bash
/opt/splunk/bin/splunk list app | grep version /opt/splunk/etc/apps/*/default/app.conf
```
{{< /note >}}

{{< note title="Install an app" >}}

```bash
/splunk install app <path to app.package>
```
{{< /note >}}

{{< note title="Update an app" >}}

```bash
/splunk install app <path to app.package> -update 1
```
{{< /note >}}

{{< note title="Remove an app" >}}

```bash
/opt/splunk/bin/splunk remove app [appname]
```
{{< /note >}}

{{< note title="Check Splunk admins" >}}

```bash
/opt/splunk/bin/splunk list user | grep admin -B2
```
{{< /note >}}

<!-- Basic config: -->
{{< note title="Enable Splunk service to start when the host boots up" >}}

```bash
/opt/splunk/bin/splunk enable
```
{{< /note >}}

{{< note title="Disable Splunk service so it doesn't start when the host boots up" >}}

```bash
/opt/splunk/bin/splunk disable
```
{{< /note >}}

<!-- Extra: -->
{{< note title="Find the startup message" >}}

```bash
cat /opt/splunk/var/log/splunk/splunkd_stdout.log | grep "Splunk>" | tail -n 1
```
{{< /note >}}

</div>