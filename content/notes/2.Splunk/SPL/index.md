---
title: Splunk searches
weight: 200
menu:
  notes:
    name: SPL
    identifier: notes-splunk-searches
    parent: notes-splunk
    weight: 20
---

<div style="display: block; width: 100%; max-width: none;">
{{< note >}}
> Most queries are tested on my onprem homelab servers. Sometimes rights might be limited for example in SplunkCloud for "| rest" queries.  
> Using tstats is faster to search in large datapools.
{{< /note >}}
<!-- Troubleshooting:  -->
{{< note title="Troubleshooting:" >}}
When troubleshooting I mostly start with this query:

```bash
index=_internal log_level=ERROR source="/opt/splunk/var/log/splunk/splunkd.log"
```
Errorcheck Splunkd.log within Splunk GUI
{{< /note >}}

<!-- Reporting:  -->
{{< note title="Reporting:" >}}
When checking the health status of the environment, I use the following queries to provide insight:
### General
```bash
| rest splunk_server=* count=1 /services/server/info 
| table version host
```
Check Splunk version 

```bash
| rest /services/server/info 
| eval LastStartupTime=strftime(startup_time, "%Y/%m/%d  %H:%M:%S")
| eval timenow=now()
| eval daysup = round((timenow - startup_time) / 86400,0)
| eval Uptime = tostring(daysup) + " Days"
| table splunk_server LastStartupTime Uptime
```
List server uptime

```bash
| rest /services/authentication/users splunk_server=local
| fields title roles email
| rename title as username
```
List users and their roles
### Logsources
```bash
| tstats values(host) as recent_host where index=* by host, index, sourcetype
| search NOT [| inputlookup monitored_systems.csv | fields host | rename host as recent_host]
| table recent_host, index, sourcetype
```
Overview of non-monitored hosts:
Using a lookupfile to keep track of hosts that we monitor. An alert is used to trigger if a host is not sending logs for a certain threshold.  
This search compares the hosts found that are currently sending logs in the configured timewindow and compares them to the alreadt monitored hosts in the lookupfile.
### Universal forwarders
```bash
index=_internal sourcetype=splunkd group=tcpin_connections version=* os=* arch=* build=* hostname=* source=*metrics.log 
| stats latest(version) as version,latest(arch) as arch,latest(os) as os,latest(build) as build by hostname
```
List universal forwarders and their version

```bash
| tstats values(host) where index=* by index
```
Overview of hosts sending logs by index

```bash
index=_internal source=*metrics.log group=tcpin_connections 
| stats  sum(kb) as total_kb by host, hostname   
| table  hostname, host, total_kb
```
List which hosts are sending through which server/forwarder + volume of data

### Indexes + sourcetypes
```bash
index=_internal source=*license_usage.log type="Usage" idx=*
| timechart span=1d sum(b) as b by st
| foreach * [ eval <<FIELD>> = round('<<FIELD>>'/1024/1024/1024,2) ]
```
Overview of volume per day by sourcetype
```bash
index=_internal source=*license_usage.log type="Usage" earliest=-7d@d latest=@d
| eval _time=strftime(_time, "%Y-%m-%d")
| stats sum(b) as bytes by _time
| eval GB=round(bytes/1024/1024/1024,2)
| table _time GB
| sort - _time
```
total volume per day for past 7 days
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
| rest splunk_server=* /services/data/indexes 
| eval "Retention Period (months)"=round((frozenTimePeriodInSecs/2628000),0)
| search NOT title IN ("_*", "main", "history", "summary", "splunklogger") 
| table title "Retention Period (months)" 
| rename title as Index
```for years, divide frozenTimePeriodInSecs/31556952; for days, divide frozenTimePeriodInSecs/86400```
```
List indexes with the retention period

```bash
| tstats values(sourcetype) as sourcetype WHERE index=* OR index=_* by index 
```
List sourcetypes per index

```bash
| metadata type=sourcetypes index=<index_name>
| table sourcetype
```
List sourcetypes in specific index

```bash
index=_internal source=*license_usage.log type="Usage" idx=<index_name>
| stats sum(b) as b by st
| eval GB=round((b/1024/1024/1024),2)
| eventstats sum(GB) as total_GB
| eval percentage=round((GB/total_GB)*100,2)
| sort -GB
| fields - b, total_GB
| rename st as "Sourcetype", GB as "Size (GB)", percentage as "Percentage (%)"
| fields "Sourcetype", "Size (GB)", "Percentage (%)"
```
List sourcetypes in specific index by volume per sourcetype

```bash
index=eu_cato
| dedup sourcetype
| table _time sourcetype _raw
```
Quick and dirty example log extraction per sourcetype

### Datamodels
```bash
| tstats count FROM datamodel=<datamode_name> BY sourcetype
```
find sourcetypes used in specific datamodel

```bash
| rest /servicesNS/-/-/saved/searches splunk_server=local
| rex field=title "^(standard|custom)\-production\-alert\-(?<SPA>.*)"
| where SPA!=""
| rex field=qualifiedSearch "collect\sindex=(?<into_index>(sla|slad))\s.*"
| eval into_index = coalesce(into_index, 'action.logevent.param.index')
| fillnull value="-" into_index
| search into_index=* eai:acl.app=<app_name>
| eval is_scheduled=if(is_scheduled==1 AND disabled==0,"Active","Not active")
| rex field=description "(?<use_case>^([^.]+))"
| rex field=search "(FROM datamodel=(\")?(?<tstats_datamodel>\w+))"
| rex field=search "(WHERE nodename=\"(\w+\.)?(?<tstats_nodename>\w+))"
| rex field=search "(?<first_row>[^\r\n].*)"
| rex field=first_row "(\`(?<macro>.*?)\`)|(\| from datamodel (?<datamodel>.*))|(index=\"?(?<index>\w+)\"?)|(sourcetype=\"?(?<sourcetype>\w+(:\w+)?)\"?)"
| eval dependency_tmp = case(isnotnull(tstats_datamodel) AND isnotnull(tstats_nodename), tstats_datamodel + "." + tstats_nodename,
    isnotnull(tstats_datamodel) AND isnull(tstats_nodename), tstats_datamodel,
    isnotnull(macro), "`" + macro + "`")
| eval dependency = coalesce(dependency_tmp, datamodel, index, sourcetype)
| eval indicator_type = case(isnotnull(tstats_datamodel) OR isnotnull(tstats_nodename), "tstats",
    isnotnull(macro), "macro",
    isnotnull(datamodel), "datamodel",
    isnotnull(index), "index",
    isnotnull(sourcetype), "sourcetype",
    true(), "other")
| table title, SPA, use_case, indicator_type, dependency, is_scheduled, into_index, cron_schedule,
| rename use_case AS "use case"
| sort + SPA
| fields - SPA
| search dependency IN (<data_model1>, <data_model2>)
```
Find alerts related to datamodels 
> Mind that this is something custom and might not work in your environment. This is more of a sidenote for me to reference to.

### Licenses
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

### Deployment server + apps
```bash
| rest /services/deployment/server/applications
| stats list(title), count(title) by serverclasses
| rename list(title) as apps, count(title) as "amount of apps"
```
List serverclasses and apps

```bash
| rest /services/apps/local 
| search disabled IN ("false",0)
| table title version description splunk_server
```
List all apps

### Parsing + knowledge objects
```bash
| rest /services/data/transforms/extractions 
| table eai:acl.app, title, SOURCE_KEY, REGEX, FORMAT, DEST_KEY 
| sort eai:acl.app title 
| eval DEST_KEY=if(DEST_KEY="","N/A",DEST_KEY) 
| rename eai:acl.app as App, title as Title, SOURCE_KEY as "Source Key", REGEX as RegEx, FORMAT as Format, DEST_KEY as "Dest Key"  
```
List all extractions
```bash
| rest /servicesNS/-/-/admin/directory count=0 splunk_server=local 
| rename eai:* as *, acl.* as * 
| eval updated=strptime(updated,"%Y-%m-%dT%H:%M:%S%Z"), updated=if(isnull(updated),"Never",strftime(updated,"%d %b %Y"))
| sort type 
| stats list(title) as title, list(type) as type, list(orphaned) as orphaned, list(sharing) as sharing, list(owner) as owner, list(updated) as updated by app 
```
List all knowledge objects
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