---
title: "Tech report: Cato Networks log ingestion"
date: 2026-02-02T17:17:23+02:00
#hero: /images/posts/logrotate.svg
hero: /images/posts/cato.jpg
description: Cato Networks log ingestion interruption
theme: Toha
menu:
  sidebar:
    name: TR_CatoNetworks
    identifier: cato
    parent: cat-siem
    weight: 502
---
# Cato Networks Log Ingestion Interruption

## Executive summary

Cato Networks logs stopped ingesting into Splunk following a customer-initiated unannounced maintenance. Investigation revealed that the API connection for log retrieval was interrupted, requiring manual intervention to restore service. The incident highlighted the impact of uncoordinated maintenance activities on log ingestion stability. Immediate checks were performed on the Heavy Forwarder logs, and the scripted input was reset to restore log flow. No other log sources were found to be affected at this time.

**Data loss:** No Cato Networks logs were ingested from maintenance until manual intervention, creating a monitoring gap. Logs were ingested using backlog. Only loss was visibility during the halted ingestion.
**Manual recovery:** Log ingestion was restored by restarting the scripted input, but the root cause was traced to uncoordinated system maintenance.
**Lesson learned:** Lack of communication regarding planned maintenance led to repeated manual fixes.

## 1. Background

Cato Networks logs are ingested into Splunk via a scripted input (python script EventFeed.py) on the Splunk Heavy Forwarder. After customer maintenance, log ingestion stopped, no new logs arrived in Splunk. We received an incident ticket about a missing logsource. The objective was to restore log ingestion, determine the root cause, and identify process improvements to prevent recurrence.

## 2. Incident timeline

Last log ingested: 2026-01-29T02:42:23Z
Repeated failures: Similar interruptions occurred several times in the preceding days.
Logs stopped ingesting around the same time (3am) but no steady pattern for past 4 days.
  
  
> Temporary fix: Disabling and re-enabling the scripted input which restored log flow.

## 3. Troubleshooting steps

### 3.1. Log review

Checked Splunk internal logs:
```bash
index=_internal log_level=ERROR source="/opt/splunk/var/log/splunk/splunkd.log" cato
```
Searched for errors related to Cato log ingestion and the scripted input, indicating API connection failures.
```bash
ConnectionResetError: [Errno 104] Connection reset by peer errors
```
Checked system logs:
```bash
journalctl | grep cato
```
  
> Used journalctl and Splunk’s internal logs to confirm no other log sources were affected.

### 3.2. Scripted input management

Splunk GUI: Disabled and re-enabled $SPLUNK_HOME/etc/apps/Splunk_Cato_TA/bin/script/cato.sh  
Log ingestion resumed.

### 3.3. Root cause

API connection interrupted: The API call for Cato Networks logs was interrupted by the maintenance, causing the script to fail.  
No notification: The SIEM team was not informed of the planned maintenance, leading to manual recovery.

## 4. Impact assessment

**Monitoring gap:** No Cato Recycling logs were ingested for an extended period, reducing visibility for security monitoring.  
**Operational Overhead:** Manual intervention was required to restore service, increasing workload and response time.  
**Other Log Sources:** No evidence of issues with other log sources at this time, but continued monitoring is recommended.

## 5. Lessons learned

**Importance of communication:** Unannounced maintenance and upgrades can disrupt critical log ingestion processes. The SIEM team must be notified of any planned changes to infrastructure or log sources to coordinate and validate service continuity.  
**Proactive monitoring:** Automated monitoring and alerting for log ingestion failures should be implemented to reduce reliance on manual detection and intervention.  
**Change management:** All changes to log collection infrastructure should follow a formal change management process, including stakeholder notification and post-change validation.