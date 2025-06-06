---
title: "Logrotate"
date: 2025-06-06T14:17:23+02:00
#hero: /images/posts/logrotate.svg
hero: /images/posts/logrotate.jpg
description: Explaining logrotate configuration
theme: Toha
menu:
  sidebar:
    name: Logrotate
    identifier: logrotate
    parent: cat-linux
    weight: 500
---
# Mastering Logrotate: The Unsung Hero of Log Management

In the trenches of system administration, there's one silent guardian that keeps your disk space from imploding under a mountain of logs: **Logrotate**. Whether you're wrangling logs on a sprawling Kubernetes cluster or just babysitting a single Linux box, Logrotate ensures your logs don’t spiral out of control.

In this deep dive, we’ll explore how Logrotate works, why it’s essential, how to configure it like a pro, and how to troubleshoot when it throws a tantrum.

---

## What is Logrotate?

At its core, **Logrotate** is a Unix utility designed to automatically rotate, compress, and remove log files. Think of it as your log janitor — sweeping away old logs, zipping them up neatly, and ensuring your system never runs out of space due to runaway logging.

### Why It Matters:

Imagine you’re running NGINX on a production server handling thousands of requests per second. Without log rotation, your access and error logs could balloon into gigabytes overnight. That’s a **disk I/O nightmare** waiting to happen.

---

## Logrotate Configuration: From Basics to Battle-Hardened

Logrotate’s power lies in its flexibility. You can define global rules in `/etc/logrotate.conf`, or per-service rules in `/etc/logrotate.d/`.

### Example: `/etc/logrotate.d/nginx`

```bash
/var/log/nginx/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        systemctl reload nginx > /dev/null 2>&1 || true
    endscript
}
```

## What Each Directive Does:  

`daily: `Rotate logs every day (alternatives: weekly, monthly, or custom intervals).  
`rotate 14: `Keep 14 archived logs before purging.  
`compress: `Gzip old logs to save space.  
`delaycompress: `Wait a cycle before compressing (avoids compressing still-used logs).  
`missingok: `Don’t freak out if a log file is missing.  
`notifempty: `Skip rotation for empty logs.  
`create 0640 www-data adm: `Create new logs with specific permissions.  
`sharedscripts: `Ensures postrotate only runs once per log group.  
`postrotate...endscript: `What to do after rotating logs — in this case, gracefully reload NGINX.  

> Pro Tip: Use copytruncate if your app won’t release the log file handle — useful for apps that don't support log reopening on SIGHUP.

## Automating with Cron

Logrotate is typically triggered by cron, so unless you're into manual rotations (why?), make sure this line exists in your cron jobs:

```bash
0 0 * * * /usr/sbin/logrotate /etc/logrotate.conf >/dev/null 2>&1
```

This runs it daily at midnight. You can customize it further with anacron or systemd timers if you're using newer systems like RHEL 8+ or Ubuntu 22.04+.
## Common Pitfalls and How to Debug Like a Boss

Even a veteran sysadmin can get tripped up by Logrotate quirks. Here are some frequent headaches and their antidotes:
### Problem	Fix
🔁 Log not rotating?	Check if the cron job runs. Use logrotate -d for dry-run debugging.  
🔒 Permission denied?	Ensure Logrotate has access (run as root or use sudo).  
❌ Syntax error in config?	Run logrotate -v /etc/logrotate.conf to see what it’s doing.  
💾 Disk still filling up?	Reduce rotate count or increase compression aggressiveness.  
🔄 App not releasing log files?	Use copytruncate or restart the service post-rotate.  

## Docker
Running containers? Docker log files can balloon fast under /var/lib/docker/containers/. You can use logrotate or — better — configure Docker's own json-file driver like this:
```bash
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  }
}
```
Boom — Docker will auto-rotate logs per container.
## Final Thoughts

Logrotate might not be flashy, but it’s a critical piece of your system's hygiene. Treat it like a core dependency — because it is. With smart configurations, regular checks, and a few advanced tweaks, you can keep your logs lean, searchable, and under control.

> Whether you're taming Apache logs, managing containerized chaos, or just keeping your home lab in shape, Logrotate will be your quiet, powerful ally.
## TL;DR Cheat Sheet

- Config files: /etc/logrotate.conf, /etc/logrotate.d/*

- Common flags: daily, rotate, compress, create, copytruncate

- Debug command: logrotate -d /etc/logrotate.conf

- Test manually: logrotate -f /etc/logrotate.conf

- Keep it cron’d: crontab -e

> A more extensive cheatsheet can be found here: [Logrotate cheatsheet](/notes/Linux/Logrotate/index)

Now go forth and rotate like a boss. 🌀
