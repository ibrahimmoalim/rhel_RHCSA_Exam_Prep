# Operate running systems ✅

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
# that /sysroot (the actual hard drive) is now the main root (/) directory. This allows you
# to run commands as if you had booted into the system normally.
chroot /sysroot
# You will be prompted to type and confirm the new password
passwd root
# This is the most critical step on RHEL/CentOS. Security-Enhanced Linux (SELinux) relies on
# specific cryptographic "labels" attached to files. Because you changed the password outside
# of the normal boot sequence, the /etc/shadow file just lost its correct SELinux context.
touch /.autorelabel
# This takes you out of the hard drive filesystem (/sysroot) and puts the terminal
# back into the temporary RAM disk environment.
exit
# This tells the kernel that you are done troubleshooting. The system will now catch the
# .autorelabel file you created, spend a minute or two updating its security contexts, and
# then finally present you with the normal login screen where the new password will work.
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
# You will be prompted to type and confirm the new password
passwd root
# This is the most critical step on RHEL/CentOS. Security-Enhanced Linux (SELinux) relies on
# specific cryptographic "labels" attached to files. Because you changed the password outside
# of the normal boot sequence, the /etc/shadow file just lost its correct SELinux context.
touch /.autorelabel
# Force the kernel to immediately flush all changes from RAM to the physical hard drive.
# Bypassing systemd means standard background disk-writing tools aren't running,
# so skipping this risks losing the files when the shell terminates.
sync
# Replaces the current standalone bash shell by initializing the actual systemd
# boot process ('/sbin/init'). The system will pick up where it left off, notice the
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
Try a Gracefull Kill (signal 15, which is default). It tells the application, "Please save the work, close the open files, and shut down cleanly."
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

## Adjust  process scheduling ✅
This means changing how the Linux kernel prioritizes CPU access for different tasks.

On the exam, you will be expected to know how to start a new task with a specific priority or alter the priority of an already running process.
- Linux uses `nice` values to control process priority
    - think of it as how "nice" a process is to other processes on the system
    - the scale runs from `-20` to `19`

| Nice Value | Priority Level | Behavior |
| --- | --- | --- |
| **`-20`** | **Highest Priority** | Not nice at all. It grabs CPU time aggressively. |
| **`0`** | **Default Priority** | Standard background/user task behavior. |
| **`19`** | **Lowest Priority** | Extremely nice. It only runs when the CPU has absolutely nothing else to do. |

> ⚠️ **Critical Rule:** Any standard user can make their own process *less* important (increase nice value to 10, 15, etc.). However, **only root (or `sudo`) can make a process more important** (negative nice values) because it impacts overall system performance.

- Setting Priority on a NEW Process (`nice`)

If you are about to launch a heavy, resource-intensive job (like a backup compression or a massive file search) and you don't want it to slow down the server, you start it with a high nice value.

```bash
# Start a backup task with a LOW priority (nice value of 15)
nice -n 15 tar -czf backup.tar.gz /var/log/

# Start a critical script with a HIGH priority (nice value of -5)
sudo nice -n -5 ./critical_script.sh
```

- Changing Priority on an ALREADY RUNNING Process (`renice`)

If a process is already running and causing issues, or if you need it to finish faster, you modify it live using its **PID** (Process ID).

```bash
# First, locate the PID of the rogue process (e.g., let's say 'dd' is running)
pgrep dd
# Let's say it returns PID: 4521

# Lower its priority instantly to be 'nice' to the system
sudo renice -n 10 -p 4521

# Drop it to maximum priority (requires root/sudo)
sudo renice -n -20 -p 4521
```

- Exam Tip: Verifying Your Work

If the exam task asks you to *"Change the nice value of the process named 'my_app' to 5"*, you can verify that the change took effect using `ps`:

```bash
ps -eo pid,ni,comm | grep my_app
```

* The **`ni`** column stands for Nice. Make sure it shows exactly what was requested!

## Manage tuning profiles ✅
Managing tuning profiles in RHEL revolves around the `tuned` system. It is a background service that dynamically optimizes system settings (like disk scheduling, CPU governors, and network buffers) based on the specific workload profile you select.

if the system doesn't have `tuned-adm` and `tuned` command, install it with:
```bash
sudo dnf install tuned
```
Start the background process and enable it so it stays running
```bash
sudo systemctl enable --now tuned
```

- checking the current state
Before changing anything, run this:
```bash
tuned-adm active
```
> output example: current active profile: virtual-guest
- list all available profiles on the system
```bash
tuned-adm list
```
- let the system choose a profile
If you aren't sure which profile is best for the current hardware configuration, `tuned` can analyze the environment and give you a recommendation.
```bash
tuned-adm recommend
```
- switch profiles (To change the profile, use the `profile` subcommand. This change takes effect immediately and remains permanent across system reboots.)
    - switch to a specific profile
    ```bash
    # 'throughput-perfomance' is Broadly applicable tuning that provides
    # excellent performance across a variety of common server workloads
    sudo tuned-adm profile throughput-perfomance
    ```
    - verify the change
    ```bash
    tuned-adm active
    ```
- turn off tuning
Sometimes an exam task or a troubleshooting scenario might ask you to completely disable dynamic tuning to isolate a performance issue.
```bash
# turn off tuning entirely
sudo tuned-adm off
```
To turn if back on simply select a profile again with:
```bash
sudo tuned-adm profile <profile-name>
```

## Locate and interpret system log files and journals ✅
RHEL uses a dual-logging system: traditional plain-text log files (managed by `rsyslog`) and a modern, secure binary logging system (managed by `systemd-journald`).

### Traditional Text Logs (`/var/log/`)
These are plain text files. You can read them using tools like `cat`, `less`, `tail`, or `grep`.
- `var/log/messages` -> The catch-all log for general system messages, global scripts, and non-authentication errors.
- `/var/log/secure` -> The security vault. It logs every `sudo` attemp, SSH login. password failure, and user/group creation.
- `/var/log/dnf.log` -> Shows what packages were installed, updated, or deleted and when.

### Modern Binary Logging (`journalctl`)
The systemd journal captures everything from early boot messages to application crashes. Because it is a binary format, you cannot open it with `less` or `cat`. You must use `journalctl`.
- Filter by service
```bash
# -u means unit
journalctl -u httpd.service
```
- Filter by severity
See only warnings, critical issues, or errors:
```bash
# -p means priority
journalctl -p err
```
- Filter by time
```bash
journalctl --since "15 minutes ago"
journalctl --since "2026-07-09 10:00:00"
```
- LIve track (like `tail -f`)
```bash
journalctl -f
```

## Preserve system journals ✅
By default on many RHEL installations, the systemd journal is ephemeral. This means all those detailed boot and service logs are stored in a temporary RAM directory (`/run/log/journal/`). The moment you reboot or shut down the machine, the logs disappear forever.
- make the journal persistent
```bash
# it doesn't matter if the file already exists or not
sudo vi /etc/systemd/journald.conf
```
- type the required section header and parameter
```bash
[Journal]
Storage=persistent
```
- save, exit, and restart the service
```bash
sudo systemctl restart systemd-journald.service
```
- generate the proper directory with correct user and ownership
```bash
sudo systemd-tmpfiles --create --prefix /var/log/journal
```
- flush the memory to the disk
Right now, the logs are still sitting in the temporary RAM cache. You need to explicitly tell the daemon to dump everything from RAM onto the newly configured disk space:
```bash
sudo journalctl --flush
```
- verify
```bash
ls -la /var/log/journal/
```
> You should instantly see a folder inside with a long string of letters and numbers (the system's unique Machine ID). That means it worked, and the journals are successfully being preserved!

## Start, stop, and check the status of network services ✅
Managing network services in RHEL 10 relies entirely on `systemctl`, the primary control interface for `systemd`.
- these are the regular, `start`, `stop`, and `restart`(after making major configuration changes to an application) operations.
- restart without dropping active connections with `reload` is service supports it
- the persistence operations (They do not start or stop the service immediately on their own)
    - enable a service to start at boot with `enable`
    - disable a service from starting at boot with `disable`
- instead of running `start` then `enable` as two seperate commands, you can combine them with:
```bash
sudo systemctl enable --now httpd
```
> this instantly stars Apache and sets it to auto-start on every future boot
- check status with `status`, when looking at the output, pay attention to:
    - `Loaded` -> shows if it's `enabled` or `disabled` for the next boot.
    The `vendor preset` (or just `preset:`) tells you the original out-of-the-box state that the developers intended for that service when the package was first installed.

        When you look at the `Loaded`: line, you will see a comparison between the current configuration and the default preset:
    ```bash
    Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
                                                      ↑                 ↑
                                            The Current Setting   Default Setting
    ```
    - `Active` -> shows if it's currently `active (running)` or `inactive (dead)`.
- change back to defaults
If you have heavily modified several services and want to quickly revert a specific service back to whatever Red Hat originally intended, you don't have to look up the documentation. You can tell `systemctl` to read that preset value directly:
```bash
sudo systemctl preset sshd
```
- the `mask` trap
If a service is completely broken, or if the exam wants to prevent a service from ever being started by another process or user, you can mask it. A masked service is linked to `/dev/null` and cannot be started manually or automatically until it is unmasked.
```bash
# completely lock down a service
sudo systemctl mask firewalld

# reverse it
sudo systemctl unmask firewalls
```

## Securely transfer files between systems ✅
First you have to connect the local and remote server with SSH
- generate `ssh` key on local machine
```bash
# hit enter to accept all defaults
ssh-keygen
```
> this creates a private key and a public key (ending in .pub) inside ~/.ssh/

- copy the public key to the remote server
ssh connection must be already established to the remote server with the use of password, and user must already exist in remote server (this key will make you connect without password in the future)
```bash
ssh-copy-id user@remote-server-ip
```
> you will be prompted to type the remote user's password one last time to authorize the transfer, the pub key will be added to remote servers authorized keys.

- test it
```bash
ssh user@remote-server-ip
```

- securely copy files or directories between servers
Once SSH is set up securely, you will use it as the transport layer to sync files between systems.
- using `scp` (secure copy)
great for simple, one-time file or dir transfers.
```bash
# copy local file to a remote server's /tmp dir
scp /path/to/file user@remote-server-ip:/tmp/

# copy a remote dir to local machine (requires -r)
scp -r user@remote-server-ip:/var/www/html/ ./local_backup/
```
- using `rsync` (remote sync)
`rsync` is highly preferred for large dirs because it only copies the diff between files, saving massive amounts of time and bandwidth.
```bash
# sync a local folder to a remote machine cleanly
# -a is for archive, -v is verbose, and -z is compress
rsync -avz /local/data/ user@remote-server-ip:/remote/backup/
```
> if you type `/local/data/` with a trailing slash, `rsync` copies the contents of the folder into the destination. However, if you type `/local/data` with no trailing slash, `rsync` copies the entire folder into the dest.
