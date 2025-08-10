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
> Most queries are tested on my onprem homelab servers. Sometimes rights might be limited for example in SplunkCloud for "| rest" queries.  
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
index=_internal sourcetype=splunkd group=tcpin_connections version=* os=* arch=* build=* hostname=* source=*metrics.log 
| stats latest(version) as version,latest(arch) as arch,latest(os) as os,latest(build) as build by hostname
```
List universal forwarders and their version

```bash
| rest /services/server/info | eval LastStartupTime=strftime(startup_time, "%Y/%m/%d  %H:%M:%S")
| eval timenow=now()
| eval daysup = round((timenow - startup_time) / 86400,0)
| eval Uptime = tostring(daysup) + " Days"
| table splunk_server LastStartupTime Uptime
```
List server uptime

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
index=_internal source=*metrics.log group=tcpin_connections | stats  sum(kb) as total_kb by host, hostname   | table  hostname, host, total_kb
```
List which hosts are sending through which server/forwarder + volume of data

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

```bash
| rest splunk_server=local "/services/licenser/licenses"
| eval creation_time=strftime(creation_time,"%d-%m-%Y"), days_until_expiration=round((expiration_time-now())/86400) , expiration=strftime(expiration_time,"%d-%m-%Y") ,quota = ('quota'/1024/1024/1024), Volume = quota+" GB", is_unlimited =if('is_unlimited'==0,"no","yes")
| fields  creation_time days_until_expiration expiration expiration_time Volume stack_id status subgroup_id type window_period max_violations label is_unlimited group_id features eai:acl.perms.write eai:acl.perms.read quota
| eval renewal_period = case(days_until_expiration<14, "critical", days_until_expiration<30, "warning", 1=1, "Ok")
| eval quota1 = case(quota>1000, quota/1024) , quota1 = quota1+" TB"
| eval quota2 = case(quota<1, quota*1024) , quota2 = quota2+" MB"
| eval Volume = coalesce(quota1,quota2,Volume)
| fields group_id label Volume creation_time expiration renewal_period days_until_expiration is_unlimited   
| rename creation_time AS "Purchase", expiration AS "Expiration", renewal_period AS "Renewal", days_until_expiration AS "Remaining Days", is_unlimited AS Unlimited,  group_id AS "License Type" , features AS "Available Features" , label AS License
```
List information about Licenses, including expiration dates.

```bash
| rest /services/deployment/server/applications
| stats list(title), count(title) by serverclasses
| rename list(title) as apps, count(title) as "amount of apps"
```
List serverclasses and apps

```bash
| rest /services/apps/local | search disabled IN ("false",0)| table title version description splunk_server
```
List all apps

```bash
| rest /services/data/transforms/extractions | table eai:acl.app, title, SOURCE_KEY, REGEX, FORMAT, DEST_KEY | sort eai:acl.app title | eval DEST_KEY=if(DEST_KEY="","N/A",DEST_KEY) | rename eai:acl.app as App, title as Title, SOURCE_KEY as "Source Key", REGEX as RegEx, FORMAT as Format, DEST_KEY as "Dest Key"  
```
List all extractions

```bash
| rest /servicesNS/-/-/admin/directory count=0 splunk_server=local | rename eai:* as *, acl.* as * | eval updated=strptime(updated,"%Y-%m-%dT%H:%M:%S%Z"), updated=if(isnull(updated),"Never",strftime(updated,"%d %b %Y"))| sort type | stats list(title) as title, list(type) as type, list(orphaned) as orphaned, list(sharing) as sharing, list(owner) as owner, list(updated) as updated by app 
```
List all knowledgeobjects

```bash
| tstats values(sourcetype) as sourcetype WHERE index=* OR index=_* by index 
```
List sourcetypes per index
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