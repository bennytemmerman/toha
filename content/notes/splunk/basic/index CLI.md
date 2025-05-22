---
title: Splunk CLI commands
weight: 210
menu:
  notes:
    name: CLI
    identifier: notes-splunk-CLI
    parent: notes-splunk
    weight: 10
---

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