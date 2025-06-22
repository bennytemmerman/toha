---
title: fdisk
weight: 501
menu:
  notes:
    name: fdisk
    identifier: notes-fdisk
    parent: notes-tools
    weight: 51
---
<div style="display: block; width: 100%; max-width: none;">

{{< note title="fdisk cheatsheet:" >}}

fdisk is a powerful, text-based utility used to create, delete, and manage disk partitions in Linux systems. It supports MBR (Master Boot Record) partition tables and is best suited for systems not using GPT (GUID Partition Table).

## Basic Syntax

```bash
fdisk [options] /dev/sdX
```

Where `/dev/sdX` is the disk you want to operate on (e.g., `/dev/sda`, `/dev/sdb`).

> Always double-check the disk name to avoid data loss.

---

## Key Commands in `fdisk` Interactive Mode

| Command | Description |
| --- | --- |
| `m`     | Print help menu |
| `p`     | Display existing partition table |
| `n`     | Create a new partition |
| `d`     | Delete a partition |
| `t`     | Change a partition's system ID (type) |
| `a`     | Toggle bootable flag |
| `w`     | Write changes and exit |
| `q`     | Quit without saving changes |

---

## Common Workflow Examples

### 1. View Partition Table

```bash
sudo fdisk -l
```

Lists all disks and their partitions.

### 2. Start fdisk on a Specific Disk

```bash
sudo fdisk /dev/sdX
```

Enters interactive mode.

### 3. Create a New Partition

```bash
Command (m for help): n
Select default or choose primary (p) or extended (e)
Partition number: 1
First sector: [Press Enter to accept default]
Last sector: +1G  # or specify size like +20G
```

### 4. Change Partition Type

```bash
Command (m for help): t
Partition number: 1
Hex code (type L for list): 83  # Linux filesystem
```

### 5. Set Bootable Flag

```bash
Command (m for help): a
Partition number: 1
```

### 6. Write Changes to Disk

```bash
Command (m for help): w
```

Writes the partition table to disk and exits.

### 7. Quit Without Saving

```bash
Command (m for help): q
```

Exits without modifying the disk.

---

## Important Options

```bash
-f       # Use a script file
-l       # List partition tables for all devices
-u       # Display sectors in cylinders or sectors
```

---

## Partition Type Codes (Common)

| Code | Filesystem Type       |
|------|------------------------|
| 83   | Linux                  |
| 82   | Linux swap             |
| 7    | HPFS/NTFS/exFAT        |
| b    | W95 FAT32              |
| c    | W95 FAT32 (LBA)        |
| a5   | FreeBSD                |

To list all type codes:

```bash
Command (m for help): L
```

---

## Post Partitioning

After creating partitions, always format them:

```bash
mkfs.ext4 /dev/sdX1
mkfs.vfat /dev/sdX2
```

Mount the partition:

```bash
mount /dev/sdX1 /mnt/mydisk
```

---

## Helpful Tips

- Use `partprobe` or reboot after changing partition table:
  ```bash
  sudo partprobe /dev/sdX
  ```
- Use `lsblk` or `blkid` to view and identify new partitions:
  ```bash
  lsblk
  blkid
  ```

---

## References

- `man fdisk`
- `https://wiki.archlinux.org/title/Fdisk`

{{< /note >}}
</div>
