---
title: "TCPDump"
date: 2025-06-24T10:25:15+02:00
#hero: /images/posts/logrotate.svg
#hero: /images/posts/tcpdump.png
description: TCPDump usage
theme: Toha
menu:
  sidebar:
    name: tcpdump
    identifier: tcpdump
    parent: cat-linux
    weight: 302
---
# TCPDump deepdive for Linux engineers

As a SIEM platform administrator, one of the most invaluable tools in my troubleshooting arsenal is tcpdump. It allows the user to display TCP/IP and other packets being transmitted or received over a network. Despite its simplicity, it is incredibly powerful for debugging complex network issues, monitoring traffic, or simply learning how different protocols behave.

## Why use tcpdump

- Real-time packet inspection
- Lightweight and scriptable
- No need for a GUI
- Useful for security auditing and troubleshooting

## How to install tcpdump

On Debian/Ubuntu-based systems:
```bash
sudo apt update && sudo apt install tcpdump
```
On RHEL/CentOS (YUM-based systems):
```bash
sudo yum install tcpdump
```
On Fedora/DNF-based systems:
```bash
sudo dnf install tcpdump
```
From Source:
```bash
sudo apt install libpcap-dev gcc make -y
wget http://www.tcpdump.org/release/tcpdump-4.99.4.tar.gz
tar -xvzf tcpdump-4.99.4.tar.gz
cd tcpdump-4.99.4
./configure
make
sudo make install
```

## Basic Usage Examples

Capture traffic on eth0
```bash
tcpdump -i eth0
```
Capture only TCP packets
```bash
tcpdump -i eth0 tcp
```
Capture packets from a specific host
```bash
tcpdump -i eth0 host 192.168.1.100
```
Save captured data to a file
```bash
tcpdump -i eth0 -w capture.pcap
```
Read from a saved capture file
```bash
tcpdump -r capture.pcap
```
Limit the number of packets
```bash
tcpdump -i eth0 -c 100
```
Filter by port
```bash
tcpdump -i eth0 port 443
```
Verbose output with hex
```bash
tcpdump -XX -i eth0
```
Display packet timestamp in readable format
```bash
tcpdump -tttt -i eth0
```
Real-World Example: Debugging HTTP Requests
```bash
tcpdump -i eth0 -nn -s 0 -v tcp port 80
```
This captures HTTP requests in verbose mode, disables hostname and port name resolution, and captures the entire packet.

## Security Consideration

You need elevated privileges to run tcpdump. Consider creating a dedicated group or using capabilities:
```bash
sudo setcap cap_net_raw,cap_net_admin=eip $(which tcpdump)
```

## Conclusion
Whether you're a network administrator, security professional, or developer, tcpdump is a skill worth mastering. Its ability to dissect network traffic at the most granular level makes it indispensable in any Linux environment.