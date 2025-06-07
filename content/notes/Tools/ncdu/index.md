---
title: ncdu
weight: 210
menu:
  notes:
    name: ncdu
    identifier: notes-ncdu
    parent: notes-tools
    weight: 10
---
<div style="display: block; width: 100%; max-width: none;">

{{< note title="ncdu:" >}}
Ncdu is a disk usage analyzer with a text-mode user interface. It is designed to find space hogs on a remote server where you don’t have an entire graphical setup available, but it is a useful tool even on regular desktop systems. Ncdu aims to be fast, simple, easy to use, and should be able to run on any POSIX-like system.  
install:
```bash
wget https://dev.yorhel.nl/download/ncdu-2.8.1-linux-x86_64.tar.gz
tar -xzf ncdu-2.8.1-linux-x86_64.tar.gz
sudo mv ncdu /usr/local/bin/
```
usage:
```bash
ncdu <directory_name>
```
{{< /note >}}
</div>
