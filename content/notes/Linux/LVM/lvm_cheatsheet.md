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
{{< note title="Script: add sudo user" >}}
# LVM Cheatsheet

## Initialization

```bash
# Create Physical Volume
pvcreate /dev/sdX

# Create Volume Group
vgcreate vg_name /dev/sdX

# Extend Volume Group
vgextend vg_name /dev/sdY
```

## Logical Volume Operations

```bash
# Create Logical Volume
lvcreate -L 10G -n lv_name vg_name

# Create Logical Volume using all free space
lvcreate -l 100%FREE -n lv_name vg_name

# Resize Logical Volume (online)
lvextend -L +5G /dev/vg_name/lv_name
resize2fs /dev/vg_name/lv_name

# Reduce Logical Volume
umount /mnt
e2fsck -f /dev/vg_name/lv_name
resize2fs /dev/vg_name/lv_name 5G
lvreduce -L 5G /dev/vg_name/lv_name
mount /mnt

# Remove Logical Volume
lvremove /dev/vg_name/lv_name
```

## Snapshots

```bash
# Create Snapshot
lvcreate --size 1G --snapshot --name snap_name /dev/vg_name/lv_name

# Restore from Snapshot
lvconvert --merge /dev/vg_name/snap_name
```

## Information & Listing

```bash
pvs        # List physical volumes
vgs        # List volume groups
lvs        # List logical volumes

lvdisplay  # Show details of logical volumes
vgdisplay  # Show details of volume groups
pvdisplay  # Show details of physical volumes
```

## Removal

```bash
lvremove /dev/vg_name/lv_name
vgremove vg_name
pvremove /dev/sdX
```
{{< /note >}}
</div>