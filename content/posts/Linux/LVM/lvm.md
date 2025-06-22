---
title: "LVM"
date: 2025-06-22T9:17:23+02:00
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
# Deepdive into LVM (Logical Volume Manager)

## What is LVM?

LVM stands for **Logical Volume Manager**, a device mapper framework that provides logical volume management for the Linux kernel. LVM allows administrators to create, resize, and delete volumes dynamically, offering more flexibility than traditional partitioning schemes.

## Why use LVM?

- **Dynamic resizing**: Easily resize (extend/reduce) logical volumes without unmounting.
- **Snapshot support**: Create point-in-time snapshots for backup or testing.
- **Volume grouping**: Aggregate multiple physical devices into a single storage pool.
- **Migration**: Move volumes across physical devices live.
- **Flexibility**: Logical volumes can span across multiple disks.

## LVM architecture

![LVM Architecture](images/posts/lvm.png)

- **Physical Volume (PV)**: Raw physical storage (disk, partition, etc).
- **Volume Group (VG)**: Pool of storage formed by combining multiple PVs.
- **Logical Volume (LV)**: Virtual partition formed from a part of the VG.
- **Physical Extents (PE)**: Smallest unit of storage on a PV, a block within the partition/volume
- **Logical Extents (LE)**: Mapped 1:1 to PEs for simplicity.

## Comparison: LVM vs traditional partitioning

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
- **Traditional partitioning**: Simple, low overhead, but limited flexibility.

## Usecase: LVM for database storage

A PostgreSQL database needs fast backups and storage flexibility. With LVM:
- Create an LV for the database.
- Use `lvcreate --size 10G`.
- Take snapshots before major upgrades: `lvcreate --snapshot`.
- Extend storage as DB grows with `lvextend`.

## Important!

Extending a logical volume increases the underlying block device size, but the filesystem itself must also be explicitly resized to utilize the newly allocated space. Without this step, the additional storage remains unavailable to the operating system and applications.  
  
**XFS**  
XFS filesystems must be mounted to resize. Use xfs_growfs on the mount point
```bash
xfs_growfs /path/to/your/mount/point. 
```
  
**ext filesystems**  
For ext-based filesystems (ext2/3/4), you can resize the filesystem either by targeting the block device or the mount point (if already mounted)
```bash
resize2fs /dev/vg_name/lv_name
# or
resize2fs /mount/point
```

## Cheatsheet
> A more extensive cheatsheet can be found here: [LVM cheatsheet](/notes/linux/lvm/lvm_cheatsheet)

