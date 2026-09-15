---
title: "RHCSA journey"
date: 2026-09-14T12:00:15+02:00
hero: /images/posts/rhcsa.png
description: "Learning Linux Properly: My RHCSA Journey"
theme: Toha
menu:
  sidebar:
    name: RHCSA journey
    identifier: rhcsa_post
    parent: cat-linux
    weight: 307
---

# RHCSA

## Goal

I want to explore the body of knowledge covered by the RHCSA in order to fill some of the gaps in my Linux knowledge.

The RHCSA is aimed at people building their Linux administration fundamentals, and that's exactly what interests me about it. I'm not approaching this as a certification challenge, I want to understand what I'm doing.

For years I've worked with Linux in one way or another, but I've often found myself in the uncomfortable position of being able to get something working without fully understanding everything underneath it.

I want to change that.

### How I got here

I started my IT career in an MSP, initially working on a Windows-focused helpdesk. I learned a lot there, after about a year I moved into onsite support. That was where I discovered that I really enjoyed troubleshooting. I worked with a broad range of people and environments, solving everything from everyday IT problems to more technical infrastructure issues. Two things in particular started to stand out to me: PowerShell and the command line.

PowerShell was incredibly useful for quickly gathering system information, troubleshooting problems and documenting environments. The other thing I enjoyed was working with network equipment through the CLI. I had previously encountered the CLI while studying for my CCNA, and I remembered how much I enjoyed being able to inspect and configure a system directly rather than relying on a graphical interface.

_There was something satisfying about it._

At the same time, constantly switching between different projects meant that I often lost momentum. I'd learn something, move to something else, and eventually have to start over or repeat a lot of basic work. I wanted to go deeper. From systems to SIEM.

That eventually led me toward a managed security role, where I could learn more about Linux, logs and security infrastructure. When I started, my Linux knowledge was pretty basic. I knew how to navigate directories, list files, create and remove things, and perform the basic tasks I needed. As a SIEM specialist, a large part of my work became Splunk administration. I went through the Splunk certifications relatively quickly: Core User, Power User and Administrator. For a while I felt like I was learning at an incredible pace. I even built and maintained my own Splunk environment at home so I could experiment and learn outside of work.

_This is where I discovered something important about how I learn._

> I learn extremely well when I can build something, break it, investigate why it broke, fix it and document what happened. The problem was that professional life doesn't always provide the time to learn that way.

As workloads increased, experienced colleagues became busier and projects changed. Problems increasingly needed to be fixed quickly rather than properly investigated. New technologies appeared, sometimes without much documentation or training. Monitoring systems changed. Automation needed to be maintained. Azure Sentinel became part of the environment. Existing automation had to be understood and maintained. Infrastructure problems needed solving. There was always something else waiting.

Over time, I accumulated knowledge across a lot of different technologies, but I also accumulated gaps.

- I could work with Linux.
- I could troubleshoot logs.
- I could configure SIEM infrastructure.
- I could work with automation.
- I could build CI/CD pipelines.
- I could make systems talk to each other.

But sometimes I would still stop and think:

"Do I actually understand the Linux underneath all of this as well as I should?"

The answer was no. And that's okay. I'd rather acknowledge the gap and fix it than pretend it isn't there.

### Time to fix that

Recently I was going to play World of Warcraft, I ended up waiting in a login queue. So naturally, I started another project.

I built a GitLab pipeline that generates a MkDocs wiki and deploys it to a remote host running Nginx. Then I started thinking about how I could extend it with AWX to automatically collect system information and feed that information back into the documentation.

Several hours later, it was working. This is a recurring pattern for me.

> I will start with something small because I'm curious, get it working, discover another thing I could improve, and suddenly I've spent half the day building infrastructure instead of doing what I originally planned.

I enjoy it. It is the feeling of taking a system apart, understanding how the pieces connect, putting it back together and seeing the result actually work.

That's the same reason I want to work through the RHCSA material. Not because I need another certificate on my CV. Not because I want to prove that I can memorize a collection of commands.

- I want to make time to understand why those commands work.
- I want to understand what is happening underneath the abstractions I've been working with for years.
- I want Linux to stop being a collection of commands I know how to use and become a system I actually understand.

So this is my attempt to slow down, build the foundation properly, and document the journey along the way. I'll be using the RHCSA body of knowledge as the structure, building things in my lab, deliberately breaking them, troubleshooting them, and documenting what I learn.

There will probably be mistakes. There will definitely be things I don't understand at first.

That's the point.

Let's go.

---

## Roadmap

Phase 0   Lab + learning system  
Phase 1   Linux fundamentals  
Phase 2   Files, permissions & users  
Phase 3   Software & package management  
Phase 4   Processes, systemd & logs  
Phase 5   Networking  
Phase 6   Storage & filesystems  
Phase 7   Services & scheduled tasks  
Phase 8   Security & SELinux  
Phase 9   Bash scripting  
Phase 10  Boot & system recovery  
Phase 11  Troubleshooting & integration  
Phase 12  Capstone projects

---

## Log
- 2026/09/14: creating blogpost (1h)
- 2026/09/15: phase 0, setting up homelab

---

## PHASE 0: Build your learning lab
_Estimated time: 2–3 hours_

### Goal

1. Build a solid, practical Linux administration foundation by working through the RHCSA body of knowledge and being able to operate, troubleshoot and explain a Linux system without blindly copying commands.

2. Creation of a small dedicated Linux environment. 1 RHEL-compatible distro (Rocky Linux) and 1 Debian-based distro (Ubuntu). I will be using virtual machines instead of LXC (containers) to avoid possible issues related to sharing the hypervisor kernel.

### Set up
- [x] Create Rocky Linux VM
- [x] Create Ubuntu VM
- [x] Configure SSH access
- [x] Give both VMs static/reserved IPs
- [ ] Take clean snapshots - LVM storage did not accept snapshot creation in Proxmox
- [x] Verify console access through Proxmox
- [x] Verify SSH access
- [x] Verify internet/DNS connectivity
- [x] Confirm you can destroy/reset the VM if necessary

### Success looks like
- I have two working Linux VMs
- I can SSH into them
- I can safely experiment without worrying about breaking my homelab
- I have a simple system for recording what I learn and parking unrelated ideas.

### Journal

9:30 - 11:00
As I am trying to install a Linux distro on a remote Dell server with virtual media at work, which isn't going as smooth as I thought, it is really slow so in the meantime I can continue learning for RHCSA.

I created 2 hosts on my Proxmox199 host:
- Rocky Linux 9: RHCSA-Rocky (192.168.1.10)
- Ubuntu 26: RHCSA-Ubuntu (192.168.1.11)

Avoiding the use of LXC containers as they share the kernel of the proxmox hypervisor and this might result in different behavior. Rocky Linux will be the primary RHCSA learning environment because it is part of the RHEL ecosystem. Ubuntu is there mainly as a comparison environment. When I encounter a command or configuration that is different between distributions, I can use Ubuntu to investigate the difference.

Using my homepage to quickly access my proxmox web UI, I was thinking about SSO while logging in. Another project to park so I don't get sidetracked too much. I have a Rocky 9 iso ready and a Ubuntu 26, both recent enough for updates + room for dist-upgrade for Rocky. Resources don't need to be over the top, only the install is needed for now and basic config. Storage can be added later on, which will be fun with LVM.

During the installation of Rocky I could already configure the Static IP, perfect. Not sure if I skipped it on Ubuntu or just clicked next too fast, but we can fix this after the install. Looks like Rocky Linux 9 minimal has ssh working right away. I remember in the past having to enable the openssh server or allowing in firewall rules. testing dns and internet connectivity by pinging google.com and running a update and upgrade. Somehow changing the ip to manual in ubuntu desktop changes it back to automatic so it receives from dhcp... Checking the netplan config shows that the config is correct. Let's reboot the host. In order to allow ssh on Ubuntu I had to install the openssh-server first. Next thing to tackle, Rocky Linux minimal has no GUI, so installing that and switching to GUI, setting it as default. I really enjoy CLI, but I think it's equally important to have a GUI feel aswell.

13:00 - 14:00
Next up is making a backup of each vm and creating a snapshot in order to test whether we can restore the vm after making changes or breaking something. Encountered an issue where I can't create snapshots, a forum post explained that lvm storage does not support the creation of snapshots and my vm's are on a lvm configured storage. But we could always restore from backup. Creating a file on each vm to check if the host reverted to the clean install.

The Ubuntu restore worked perfectly as expected. Ready to go. The Rocky vm had localhost as hostname, I wanted to change this first using hostnamectl. After a reboot, the name got changed and we can continue to test the backup. The backup restore for Rocky went equally smooth so we're ready with our testenvironment, let's get learning!

```bash
# Check dns (internal and external) + internet connectivity
ping nextcloud.siemforge.xyz
ping google.com
# Rocky update
dnf update && dnf upgrade -y
# Ubuntu update
apt update && apt upgrade -y
# Check netplan + config
ls /etc/netplan
nano /etc/netplan/90-NM-64002eca-9493-3b7e-be64-07db9f81dd8b.yaml
# Install openssh-server
sudo apt install openssh-server
# Install GUI on Rocky, the isolate command is to switch to GUI, you can also just reboot.
sudo dnf group install "Server with GUI" -y
sudo systemctl set-default graphical.target
sudo systemctl isolate graphical.target
# Configure hostname on Rocky
hostnamectl set-hostname new_hostname
# Also edit in hosts file: 127.0.0.1 RHCSA-Rocky
nano /etc/hosts
# Restart systemd-hostnamed
systemctl restart systemd-hostnamed
```

---

## Distraction log

- Distraction #1: WoW login queue → built GitLab/MkDocs deployment pipeline → 4 hours disappeared.

### possible future projects

- [ ] Add playbooks to AWX repo in order to gather systeminfo to add to my wiki.
- [ ] Configure Authentik to work with Proxmox login.
- [ ] Figure out a way to document an overview of used IP's in the network.