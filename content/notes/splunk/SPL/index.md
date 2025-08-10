---
title: Splunk searches
weight: 300
menu:
  notes:
    name: SPL
    identifier: notes-splunk-searches
    parent: notes-splunk
    weight: 30
---

<div style="display: block; width: 100%; max-width: none;">

<!-- Troubleshooting:  -->
{{< note title="Troubleshooting:" >}}
When troubleshooting I mostly start with this query:
```bash
index=_internal log_level=ERROR source="/opt/splunk/var/log/splunk/splunkd.log"
```
Errorcheck within Splunk GUI
{{< /note >}}

<!-- Reporting:  -->
{{< note title="Reporting:" >}}
When checking the health status of the environment, I use the following queries to provide insight:
```bash
| rest splunk_server=* count=1 /services/server/info 
| table version host
```
Check Splunk version 
```bash
index=_internal source=*license_usage.log type="Usage"
| bin _time span=1d
| stats sum(b) as b by idx, _time
| eval GB=round((b/31556952),2)
| eventstats sum(GB) as total_GB by _time
| eval percentage=round((GB/total_GB)*100,2)
| sort -_time -GB
| fields - b, total_GB
| rename idx as Index, GB as "Size (GB)", percentage as "Percentage (%)"
| convert timeformat="%Y-%m-%d" ctime(_time) as Date
| fields Date, Index, "Size (GB)", "Percentage (%)"
```for years, divide b/31556952; for days, divide b/31556952```
```
Usage per day by index + percentages
```bash
| tstats values(host) where index=* by index
```
Overview of hosts sending logs by index
```bash
| rest /services/authentication/users splunk_server=local
| fields title roles email
| rename title as username
```
List users and their roles
```bash
| rest splunk_server=* /services/data/indexes 
| eval "Retention Period (months)"=round((frozenTimePeriodInSecs/2628000),0)
| search NOT title IN ("_*", "main", "history", "summary", "splunklogger") 
| table title "Retention Period (months)" 
| rename title as Index
```for years, divide frozenTimePeriodInSecs/31556952; for days, divide frozenTimePeriodInSecs/86400```
```
List indexes with the retention period
{{< /note >}}

<!-- Maintenance  -->
{{< note title="Maintenance:" >}}
Using the following query, it is often used after a Splunk version or app/addon upgrade to compare the amount of logs.  
Keep in mind that it is using stats, so big pools of data might slow the search down.
```bash
index=* earliest=-1mon@m-15m latest=-1mon@m
| stats min(_time) as _time count as Count by sourcetype, index
| eval Day="Last month" 
| fields Day, _time, Count, sourcetype, index
| append [ search index=* earliest=-1w@m-15m latest=-1w@m 
    | stats min(_time) as _time count as Count by sourcetype, index
    | eval Day="Last week" 
    | fields Day, _time, Count, sourcetype, index] 
| append [ search index=* earliest=-15m@m latest=@m
    | stats min(_time) as _time count as Count by sourcetype, index
    | eval Day="Today" 
    | fields Day, _time, Count, sourcetype, index]     
| sort sourcetype
```
Compare amount of events per sourcetype last 15 minutes with same 15 minutes last week and last month.
{{< /note >}}
</div>