# Create and configure file systems

## Create, mount, unmount, and use VFAT, ext4, and XFS file systems

## Mount and unmount network file systems using NFS

## Configure autofs

## Extend existing logical volumes
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