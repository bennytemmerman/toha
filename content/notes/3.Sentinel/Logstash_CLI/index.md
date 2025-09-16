---
title: Logstash CLI commands
weight: 302
menu:
  notes:
    name: Logstash CLI
    identifier: notes-logstash-CLI
    parent: notes-sentinel
    weight: 32
---

<div style="display: block; width: 100%; max-width: none;">
{{< note >}}
# Logstash management
I found these notes from a colleague who worked on a Logstash machine.
```bash
alias logtail='tail -f /var/log/logstash/logstash-plain.log'
alias logconf='cd /etc/logstash/conf.d'
alias logstart='systemctl start logstash'
alias logstop='systemctl stop logstash'
alias logrestart='systemctl restart logstash'
alias logstatus='systemctl status logstash'
alias logsamp='cd /usr/share/logstash/samples'
```
__logtail:__ Continuously displays new entries in the Logstash log file.  
__logconf:__ Navigates to the Logstash configuration directory.  
__logstart:__ Starts the Logstash service.  
__logstop:__ Stops the Logstash service.  
__logrestart:__ Restarts the Logstash service.  
__logstatus:__ Checks the current status of the Logstash service.  
__logsamp:__ Navigates to the directory containing sample Logstash configurations.  

### Making alias permanent for all users
To ensure these aliases are available for all users permanently, you can add them to a global shell configuration file such as /etc/bashrc, /etc/bash.bashrc, or /etc/profile, depending on your system and shell preferences. 
```bash
sudo nano /etc/bashrc
```
Edit bashrc file.
```bash
alias logtail='tail -f /var/log/logstash/logstash-plain.log'
alias logconf='cd /etc/logstash/conf.d'
alias logstart='systemctl start logstash'
alias logstop='systemctl stop logstash'
alias logrestart='systemctl restart logstash'
alias logstatus='systemctl status logstash'
alias logsamp='cd /usr/share/logstash/samples'
```
add the alias commands at the end of the file.
```bash
source /etc/bashrc
```
Save the file and reload the shell configuration.
{{< /note >}}
</div>