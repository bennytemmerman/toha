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

<div style="display: block; width: 100%; max-width: none;">

<!-- Troubleshooting:  -->
{{< note title="Troubleshooting:" >}}
Errorcheck within Splunk GUI
```bash
index=_internal log_level=ERROR source="/opt/splunk/var/log/splunk/splunkd.log"
```
Check Splunk version
```SPL
| rest splunk_server=* count=1 /services/server/info 
| table version host
```
{{< /note >}}

<!-- Index -->
{{< note title="List indexes with the retention period" >}}
```bash
| rest splunk_server=* /services/data/indexes 
| eval "Retention Period (months)"=round((frozenTimePeriodInSecs/2628000),0)
| search NOT title IN ("_*", "main", "history", "summary", "splunklogger") 
| table title "Retention Period (months)" 
| rename title as Index
```for years, divide frozenTimePeriodInSecs/31556952; for days, divide frozenTimePeriodInSecs/86400```
```
{{< /note >}}

<!-- Hosts -->
{{< note title="Overview of hosts sending logs by index" >}}
```bash
| tstats values(host) where index=* by index
```
{{< /note >}}

<!-- License -->
{{< note title="Usage per day by index + percentages" >}}
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
{{< /note >}}

<!-- Users -->
{{< note title="List users and their roles" >}}
```bash
| rest /services/authentication/users splunk_server=local
| fields title roles email
| rename title as username
```
{{< /note >}}

</div>