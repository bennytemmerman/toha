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
- 2026/09/15: phase 0, setting up homelab (2.5h)

---

## Learning journal
- 2026/09/15: LVM storage on proxmox does not support snapshot creation

---

## PHASE 0: Build your learning lab
_Estimated time: 2–3 hours_

### Lab environment

1. Build a solid, practical Linux administration foundation by working through the RHCSA body of knowledge and being able to operate, troubleshoot and explain a Linux system without blindly copying commands.

2. Creation of a small dedicated Linux environment. 1 RHEL-compatible distro (Rocky Linux) and 1 Debian-based distro (Ubuntu). I will be using virtual machines instead of LXC (containers) to avoid possible issues related to sharing the hypervisor kernel.

#### Set up
- [x] Create Rocky Linux VM
- [x] Create Ubuntu VM
- [x] Configure SSH access
- [x] Give both VMs static/reserved IPs
- [ ] Take clean snapshots - LVM storage did not accept snapshot creation in Proxmox
- [x] Verify console access through Proxmox
- [x] Verify SSH access
- [x] Verify internet/DNS connectivity
- [x] Confirm you can destroy/reset the VM if necessary

#### Success looks like
- I have two working Linux VMs
- I can SSH into them
- I can safely experiment without worrying about breaking my homelab
- I have a simple system for recording what I learn and parking unrelated ideas.

#### Journal

9:30 - 11:00  
As I am trying to install a Linux distro on a remote Dell server with virtual media at work, which isn't going as smooth as I thought, it is really slow so in the meantime I can continue learning for RHCSA.

I created 2 hosts on my Proxmox199 host:
- Rocky Linux 9: RHCSA-Rocky (192.168.1.10)
- Ubuntu 26: RHCSA-Ubuntu (192.168.1.11)

Avoiding the use of LXC containers as they share the kernel of the proxmox hypervisor and this might result in different behavior. Rocky Linux will be the primary RHCSA learning environment because it is part of the RHEL ecosystem. Ubuntu is there mainly as a comparison environment. When I encounter a command or configuration that is different between distributions, I can use Ubuntu to investigate the difference.

Using my homepage to quickly access my proxmox web UI, I was thinking about SSO while logging in. Another project to park so I don't get sidetracked too much. I have a Rocky 9 iso ready and a Ubuntu 26 iso. Resources don't need to be over the top, only the install is needed for now and basic config. Storage can be added later on, which will be fun with LVM.

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

## PHASE 1: Essential Linux tools & command line
_Estimated time: 6-10 hours_

### 1.1 Shell fundamentals
_Estimated: 1–1.5 hours_

#### Commands to test
```bash
pwd
ls
cd
echo
type
which
command
history
env
printenv
whoami
id
sudo
su
```

#### Lab 
Navigation in CLI
```bash
cd /
ls
cd /etc
pwd
cd ~
pwd
```
Explore other commands
```bash
echo $PATH
echo $HOME
echo $USER
whoami
id
```

#### Success looks like
You can sit at a shell prompt and comfortably answer:

- Where am I?
- Who am I?
- What commands are available?
- Where does this command come from?
- How do I get somewhere else?
- What does this command's output mean?
- Did my previous command succeed?

You shouldn't need to memorize every command. The important part is understanding the environment.

#### Journal

### 1.2 Files and directories
_Estimated: 1.5–2 hours_

#### Commands to test
```bash
ls
cd
pwd
mkdir
touch
cp
mv
rm
rmdir
file
stat
```

#### Lab 
Create the following filesystem:
```bash
~/rhcsa-lab/
├── documents/
├── scripts/
├── backups/
└── test/
```
Create files, copy them, move them, rename them and delete them.

#### Success looks like

You can manipulate files/directories confidently without thinking about every command.
what are the following:
- absolute paths
- relative paths
- hidden files

#### Journal

### 1.3 Reading and manipulating text
_Estimated: 1.5–2 hours_

#### Commands to test
```bash
cat
less
head
tail
wc
sort
uniq
cut
grep
|
>
>>
2>
```

#### Lab 
Try the following commands and explain how they work
```bash
cat /etc/passwd | grep bash
grep root /etc/passwd
grep -i root /etc/passwd
grep -n root /etc/passwd
cat /etc/passwd > users.txt
echo "test" >> users.txt
cat users.txt
```
#### Success looks like

You understand this: 
command A | command B
and:
stdout ──> file
stderr ──> file

#### Journal

### 1.4 Searching and finding things
_Estimated: 1 hour_

#### Commands to test
```bash
find
locate
which
whereis
type
grep
```

#### Success looks like
Someone tells you:
“There's a configuration file somewhere under /etc.”
Your first reaction isn't:
“Where is that again?”
It's:
“I'll find it.”

#### Journal

### 1.5 Archives and compression
_Estimated: 1 hour_

#### Commands to test
```bash
tar
gzip
gunzip
bzip2
bunzip2
```
#### Lab 
```bash
tar -cf archive.tar directory/
tar -xf archive.tar
tar -czf archive.tar.gz directory/
tar -xzf archive.tar.gz
```

#### Success looks like
You can archive a directory, inspect the archive, extract it somewhere else and explain what happened.
Understand the distinction:
- tar     = archive
- gzip    = compression
- tar.gz  = tar archive compressed with gzip

#### Journal

### 1.6 Links
_Estimated: 45–60 minutes_

#### Commands to test
```bash
ln
ln -s
ls -li
```
#### Lab 
```bash
touch original.txt
ln original.txt hardlink.txt
ln -s original.txt symlink.txt
ls -li
```
Delete the original and observe what happens to each link.
#### Success looks like
You can explain:
What is the difference between a hard link and a symbolic link?
without looking it up.
You don't necessarily need to memorize filesystem implementation details yet.

#### Journal

### 1.7 System documentation
_Estimated: 1 hour_

#### Commands to test
```bash
man
info
ls /usr/share/doc
```
#### Lab 
Learn to use:
```bash
man ls
man systemctl
man chmod
```
Navigation:
```bash
/       search
n       next match
q       quit
```
Also:
```bash
info
```
and:
```bash
ls /usr/share/doc
```

#### Success looks like
Don't try to memorize everything.
Instead:
Learn how to find the answer.
That is a much more useful Linux skill.

#### Journal

### 1.8 SSH and remote systems
_Estimated: 1 hour_

#### Commands to test
```bash
ssh user@host
ssh -p PORT user@host
scp
```

#### Lab 
connect to Rocky vm
copy a file to Rocky vm

#### Success looks like
You can comfortably connect between your systems and transfer a file between them.
#### Journal

### 1.9 Multi-user targets
_Estimated: 45–60 minutes_
The RHCSA objective explicitly includes logging in and switching users in multi-user targets.

#### Commands to test
```bash
systemctl get-default
systemctl list-units
```
#### Lab 
accidentally encountered this while installing GNOME:
```bash
systemctl set-default graphical.target
systemctl isolate graphical.target
```
#### Success looks like
understand the concept of:
- multi-user.target
- graphical.target

#### Journal

---  

### Challenge

Start with a clean Rocky VM.

Without following a step-by-step tutorial:

- [ ] SSH into Rocky.
- [ ] Create a directory structure.
- [ ] Create several files.
- [ ] Copy and move files.
- [ ] Create a hard link and symbolic link.
- [ ] Find files using find.
- [ ] Search their contents using grep.
- [ ] Redirect output to a file.
- [ ] Append output to another file.
- [ ] Create a .tar.gz archive.
- [ ] Extract it again.
- [ ] Find the relevant documentation using man.
- [ ] SSH from Rocky into Ubuntu.
- [ ] Transfer a file between the VMs.
- [ ] Determine your current user, hostname and working directory.
- [ ] Explain what happens when you run a command that doesn't exist.
- [ ] Explain the difference between:
  - [ ] >
  - [ ] >>
  - [ ] |
  - [ ] 2>
- [ ] Explain the difference between a hard link and a symbolic link.

---

## Distraction log

- Distraction #1: WoW login queue → built GitLab/MkDocs deployment pipeline → 4 hours disappeared.

### possible future projects

- [ ] Add playbooks to AWX repo in order to gather systeminfo to add to my wiki.
- [ ] Configure Authentik to work with Proxmox login.
- [ ] Figure out a way to document an overview of used IP's in the network.