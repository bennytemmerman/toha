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
# Troubleshooting
A collection of KQL I use to troubleshoot incidents in Sentinel.
## Failed analytics rule
```bash
SentinelHealth
| where SentinelResourceName in ("Successful logon from IP and failure from a different IP")
| where Status == "Failure"
| project TimeGenerated, SentinelResourceName, Status
| sort by TimeGenerated desc
```
Finding the failed runs of an analytics rule can help narrow down the time window to investigate further.
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

```bash
SentinelHealth
| where SentinelResourceName in ("Successful logon from IP and failure from a different IP")
| project TimeGenerated, SentinelResourceName, Status
| sort by TimeGenerated desc
```
Expand the failed run time window to investigate warnings or successful runs. It is possible that Sentinel runs out of resources where a failed run can be expected and closed as true benign positive, expected behavior. Multiple failed runs might indicate another issue with the analytics rule (Syntax related or other) after an update of the rule. Further investigation is needed to resolve the issue.

## Table does not receive data
```bash
SecurityAlert
| summarize EventCount = count() by bin(TimeGenerated, 1h)
| render timechart
```
This query counts the amount of events per hour and renders it in a timechart. This is an easy way to pinpoint anomalies in log event ingestion.  
```bash
SecurityAlert
| where TimeGenerated >= ago(30d)
| sort by TimeGenerated asc
| serialize
| extend TimeDiff = datetime_diff('second', TimeGenerated, prev(TimeGenerated))
| where TimeDiff >= 3600
| project TimeGenerated, prev_TimeGenerated=prev(TimeGenerated), TimeDiff
```
Statistical overview of gaps (over 1 hour) between events for the last 30 days. This one is mostly used to determine tuning for the threshold used in an analytic rule to check if a table did not receive data. (could indicate a logsource stopped sending or a misconfiguration in sending the logs to another table)
{{< /note >}}
</div>