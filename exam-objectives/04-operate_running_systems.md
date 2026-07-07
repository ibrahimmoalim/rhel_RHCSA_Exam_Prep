# Operate running systems

## Boot, reboot, and shut down a system normally ✅
- so this is regular server boot, reboot and shutdown, but there's two ways to shutdown, one is higher level `shutdown` command with more options, the other is low level `poweroff` command which instantly cuts the power to the motherboard.
- if you just type `shutdown`, it'll show a message saying something like
system will shutdown in a minute', it won't shutdown right away, you can schedule a shutdown in 10 mins and follow up with a message:
```bash
sudo shutdown -h +10 "System shutting down for maintenance in 10 minutes"
```
For instant shutdown, do:
```bash
sudo poweroff
```
- reboot
```bash
sudo reboot
```

## Boot systems into different targets manually ✅
- see all targets
```bash
systemctl set-default <double-tab-to-see-targets>
```
- view current default target
```bash
systemctl get-default
```
- switch to a diff target on a live system
```bash
systemctl isolate multi-user.target # minimal (no GUI)
systemctl isolate graphical.target # GUI
```
- change the permanent default boot target
```bash
systemctl set-default multi-user.target
```

## Interrupt the boot process in order to gain access to a system (root pass reset) ✅
There are two ways to do this, one is the `init=/bin/bash` method, the other is `rd.break`, RHCSA exam prefers the latter, here's how to do it:
### `rd.break` method
- reboot the system and press any key at the GRUB menu to pause it
- select the kernel (top option) and press `e` to edit
- find the line starting with `linux` or `linux16`, move to the end with `ctrl+e` or arrow keys, and append:
```bash
rd.break
```
- press `ctrl+x` to save edit and boot, the system will boot into an temp/emergency RAM disk environment with the root filesystem mounted as read-only(ro) on `/sysroot`
- remount and reset the password
```bash
mount -o remount,rw /sysroot
# chroot stands for "Change Root". After the remount, the terminal thinks the temporary RAM disk
# is the "top" of the system (/). By running this command, you trick the terminal into believing
# that /sysroot (your actual hard drive) is now the main root (/) directory. This allows you
# to run commands as if you had booted into the system normally.
chroot /sysroot
# You will be prompted to type and confirm your new password
passwd root
# This is the most critical step on RHEL/CentOS. Security-Enhanced Linux (SELinux) relies on
# specific cryptographic "labels" attached to files. Because you changed the password outside
# of the normal boot sequence, the /etc/shadow file just lost its correct SELinux context.
touch /.autorelabel
# This takes you out of your hard drive filesystem (/sysroot) and puts your terminal
# back into the temporary RAM disk environment.
exit
# This tells the kernel that you are done troubleshooting. The system will now catch the
# .autorelabel file you created, spend a minute or two updating its security contexts, and
# then finally present you with the normal login screen where your new password will work.
exit
```
### `init=/bin/bash` method
- reboot the system and press any key at the GRUB menu to pause it
- select the kernel (top option) and press `e` to edit
- find the line starting with `linux` or `linux16`, move to the end with `ctrl+e` or arrow keys, and append:
```bash
init=/bin/bash
```
- press `ctrl+x` to save edit and boot. The system will bypass the normal boot sequence and drop you directly into a raw, passwordless bash shell with root access.
- remount and reset the password
```bash
# By default, interrupting the boot process leaves the filesystems mounted as
# Read-Only ('ro') to protect them. You must explicitly remount the main root
# directory ('/') with Read-Write ('rw') permissions to make changes.
mount -o remount,rw /
# change root password
# You will be prompted to type and confirm your new password
passwd root
# This is the most critical step on RHEL/CentOS. Security-Enhanced Linux (SELinux) relies on
# specific cryptographic "labels" attached to files. Because you changed the password outside
# of the normal boot sequence, the /etc/shadow file just lost its correct SELinux context.
touch /.autorelabel
# Force the kernel to immediately flush all changes from RAM to the physical hard drive.
# Bypassing systemd means standard background disk-writing tools aren't running,
# so skipping this risks losing your files when the shell terminates.
sync
# Replaces the current standalone bash shell by initializing the actual systemd
# boot process ('/sbin/init'). The system will pick up where it left off, notice your
# '.autorelabel' file, run the security scan, and boot cleanly to the login screen.
exec /sbin/init
```

## Identify CPU/memory intensive processes and kill processes ✅
To track down resource hogs, we can use either `top`(interactive)(`htop` won't be available on the exam as it's un-official package, it won't be in the offline repo list either) or `ps`(snapshot/scriptable).
- Run `top` in the terminal, it updates every 3 seconds. To sort the list instantly while it's running, use these shortcuts:
    - Press M (shift+m) to sort by Memory usage
    - Press P (shift+p) to sort by CPU usage
    - Press q to exit
- For the scriptable way (`ps`), to get the top 5 resource-heavy processes, use these:
```bash
# find the top 5 CPU-hogging processes
ps aux --sort=-%cpu | head -n 6

# find the top 5 Memory-hogging processes
ps aux --sort=-%mem | head -n 6
```
- Terminate processes safely
Try a Gracefull Kill (signal 15, which is default). It tells the application, "Please save your work, close your open files, and shut down cleanly."
```bash
# or simple do: kill <PID>
# since -15 is default
kill -15 <PID>
```
- Force kill a process
```bash
kill -9 <PID>
```
- kill by name
If you have multiple instances of a process running (like a rogue script or web worker) and you don't want to hunt down every single PID, use pkill or killall:
```bash
# kills all running processes named "httpd" gracefully
pkill httpd

# force kill
pkill -9 <process-name>
```
- On the exam, Red Hat might give you a scenario where a specific user or background process is consuming resources or blocking a port.
If a task tells you to "Terminate all processes owned by user 'ali'", you don't have to manually hunt every PID down. Just use the `-u` flag:
```bash
sudo pkill -u ali
```

## Adjust  process scheduling
This means changing how the Linux kernel prioritizes CPU access for different tasks.

On the exam, you will be expected to know how to start a new task with a specific priority or alter the priority of an already running process.