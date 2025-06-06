---
title: Logrotate config
weight: 211
menu:
  notes:
    name: Logrotate
    identifier: notes-logrotate
    parent: notes-linux
    weight: 11
---

<div style="display: block; width: 100%; max-width: none;">
<!-- Troubleshooting: -->
{{< note title="Logrotate Configuration Cheat Sheet:" >}}
This cheat sheet provides an extensive list of Logrotate configuration directives, their descriptions, and examples. 
Use this as a quick reference to master log rotation on Unix-like systems.
{{< /note >}}

{{< note title="Basic structure" >}}
Each configuration block is tied to a log file or set of log files. Example:
```bash
/var/log/example.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 0640 root adm
}
```
{{< /note >}}

{{< note title="Configuration Directives & Examples" >}}
#### Basic Settings:

    - rotate <count>
    Keep <count> number of old log files before deleting them.

    - daily | weekly | monthly | yearly
    Frequency of rotation.

#### Compression Options:

    compress
    Compress old versions of log files with gzip.

    nocompress
    Do not compress old logs.

    delaycompress
    Postpone compression to the next rotation cycle (used with compress).

#### File Handling:

    missingok
    Ignore missing log files and don’t issue an error.

    notifempty
    Do not rotate the log if it is empty.

    ifempty
    Rotate the log even if it is empty (default behavior).

    create <mode> <owner> <group>
    Create a new log file with specified permissions.

    copy
    Make a copy of the log file and truncate the original.

    copytruncate
    Truncate the original log file after copying it (useful for active logs).

#### Date & Naming:

    dateext
    Append an extension with the current date to rotated log files.

    dateformat .%Y-%m-%d
    Custom format for dateext (e.g., .2025-06-06).

    extension <ext>
    Force specific extension for rotated files (e.g., .log).

#### Size-Based Rotation:

    maxage <days>
    Remove rotated logs older than <days>.

    minsize <size>
    Rotate only if log size is above <size>.

    size <size>
    Rotate if log file size meets threshold, regardless of time.

    maxsize <size>
    Do not rotate if log is larger than specified size.

#### Directory & Scripts:

    olddir <dir>
    Move rotated logs to a specified directory.

    sharedscripts
    Run postrotate script once for all matching logs.

    postrotate/endscript
    Script to run after log rotation.

    prerotate/endscript
    Script to run before log rotation.

    firstaction/endscript
    Run only once before rotation begins (before prerotate).

    lastaction/endscript
    Run once after rotation finishes (after postrotate).

    tabooext + <ext>
    Treat additional extensions as taboo (not rotated).
{{< /note >}}
{{< note title="Full Example Configuration" >}}
```bash
/var/log/myapp/*.log {
    daily
    rotate 10
    size 100M
    compress
    delaycompress
    missingok
    notifempty
    create 0640 appuser adm
    sharedscripts
    postrotate
        systemctl reload myapp > /dev/null 2>&1 || true
    endscript
}
```
{{< /note >}}
{{< note title="Tips" >}}
•	Run `logrotate -d <config>` to debug your config without applying changes.
•	Use `logrotate -f <config>` to force rotation for testing.
•	Logrotate is typically triggered via cron or systemd timers.
•	Keep your config DRY by centralizing shared logic in /etc/logrotate.conf and using includes.
{{< /note >}}
</div>