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
{{< note title="Troubleshooting:" >}}

# Troubleshooting
A collection of KQL I use to troubleshoot incidents in Sentinel.

## Failed analytics rule

```bash
SentinelHealth
| where SentinelResourceName startswith "AnalyticsRule -"
| where OperationName == "Scheduled analytics rule run"
| where Status == "Failure"
| summarize count(Status) by SentinelResourceName
| where count_Status > 3
| extend info_name = "failed_analytic-rule"
| extend info_sub_route = "Sentinel"
```
Detects rules that have failed more than three times, indicating potential issues with rule execution.

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
union withsource=TableName
vcenter_CL, SecurityEvent, CommonSecurityLog, Syslog // Add/remove tables as needed
| where TimeGenerated >= ago(7d)
| summarize MinutesSinceLastEvent = datetime_diff("minute", now(), max(TimeGenerated)), LastIngestionTime = max(TimeGenerated) by TableName
| where MinutesSinceLastEvent >= 60
| project TableName, alert_minutes_last_event = MinutesSinceLastEvent, LastIngestionTime
| extend info_name = "table_not_receiving-data"
| extend info_sub_route = "Sentinel"
```
Identifies tables that haven't received data for over an hour, signaling possible ingestion problems.

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

## Silent logsource
```bash
 let _SecurityEvent = 
   SecurityEvent
   | where TimeGenerated > ago(15d)
   | summarize
       MinutesSinceLastEvent = datetime_diff("minute", now(), max(TimeGenerated)),
       LastIngestionTime = max(TimeGenerated)
       by
       Asset = Computer,
       Table = Type,
       sourcetype = EventSourceName
 ;
 let _Syslog = 
   Syslog
   | where TimeGenerated > ago(15d)
   | summarize
       MinutesSinceLastEvent = datetime_diff("minute", now(), max(TimeGenerated)),
       LastIngestionTime = max(TimeGenerated)
       by 
       Asset = Computer, 
       Table = Type
       //sourcetype = strcat(
       //         iff(DeviceVendor == "Fortinet", DeviceEventCategory, ""),
       //         iff(DeviceVendor == "Palo Alto Networks", Activity, ""),
       //         iff(DeviceVendor == "F5", DeviceProduct, "")
       //     )
 ;
 let _Heartbeat = 
   Heartbeat
   | where TimeGenerated > ago(15d)
   | summarize
       MinutesSinceLastEvent = datetime_diff("minute", now(), max(TimeGenerated)),
       LastIngestionTime = max(TimeGenerated)
       by
       Asset = Computer,
       Table = Type,
       sourcetype = Type
 ;
 union _SecurityEvent, _Syslog, _Heartbeat
 | join _GetWatchlist('CriticalLogs')
   on
   $left.Asset == $right.Host,
   $left.sourcetype == $right.SubType,
   $left.Table == $right.DataTable
 | where Maintenance != "TRUE"
 | where MinutesSinceLastEvent > toint(Threshold) * 60 // Watchlist in hours need to convert to min
 | project Table, Asset, MinutesSinceLastEvent, sourcetype, SubType, Threshold, LastIngestionTime
 | extend info_name = "critical_log_source_goes-silent"
 | extend info_sub_route = "Sentinel"
```
Finds critical logs from assets that haven't reported in over a specified threshold, suggesting potential issues or outages.

```bash
SecurityEvent 
| where Computer contains ("test-pc")
| summarize Type = count() by Computer, bin(TimeGenerated, 1h) 
| render timechart
```
This KQL query filters security events to include only those from computers with "test-pc" in their name, then counts the number of events for each computer grouped by 1-hour intervals, and finally displays the results as a time chart. I use this to check for patterns in log ingestion which can indicate ingestion issues or expected behavior.
{{< /note >}}

{{< note title="Reporting:" >}}

# Reporting
```bash
Usage
| where IsBillable == true
| where QuantityUnit == "MBytes"
| summarize totalMB = sum(Quantity) by day = bin(TimeGenerated, 1d)
| summarize avgGBperDay = round(avg(totalMB / 1024), 2)
```
A Sentinel KQL query to extract the average daily volume, adjust time range as needed.

```bash
Usage
| where IsBillable == true
| where QuantityUnit == "MBytes"
| summarize totalMB = sum(Quantity) by day = bin(TimeGenerated, 1d)
| extend GB = round(totalMB / 1024, 2)
| order by day asc
```
A Sentinel KQL query aggregated the total number of bytes ingested per day.

{{< /note >}}
</div>