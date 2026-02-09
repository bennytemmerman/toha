---
title: Linux Bash commands
weight: 400
menu:
  notes:
    name: Bash
    identifier: notes-linux-bash
    parent: notes-linux
    weight: 40
---

<div style="display: block; width: 100%; max-width: none;">
<!-- Troubleshooting: -->
{{< note title="Troubleshooting (files):" >}}
Find a file or directory
```bash
find / -type f -name "filename" 2>/dev/null
find / -type d -name "dirname" 2>/dev/null
```
find files that contain a pattern
```bash
find . -type f -exec grep -l 'version' {} \;
```
{{< /note >}}
<!-- Troubleshooting: -->
{{< note title="Troubleshooting (system):" >}}
Check logs if a service has executed/failed before (I needed to check if logrotate was running daily, also checked for rsyslog after)
```bash
journalctl | grep logrotate
```
check if machine is a vm or barebone
```bash
dmidecode -s system-manufacturer
```
check folder disk usage
```bash
du -hs * | sort -h
```
check open port
```bash
(echo > /dev/tcp/10.254.4.54/22) >/dev/null 2>&1 && echo "It's up" || echo "It's down"
```
check for listening ports
```bash
sudo lsof -i -P -n | grep LISTEN
sudo netstat -tulpn | grep LISTEN
sudo ss -tulpn | grep LISTEN
sudo lsof -i:22 ## see a specific port such as 22 ##
sudo nmap -sTU -O IP-address-Here
```
{{< /note >}}

<!-- Troubleshooting: -->
{{< note title="Troubleshooting (SSL):" >}}
Check expiry date on .pem file
```bash
openssl x509 -enddate -noout -in /path/to/certificate.pem
```
{{< /note >}}

<!-- Maintenance: -->
{{< note title="Maintenance:" >}}
dry-run an update on OS
```bash
sudo yum check-update
```
Check if a reboot is required
```bash
needs-restarting -r
```
remove cache of updates for old data
```bash
rm -rf /var/cache/yum
```
block icmp
```bash
sysctl -w net.ipv4.icmp_echo_ignore_all=1
```
{{< /note >}}
<!-- File manipulation: -->
{{< note title="File manipulation:" >}}
remove empty lines from file
```bash
sed -i '/^$/d' <filename>
```
{{< /note >}}
<!-- Disk manipulation: -->
{{< note title="Disk manipulation:" >}}
Check logs while plugging in the new drive
```bash
dmesg --follow
```
List all block devices, including filesystems, UUIDs and labels
```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,LABEL,UUID,MOUNTPOINT
```
detailed info about new disk (Replace X with your new disk e.g. --name=/dev/sdb)
```bash
udevadm info --query=all --name=/dev/sdX
```
Filesystem details (if you encounter command not found -> try sudo)
```bash
blkid
```
SMART/health info (Replace X with your new disk e.g. --name=/dev/sdb)
```bash
sudo smartctl -a -d scsi /dev/sdX
```
{{< /note >}}
<!-- Account: -->
{{< note title="Account control:" >}}
change to root
```bash
sudo -i
```
Grep sudo users
```bash
rm -f /tmp/names; for user in $(getent passwd | cut -d: -f1); do count=$((count+1)); if sudo -l -U "$user" | grep -q "ALL"; then echo "$user" >> /tmp/names; echo "Checked $count of $(getent passwd | cut -d: -f1 | wc -l) users."; fi; done; clear; cat /tmp/names; rm -f /tmp/names
```
{{< /note >}}

</div>