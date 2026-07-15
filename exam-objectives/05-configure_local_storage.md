# Configure local storage
![alt text](LVM_Layer_Architecture.png)

## List, create, and delete partitions on GPT disks ✅
### list partitions
to list all available disks and their partitions, run:
```bash
sudo fdisk -l
```
> This command displays detailed information for every disk. Near the top of each disk's section, look for Disklabel type. If it says gpt, it's a GPT disk.
- a simpler way to show disks and their partitions is running `lsblk`

### create a partition (the interactive flow)
First get an extra disk to create a partition on, in the exam you will have a secondary disk that you can do tasks on, but for practice on your own machine you have to create a disk in the vm you are practicing on, if you are using `virt-manager`, do this to create an extra disk:

- open the VM details windown and click on the **Lightbulb icon** (show virtual hardware details)
- click **add hardware** at the bottom left.
- select **storage**, set the size to 5GiB, and ensure the Device type is set to **Disk Device** and Bus type st to **VirtIO** (this ensures it shows up as `vdb` or `sdb` depending on the storage naming)
- click **Finish** and start the VM.
- once the VM loads, type `lsblk` to verify

if you are using VirtualBox:
- Select your RHEL VM and click **Settings** -> **Storage**.
- Click the **Add Hard Disk** icon next to the Controller (SATA or SCSI).
- Choose **Create**, select **VDI**, pick **Dynamically allocated**, and set the size to `5 GB`.
- Click **Choose/OK** and start the VM.
- once the VM loads, type `lsblk` to verify
> You should now see an unmounted, completely raw `vdb` or `sdb` listed right under `vda/sdb` with a size of 5G.

Now to create a partition from the new disk, lets say it's a `vdb`, run the partition editor:
```bash
sudo fdisk /dev/vdb
```
> This opens an interactive shell. It will not write changes to the disk until you save

Once inside `fdisk`, use these shortcuts:
- type `m` and hit `enter` -> for help, shows all shorcuts and what they do
- `g` and `enter` -> create a GPT label (this initializes the disk as a GPT disk), you skip this part if you are creating a second partition because you already set the disk to be GPT if you did `g` for the first partition.
- `n` -> create a new partition
- when asked for partition number, hit `enter` to accept default (usually 1)
- first sector, hit `enter` to accept default
- last sector (this is where you set the size), to make a `2GB` partition, type `+2G` and hit `enter`
- change the type (crucial for LVM), by default, it creates a standard Linux filesystem. If you want to use it for LVM later, type `t`, then type `30` (which is the code for Linux LVM in GPT), and hit `enter`
- save and exit by typing `w` and hitting `enter`
- after partitioning the disk, the kernal might not instantly see the new partition. Always run this command to force the OS to reload the partition table:
```bash
sudo udevadm settle
```

### delete a partition
- open the disk partition editor
```bash
sudo fdisk /dev/vdb
```
- type `d` (delete), if there are multiple partitions it'll ask you which number to delete, type the number (e.g 1) and hit `enter`
- type `w` to write the changes and exit
- reload the partition table
```bash
sudo udevadm settle
```

## Create and remove physical volumes ✅
A Physical Volume is simply a raw block device (either a whole disk like `/dev/vdb` or a partition like `/dev/vdb1`) that has been initialized so LVM knows it can write metadata and data to it.
### create a physical volume (PV)
You can turn a whole raw disk or a specific partition into a Physical Volume. On the exam, read the instructions carefully, they might specify using the **whole disk** (e.g., `/dev/vdb`) or **a specific partition** (e.g., `/dev/vdb1`).
- initialize a raw disk/partition
```bash
sudo pvcreate /dev/vdb
```
> Note: if you created a partition first, use `/dev/vdb1` instead

If you use the `pvcreate` command on a whole disk instead of a partition, and the disk previously had a partition table or filesystem on it, `pvcreate` will warn you that it found an existing signature and ask if you want to wipe it. Type `y` (yes) to confirm.
### list and verify PVs
There's 3 diff ways to do this:
1. physical volume summary, this gives you a clean, one-line-per-device summary showing the PV name, its VG (if assigned), size, and free space.
```bash
sudo pvs
```
2. physical volume scan, scans all supported LVM block devices on the system and lists them.
```bash
sudo pvscan
```
3. physical volume display, shows detailed attributes, such as UUID, exact physical extent (PE) size, and total extents.
```bash
# if you created the PV from a partition
sudo pvdisplay /dev/vdb1
```
### remove a physical volume
If you make a mistake or the exam asks you to destroy/rebuild storage, you can safely remove the LVM signature from the device.
```bash
# if you created the PV from a partition
sudo pvremove /dev/vdb1
```
> ⚠️ The Safety Catch: LVM will not let you remove a PV if it is currently assigned to an active Volume Group (VG) that is holding active Logical Volumes. You must destroy the Logical Volumes and Volume Groups first (which we'll cover next) before you can run `pvremove`.

## Assign physical volumes to volume groups ✅
Think of a VG as a big virtual hard drive built out of one or more PVs.
### create a VG (Volume Group)
```bash
sudo vgcreate <vg_name> <pv_device>
```
> e.g `sudo vgcreate demo_vg /dev/vdb1`

- On the RHCSA exam, they will sometimes add a specific requirement like "Create a volume group with a Physical Extent (PE) size of **16 MiB**." (Physical Extent is the smallest, fixed-size contiguous chunk of disk space that can be allocated), if you do not specify this, LVM defaults to 4 MiB. To set a custom PE size, use the `-s` option:
```bash
sudo vgcreate -s 16M demo_vg /dev/vdb1
```
### list and verify  VGs
- `sudo vgs` for a quick summary (recommended)
- `sudo vgscan`
- `sudo vgdisplay demo_vg` for detailed info including PE size
### extend and existing VG
If the Volume Group is running out of space, and the exam asks you to add another disk/partition to it? You don't have to rebuild it. You simply "extend" it.

If you have a second partition ready (e.g `/dev/vdb2`):
```bash
# init the new storage
sudo pvcreate /dev/vdb2

# add it to the existing pool
sudo vgextend demo_vg /dev/vdb2
```
> Now, if you run `sudo vgs`, you will see the demo_vg pool has grown by the size of `/dev/vdb2`
### remove a VG
```bash
sudo vgremove demo_vg
```
> Note: you cannot remove a VG if it has active Logical Volumes inside it

## Create and delete logical volumes ✅
Now that we have a Volume Group perfectly set up and extended, we can carve it up into Logical Volumes (LVs).

An LV is the equivalent of a standard partition, but with all the flexibility of LVM (like online resizing). This is the actual block device that you will format with a filesystem and mount to your system.
### create an LV
On the exam, you will usually be asked to create an LV with a specific size (e.g., 2 Gigabytes) or based on Extents (PEs).
- create a Logical Volume by size
To create a Logical Volume named `demo_lv` with a size of **2 Gigabytes** inside `demo_vg`:
```bash
# -L for size (use G for GB and M for MB),
# -n for name, and demo_vg is the parent VG you are carving the space from
sudo lvcreate -L 2G -n demo_lv demo_vg
```
You can create an LV in a specific PV
```bash
sudo lvcreate -L 1.5G -n demo_lv demo_vg /dev/vdb2
```
- create a Logical Volume by extents
Sometimes the exam will explicitly ask you to create a volume using a number of extents (each extent is 4MB by default unless you created the VG with a diff PE size) or a percentage of free space in the VG.
```bash
# create an LV using exactly 50 PE
sudo lvcreate -l 50 -n demo_lv demo_vg

# create an LV using 100% of the remaining free space in the VG
sudo lvcreate -l 100%FREE -n demo_lv demo_vg
```
> You can make both of the above commands target a specific PV by appending `/dev/<PV_name>`
### list and verify LVs
- `sudo lvs` for a quick summary
- `sudo lvdisplay /dev/demo_vg/demo_lv` for detailed info
### delete a Logical Volume
- First unmount it if it's mounted
```bash
sudo umount /path/to/mountpoint
```
- Remove the LV
```bash
sudo lvremove /dev/demo_vg/demo_lv

# remove all LVs under a VG
sudo lvremove /dev/demo_vg/*
```
> LVM will ask you to confirm with `y/n` because this permanently destroys any data on that volume
