# Create and configure file systems
![Linux File Systems](linuxFileSystems.png)

## Create, mount, unmount, and use VFAT, ext4, and XFS file systems ✅
Before formatting, it's good to understand the diff between these file system types:
- **XFS**: The default file system for RHEL. Highly scalable, excellent for large data sets, but note that it **cannot be shrunk** natively (only grown).
- **ext4**: The traditional Linux workhorse. Standard journaling file system that supports both growing and shrinking.
- VFAT: Essentially FAT32. Zero support for Linux permissions, but used for cross-platform compatibility (like flash drives) and required for the EFI system partition.

To format the existing LVs (or raw partitions), `mkfs` (make file system) utility/command is used, assuming you have an LVM path like `/dev/demo_vg/demo_lv` or multiple, or a partition like `/dev/vdb1`
#### XFS
```bash
sudo mkfs.xfs /dev/demo_vg/demo_lv
```
#### ext4
```bash
# if you have a LV called 'ext4_lv'
sudo mkfs.ext4 /dev/demo_vg/ext4_lv
```
#### VFAT
- If you are missing `.vfat`, install the missing tools:
    - `sudo dnf install dosfstools -y` (this will give you access to more file system types)
    - `sudo dnf install autofs nfs-utils -y` (this will allow you to configure `autofs` and `nfs` for network file systems in later sub-objectives)
```bash
# if you have a LV called 'vfat_lv'
# -F 32: forces FAT32 which is standard
sudo mkfs.vfat -F 32 /dev/demo_vg/vfat_lv
```
#### create a mount point
```bash
sudo mkdir -p /mnt/data_xfs /mnt/data_ext4 /mnt/data_vfat
```
#### persistent mount
```bash
# open file system configs
sudo vi /etc/fstab

# get the UUID of LV devices inside vim, then copy the UUID in visual mode
# do this for the other 2 LVs, if you were told to create 3 mounts
:r !sudo blkid /dev/demo_vg/demo_lv

# the format inside the file is like this:
# 1st coloumn is device/UUID, 2nd: mount point, 3rd: fstype, 4th: options
# 5th: dump, 6th: fsck, this 'fsck' column controls the File System Check (fsck)
# order when the machine boots up or recovers from an unexpected crash
# (like a sudden power loss). 0 means skip (The system completely ignores
# the drive at boot. If the file system gets corrupted due to a sudden
# crash, the OS won't try to fix it automatically.)
# 1 means highest priority (eserved exclusively for the root file system
# (/). The OS must check and repair the root drive first before it can do
# anything else.)
# 2 means secondary priority (Used for any other non-root, standard Linux
# file systems (like ext4) that you want checked and safely repaired
# automatically if the system ever shuts down dirty.)
# the reason why xfs uses 0 but ext4 prefers 2 is: If an XFS system crashes,
# it repairs itself during the actual mount process. Therefore,
# XFS always gets a 0 in fstab. However ext4 relies on boot checks, traditional
# fsck utilities to scan and fix corrupted blocks if a dirty unmount happens.
# Setting it to 2 ensures that if the VM crashes, the system will automatically
# fix any corrupted files before mounting it, protecting the data.
# it's best practice but leaving it at 0 is fine and won't break the system.
UUID=<xfs-uuid>     /mnt/data_xfs    xfs    defaults  0 0
UUID=<ext4-uuid>    /mnt/data_ext4   ext4   defaults  0 2
UUID=<vfat-uuid>    /mnt/data_vfat   vfat   defaults  0 0
```
#### test the fstab (crucial)
Never reboot without testing this command first. If you made a typo in fstab, the VM might drop into an emergency boot shell.
```bash
sudo mount -a
```
Once mounted, you can test reading and writing. Keep in mind that a freshly formatted file system is owned by `root`.

Use `findmnt` command to get info about the mounted file system like it's fstype
```bash
findmnt /mnt/data_ext4
```
You'll see something like:
```bash
TARGET         SOURCE                      FSTYPE OPTIONS
/mnt/ext4-data /dev/mapper/demo_vg-ext4_lv ext4   rw,relatime,seclabel
```

## Mount and unmount network file systems using NFS

## Configure autofs

## Extend existing logical volumes ✅
Extending an existing Logical Volume (LV) is to add space to a live volume without losing the data currently stored on it.

To do this successfully, you must remember that an LV is in two layers:
1. **The Block Device Layer**: The LVM container itself.
2. **The Filesystem Layer**: The XFS or Ext4 format sitting inside that container.

If you only extend the container, your operating system won't see the new space until you also stretch the filesystem to fill it!

While you can run separate commands to grow the LVM container and then grow the filesystem, there is a golden flag (-r or --resizefs) that does both at the exact same time safely and non-destructively.

- Extending to a specific total size
If you have an LV that is currently **2 GB** and the exam asks you to extend it to a total size of **4 GB**, use the uppercase `-L` flag:
```bash
# -L 4G: Tells LVM: "Make the final, absolute size of this volume 4 GB"
# -r: This automatically detects if the volume is XFS or Ext4, and safely
# expands the filesystem to match the new size online, while mounted!
sudo lvextend -L 4G -r /dev/demo_vg/demo_lv
```

- Adding an increment of space
If the exam asks you to add an additional **500 MB** to whatever its current size is, use a plus sign:
```bash
# if the LV was 2GB before, this will make it 2.5GB
sudo lvextend -L +500M -r /dev/demo_vg/demo_lv
```

- Extending by Extents (PEs)
If the exam asks you to extend a volume to a certain number of extents, or to grab all remaining space in the Volume Group, use the lowercase `-l` flag:
```bash
# Extend the volume to use all remaining free space in the Volume Group
sudo lvextend -l +100%FREE -r /dev/demo_vg/demo_lv
```

- Verify the Extension
After running your `lvextend` command, always verify your work with two specific tools to make sure both layers grew:
1. Check the LVM layer
```bash
sudo lvs
```
2. Check the Filesystem layer
```bash
df -h
```
> This checks if the mounted filesystem actually registered the new storage capacity and grew in size.

## Diagnose and correct file permission problems