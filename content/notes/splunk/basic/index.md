---
title: Splunk searches
weight: 210
menu:
  notes:
    name: SPL
    identifier: notes-splunk-searches
    parent: notes-splunk
    weight: 10
---

<!-- Errorcheck within Splunk GUI -->
{{< note title="Troubleshooting: Errorcheck within Splunk GUI" >}}

```bash
index=_internal log_level=ERROR source="/opt/splunk/var/log/splunk/splunkd.log"
```
{{< /note >}}

<!-- Index list -->
{{< note title="Index: List indexes" >}}
```bash
| eventcount summarize=false index=* 
| dedup index 
| fields index
```
{{< /note >}}