# Configure local storage

## List, create, and delete partitions on GPT disks
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
- `g` and `enter` -> create a GPT label (this initializes the disk as a GPT disk)
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
