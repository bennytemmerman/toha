---
title: Kusto query language
weight: 300
menu:
  notes:
    name: KQL
    identifier: notes-kql
    parent: notes-sentinel
    weight: 30
---

<div style="display: block; width: 100%; max-width: none;">
{{< note >}}
### Troubleshooting
A collection of KQL I use to troubleshoot incidents in Sentinel.
```bash
SentinelHealth
| where SentinelResourceName in ("Successful logon from IP and failure from a different IP")
| where Status == "Failure"
| project TimeGenerated, SentinelResourceName, Status
| sort by TimeGenerated desc
```
__SentinelHealth__
Selects data from the SentinelHealth table.  
__| where SentinelResourceName in ("Successful logon from IP and failure from a different IP")__
Filters records to include only those with specific resource names related to logon success and failure.  
__| where Status == "Failure"__
Further filters the data to include only failed login attempts.  
__| project TimeGenerated, SentinelResourceName, Status__
Selects only the columns for the time of the event, resource name, and status.  
__| sort by TimeGenerated desc__
Orders the results by the most recent events first.
{{< /note >}}
</div>