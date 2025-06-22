---
title: "LVM"
date: 2025-06-022T14:17:23+02:00
#hero: /images/posts/logrotate.svg
hero: /images/posts/logrotate.jpg
description: Explaining LVM configuration
theme: Toha
menu:
  sidebar:
    name: LVM
    identifier: lvm
    parent: cat-linux
    weight: 301
---
# Deep Dive into LVM (Logical Volume Manager)

## What is LVM?

LVM stands for **Logical Volume Manager**, a device mapper framework that provides logical volume management for the Linux kernel. LVM allows administrators to create, resize, and delete volumes dynamically, offering more flexibility than traditional partitioning schemes.

## Why Use LVM?

- **Dynamic Resizing**: Easily resize (extend/reduce) logical volumes without unmounting.
- **Snapshot Support**: Create point-in-time snapshots for backup or testing.
- **Volume Grouping**: Aggregate multiple physical devices into a single storage pool.
- **Migration**: Move volumes across physical devices live.
- **Flexibility**: Logical volumes can span across multiple disks.

## LVM Architecture

![LVM Architecture](https://upload.wikimedia.org/wikipedia/commons/thumb/8/82/Lvm.png/800px-Lvm.png)

- **Physical Volume (PV)**: Raw physical storage (disk, partition, etc).
- **Volume Group (VG)**: Pool of storage formed by combining multiple PVs.
- **Logical Volume (LV)**: Virtual partition carved from the VG.
- **Physical Extents (PE)**: Smallest unit of storage on a PV.
- **Logical Extents (LE)**: Mapped 1:1 to PEs for simplicity.

## Comparison: LVM vs Traditional Partitioning

| Feature                 | LVM                         | Traditional Partitioning       |
|------------------------|-----------------------------|--------------------------------|
| Resizing Volumes       | Dynamic                     | Static                         |
| Snapshots              | Yes                         | No                             |
| Disk Spanning          | Yes (via VGs)               | No                             |
| Performance Overhead   | Slight (negligible in most) | None                           |
| Complexity             | Moderate                    | Low                            |

## Alternatives

- **ZFS**: Advanced file system with built-in volume management and redundancy.
- **Btrfs**: Modern Linux filesystem with snapshot and RAID features.
- **Traditional Partitioning**: Simple, low overhead, but limited flexibility.

## Use Case: LVM for Database Storage

A PostgreSQL database needs fast backups and storage flexibility. With LVM:
- Create an LV for the database.
- Use `lvcreate --size 10G`.
- Take snapshots before major upgrades: `lvcreate --snapshot`.
- Extend storage as DB grows with `lvextend`.

## Acronyms

| Acronym | Meaning                         |
|---------|----------------------------------|
| LVM     | Logical Volume Manager          |
| PV      | Physical Volume                 |
| VG      | Volume Group                    |
| LV      | Logical Volume                  |
| PE      | Physical Extent                 |
| LE      | Logical Extent                  |

## Animated Overview

![LVM Workflow](https://linuxhint.com/wp-content/uploads/2020/11/01-LVM-Architecture.gif)

