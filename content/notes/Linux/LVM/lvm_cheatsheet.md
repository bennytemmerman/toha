---
title: LVM cheatsheet
weight: 402
menu:
  notes:
    name: LVM cheatsheet
    identifier: notes-lvmCheatsheet
    parent: notes-linux
    weight: 42
---
<div style="display: block; width: 100%; max-width: none;">
{{< note title="LVM cheatsheet" >}}

## Acronyms you must know

- **PV** = Physical Volume
- **VG** = Volume Group
- **LV** = Logical Volume

```
    PV1  PV2   PV3  PV4
      \  |      |  /
       VG1      VG2
        |       |  \ 
       LV1     LV2  LV3
```

---

## Step-by-Step LVM Setup

### 1. Add a Physical Disk
connect a physical/virtual disk to your system.  
  
Overview of block devices
```bash
lsblk
```
  
Used/available space on mounted filesystems
```bash
df -h
```

### 2. Create Physical Volumes

*Info*
```bash
pvck        # Check PV metadata
pvdisplay   # Display PV attributes
pvs         # Report PV information
pvscan      # Scan for PVs
```

#### Create or Delete
```bash
pvcreate /dev/sda /dev/sdb /dev/sdc /dev/sdd  # Initialize disks
pvremove /dev/sdX                             # Remove PV
```

#### Edit
```bash
pvchange     # Change PV attributes
pvmove       # Move physical extents
pvresize     # Resize PV
```

---

### 3. Create Volume Groups

#### Info
```bash
vgck        # Check VG metadata
vgdisplay   # VG attributes
vgs         # Report VG info
vgscan      # Scan for VGs
```

#### Create or Delete
```bash
vgcreate vg0 /dev/sda
vgextend vg0 /dev/sdc /dev/sdd
vgcreate vgbackup /dev/sdb
vgremove vg_name
```

#### Edit
```bash
vgcfgbackup / vgcfgrestore
vgchange     # Change VG attributes
vgconvert    # Metadata format change
vgexport / vgimport / vgimportclone
vgmerge / vgsplit / vgreduce / vgrename / vgmknodes
```

---

### 4. Create Logical Volumes

#### Info
```bash
lvdisplay    # Show LV attributes
lvmdiskscan  # Scan for devices
lvs          # Report info
lvscan       # Scan for LVs
```

#### Create / Delete
```bash
lvcreate -n lvbackup -L 50G vgbackup -r
lvremove /dev/vg_name/lv_name
```

#### Edit
```bash
lvchange     # Change LV attributes
lvconvert    # Mirror/snapshot conversion
lvextend     # Extend size
lvreduce     # Reduce size
lvresize     # Resize
lvrename     # Rename
resize2fs /dev/vg0/lv_root   # Resize filesystem
```

---

### 5. Create a Filesystem

```bash
mkfs.ext4 /dev/vgbackup/lvbackup
```

---

### 6. Mounting the Logical Volume

```bash
blkid /dev/vgbackup/lvbackup    # Get UUID
mkdir /path/to/folder           # Mount point
vim /etc/fstab                  # Add entry with UUID
mount -a                        # Mount all from fstab
```

---

## FDISK Reference (Useful Pre-LVM)

### Generic Commands in `fdisk`

| Command | Description                      |
|---------|----------------------------------|
| `d`     | Delete a partition               |
| `F`     | List free space                  |
| `l`     | List known partition types       |
| `n`     | Add a new partition              |
| `p`     | Print partition table            |
| `t`     | Change a partition type          |
| `v`     | Verify partition table           |
| `i`     | Information about a partition    |

```bash
sudo fdisk /dev/sdX
```

---

## Overview Commands

```bash
pvs     # List physical volumes
vgs     # List volume groups
lvs     # List logical volumes

pvdisplay / vgdisplay / lvdisplay
```

---

## Snapshotting

```bash
lvcreate --size 1G --snapshot --name snap_name /dev/vg0/lvdata
lvconvert --merge /dev/vg0/snap_name
```

---

## Filesystem Resize After LV Resize

```bash
lvextend -L +10G /dev/vgbackup/lvbackup
resize2fs /dev/vgbackup/lvbackup
```

---

## Helpful Tips

- Always back up metadata:
  ```bash
  vgcfgbackup
  ```
- Reload partition tables without rebooting:
  ```bash
  partprobe /dev/sdX
  ```

{{< /note >}}
</div>