---
title: "RHCSA journey"
date: 2026-09-14T12:00:15+02:00
hero: /images/posts/rhcsa.png
description: Learning Linux Properly: My RHCSA Journey
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
    ↓
Phase 1   Linux fundamentals
    ↓
Phase 2   Files, permissions & users
    ↓
Phase 3   Software & package management
    ↓
Phase 4   Processes, systemd & logs
    ↓
Phase 5   Networking
    ↓
Phase 6   Storage & filesystems
    ↓
Phase 7   Services & scheduled tasks
    ↓
Phase 8   Security & SELinux
    ↓
Phase 9   Bash scripting
    ↓
Phase 10  Boot & system recovery
    ↓
Phase 11  Troubleshooting & integration
    ↓
Phase 12  Capstone projects

---

## Log
- 2026/14/09: creating blogpost + phase 0

---

## PHASE 0: Build your learning lab
_Estimated time: 2–3 hours_

### Goal

1. Build a solid, practical Linux administration foundation by working through the RHCSA body of knowledge and being able to operate, troubleshoot and explain a Linux system without blindly copying commands.

2. Creation of a small dedicated Linux environment. 1 RHEL-compatible distro (Rocky Linux) and 1 Debian-based distro (Ubuntu). I will be using virtual machines instead of LXC (containers) to avoid possible issues related to sharing the hypervisor kernel.

### Set up
[] SSH
[] static/reserved IP
[] normal user
[] sudo
[] hostname
[] basic networking
[] snapshots
[] Git repository for notes
[] a lab-notes.md
[] a troubleshooting.md

### Success looks like
[] SSH into both systems
[] administer them without logging in as root for everything
[] snapshot/revert them
[] deliberately break something and recover
[] explain the purpose of each VM

---

## Distraction log

### GitLab cicd
- Distraction #1: WoW login queue → built GitLab/MkDocs deployment pipeline → 4 hours disappeared.

