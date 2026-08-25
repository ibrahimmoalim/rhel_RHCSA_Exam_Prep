# Deploy, configure, and maintain systems ✅

## Schedule tasks using at, cron and systemd timer units ✅
Automating task execution is a core administration skill. On the exam, you'll be expected to schedule jobs using three distinct mechanisms: one-off execution (`at`), traditional recurring schedules (`cron`), and modern service triggers (`systemd timers`).

### at
Use `at` when you need a job to execute exactly once at a specific time in the future.
- install `at` if missing
```bash
sudo dnf install at -y
```
- enable and start it
```bash
sudo systemctl enable --now atd
```
- create an `at` job

specify the time, enter interactive mode, type commands, and exit with `ctrl+d`
```bash
# this will open interactive mode
# do 'sudo at 12:00' if you want to run commands as root in the future
at 12:00
# type commands to run at 12:00
echo "hello" > hello.txt
# save and exit with [ctrl+d]
```
- handy time syntax examples:
    - `at 02:00 PM`
    - `at now + 5 minutes`
    - `at now + 3 days`
    - `at 9:00 AM tomorrow`
    - `at 4:00 PM Jul 25`
- managing `at` jobs (do sudo before these to manage jobs owned by root)
```bash
atq                # list all pending jobs (shows job ID, date, time)
at -c <job_id>     # view the script/commands inside that job
atrm <job_id>      # delete a scheduled job
```

### cron
`cron` is used for recurring scheduled jobs (e.g., every midnight, every Monday at 3 AM).

To edit your user's crontab safely, always use `crontab -e` (never edit `/etc/crontab` directly unless explicitly asked).
- run `crontab -e` to schedule cron tasks for current user

It'll be opened in an editor
```bash
# ┌───────────── minute (0 - 59)
# │ ┌─────────── hour (0 - 23)
# │ │ ┌───────── day of month (1 - 31)
# │ │ │ ┌─────── month (1 - 12)
# │ │ │ │ ┌───── day of week (0 - 6) (0 is Sunday)
# │ │ │ │ │
# * * * * * <command_to_execute>
```
- practical examples:
    - run every day at 2:30 AM
    ```bash
    30 2 * * * /usr/bin/tar -czf /var/backups/data.tar.gz /data
    ```
    - run every 15 minutes during work hours (8 AM - 4 PM) on weekdays
    ```bash
    */15 8-15 * * 6,0-4 <command>
    ```
    - run hourly/daily/weekly/monthly/yearly, e.g daily:
    ```bash
    @daily <command>
    ```
- managing crontabs
```bash
crontab -e       # edit current user's crontab
crontab -l       # list current user's cron jobs
crontab -r       # wipe all cron jobs for the current user

# edit another user's crontab as root
sudo crontab -u ali -e
sudo crontab -u ali -l
```

### modern scheduling with systemd timers
Red Hat is pushing heavily toward systemd timer units because they offer better logging via **journalctl**, can trigger on system events, and don't require an active user session.

A systemd timer always requires **two files** in `/etc/systemd/system/`
1. A **service unit** (`.service`), which defines **what** to execute
2. A **timer unit** (`.timer`), which defines **when** to trigger the service

- if you were to run: `/usr/bin/logger "hello"` every sunday at 8:00 AM, you'd follow these steps:
- create the service unit (`/etc/systemd/system/job_test.service`)
```Ini
[Unit]
Description=A Hello Log

[Service]
Type=oneshot
ExecStart=/usr/bin/logger "hello"
```
- create the matching timer unit (`/etc/systemd/system/job_test.timer`)

> Note: the filename prefix MUST match the service name (e.g., job_test.timer targets job_test.service).
```Ini
[Unit]
Description=Run Job Test Every Sunday Morning

[Timer]
# Format: DayOfWeek Year-Month-Day Hour:Minute:Second
OnCalendar=Sun *-*-* 08:00:00
# if the server is down when time is reached, 'Persistent=true' ensures
# systemd triggers the missed job immediately the next time the machine boots up.
Persistent=true

[Install]
# Allows the timer to start automatically when the system boots.
WantedBy=timers.target
```
- enable and start the timer
```bash
# reload systemd to recognize the new unit files
sudo systemctl daemon-reload

# enable and start ONLY the .timer unit (NOT the .service)
sudo systemctl enable --now job_test.timer
```
> If you get an error trying to use a timer targeting specific days, make sure syntax is correct in both files and validate the OnCalendar syntax with:
```bash
# this is sat-fri but because this calendar starts weekdays on mon, we have to do mon..thu first
# (the two dots mean mon through thu) then we do sat and sun. this only skips fri.
# we cannot say sat..thu directly.
systemd-analyze calendar "Mon..Thu,Sat,Sun *-*-* 08:00:00"
```
- verify timers
```bash
# list all active timers
systemctl list-timers

# check timer status
systemctl status job_test.timer

# check if the execution output succeeded via journalctl
sudo journalctl -u job_test.service
```

## Start and stop services and configure services to start automatically at boot ✅
This objective revolves almost entirely around **`systemctl`**, the primary utility used to control the `systemd` system and service manager.

- Core Service Management Commands

To manage services on a running system, use the following `systemctl` actions:
```bash
# Start a service immediately
sudo systemctl start <service_name>

# Stop a running service
sudo systemctl stop <service_name>

# Restart a service (stops then starts it again, useful after editing config files)
sudo systemctl restart <service_name>

# Reload service configuration without stopping it (if supported by the service)
sudo systemctl reload <service_name>

# Check detailed status, PID, and recent log outputs of a service
systemctl status <service_name>
```

- Boot-Time Configuration (Enabling/Disabling)

Controlling whether a service starts automatically when the system boots is distinct from starting or stopping it right now:
```bash
# Enable a service to start automatically at boot
sudo systemctl enable <service_name>

# Disable a service from starting automatically at boot
sudo systemctl disable <service_name>

# Combined shortcut: Start the service NOW AND enable it for boot in one command
sudo systemctl enable --now <service_name>

# Combined shortcut: Stop the service NOW AND disable it for boot
sudo systemctl disable --now <service_name>
```

- Verification & Troubleshooting Commands

On the RHCSA exam, verifying state quickly is key. Use these fast check options:
```bash
# Check if a service is currently active (running) -> returns "active" or "inactive"
systemctl is-active <service_name>

# Check if a service is enabled to start at boot -> returns "enabled" or "disabled"
systemctl is-enabled <service_name>

# Check if a service failed during startup
systemctl is-failed <service_name>

# List all currently active services on the system
systemctl list-units --type=service --state=running

# List all installed services and whether they are enabled/disabled for boot
systemctl list-unit-files --type=service
```

- Masking Services (Exam Trap Alert!)

Sometimes an exam task or troubleshooting question will ask you to completely prevent a service from starting even if another service or administrator tries to start it manually.

```bash
# Mask a service (links unit file to /dev/null so it cannot be started by any means)
sudo systemctl mask <service_name>

# Try starting a masked service:
sudo systemctl start <service_name>
# Output: Failed to start <service_name>.service: Unit is masked.

# Unmask a service to allow standard operations again
sudo systemctl unmask <service_name>
```

- Summary

| Task | Command |
| --- | --- |
| **Start right now** | `systemctl start <service>` |
| **Stop right now** | `systemctl stop <service>` |
| **Enable on boot** | `systemctl enable <service>` |
| **Disable on boot** | `systemctl disable <service>` |
| **Start AND Enable** | `systemctl enable --now <service>` |
| **Prevent starting completely** | `systemctl mask <service>` |

## Configure systems to boot into a specific target automatically ✅
A target is essentially a group of system services bundled together to achieve a specific operating state.

- The two main targets:

1. **multi-user.target** (formerly Runlevel 3): Non-graphical, command-line interface (CLI) mode. Loads all networking and multi-user services, but no GUI.
2. **graphical.target** (formerly Runlevel 5): Full graphical user interface (GUI) desktop mode. Includes everything in **multi-user.target** plus the display manager and graphical desktop.

- Checking the current target

To inspect your system's current default target and running target state:
```bash
# Display the target configured to load at boot
systemctl get-default

# Check the targets currently loaded and active on the system
systemctl list-units --type=target

# See all targets available to change to
systemctl set-default <double-tab-to-see-targets>
```

- Change the default boot target (Persistent)

To change which target the machine boots into automatically upon restart, use set-default:
```bash
# Set default boot target to CLI (multi-user)
sudo systemctl set-default multi-user.target

# Set default boot target to GUI (graphical)
sudo systemctl set-default graphical.target

# reboot for it to take effect
sudo reboot
```

- Verify

Run `systemctl get-default` again to confirm it shows the expected target.
> How it works under the hood: `systemctl set-default` simply updates a symbolic link at `/etc/systemd/system/default.target` pointing to the target file in `/usr/lib/systemd/system/`

- Switch target immediately (no reboot required, non-persistent)
```bash
# Switch current running state to multi-user (CLI)
sudo systemctl isolate multi-user.target

# Switch current running state to graphical (GUI)
sudo systemctl isolate graphical.target
```

- Other important systemd targets to know (while `multi-user` and `graphical` are the primary ones you will configure, you should recognize these for recovery and management):
    - `rescue.target`: Single-user recovery mode. Mounts root filesystem read-only or read-write with minimal services for repair.
    - `emergency.target`: Absolute bare-minimum shell. Root filesystem is mounted read-only, no network, used when `fstab` or storage fails completely.
    - `reboot.target`: System reboot.
    - `poweroff.target`: System shutdown.

## Configure time service clients ✅
On RHEL, time synchronization is handled by **chrony** (via the chronyd service). You need to know how to verify NTP synchronization, configure NTP (Network Time Protocol) servers, and manually adjust system time/timezone using timedatectl and chronyc.

- Primary utilities overview
    - `timedatectl`: Manages system time, date, timezones, and overall NTP enablement.
    - `chronyc`: The command-line interface to monitor and manage the chronyd daemon.
    - `/etc/chrony.conf`: The main configuration file for chrony where NTP time servers are defined.

- Managing timezones and local clock (timedatectl)
```bash
# check time and NTP status
timedatectl status
```
Look for: `System clock synchronized: yes` and `NTP service: active`

- Listing and setting timezones
```bash
# list available timezones matching a region
timedatectl list-timezones | grep -i africa

# set the system timezone
sudo timedatectl set-timezone Africa/Mogadishu
```

- Enable/Disable Network Time Synchronization
```bash
# enable automatic NTP synchronization
sudo timedatectl set-ntp true

# Disable NTP synchronization (required if setting time manually)
sudo systemctl stop chronyd
sudo timedatectl set-ntp false
sudo timedatectl set-time "2026-07-25 10:30:00"
```

- Configure NTP client servers (`/etc/chrony.conf`)

On the exam, you may be given an NTP server hostname or IP address (e.g., time.example.com or 192.168.55.254) and asked to configure your machine to synchronize with it.
- edit `/etc/chrony.conf`
```bash
sudo vi /etc/chrony.conf

# add the provided NTP server using the server directive
# in the middle is hostname or ip-addr provided
# iburst option is for fast initial sync
server time.example.com iburst
```
> comment out or remove any default pool or server lines if the prompt asks to use ONLY the specified server

- Restart and enable chronyd
```bash
sudo systemctl restart chronyd
sudo systemctl enable chronyd
```

- Verify synchronization (`chronyc`)

Once configured, verify your client is talking to the remote NTP server using chronyc:
```bash
# check NTP sources
chronyc sources -v
```
Look for:
- **^***: The asterisk * next to a server IP/hostname means chronyd has selected it as its current active synchronized source.
- **^+**: A plus sign means it's an acceptable candidate source.
- **^?**: A question mark means connectivity is lost or reachability checks failed (check network/firewall).

- Check synchronization tracking details
```bash
chronyc tracking
```
> Confirms reference server name, stratum, and system offset.

## Install and update software packages from Red Hat Content Delivery Network, a remote repository, or from the local file system ✅
Package management in RHEL is handled via dnf (Dandified YUM) and the lower-level rpm utility. On the exam, you will be expected to install software from standard registered channels, custom remote repositories, or local standalone .rpm files while automatically resolving dependencies.

### Inspecting and managing repositories
- list active repos
```bash
dnf repolist
```
- list all enabled and disabled repos
```bash
dnf repolist all
```
- configure custom remote repositories

repository definition files live in `/etc/yum.repos.d/`. To add a remote repo, create a `.repo` file:
```Ini
[custom-remote]
name=Custom remote repo
baseurl=<http-link-provided-in-exam>
enabled=1
# make this zero if no gpg-key specified
gpgcheck=1
# don't add below line if no gpgkey specified
gpgkey=file:///etc/pki/rpm-gpg/....
```
- clear dnf cache (useful if metadata gets corrupted)
```bash
sudo dnf clean all
sudo dnf makecache
```

### Installing and updating packages via dnf
Whether pulling from the Red Hat Content Delivery Network (CDN) or a custom remote repository, the syntax remains the same.

- search for a pkg or file provider
```bash
dnf search <pkg-name>
# if you know the command but not the package name
dnf provides */<command>
```
- install a pkg
```bash
sudo dnf install <pkg-name> -y
```
- update/remove
```bash
# update a specific pkg
sudo dnf update <pkg-name>
# update all system pkgs
sudo dnf update -y
# remove a pkg
sudo dnf remove <pkg-name>
```

### Installing from the local file system
The exam will often drop a raw `.rpm` file somewhere on your hard drive (e.g., in `/tmp` or a mounted directory) and ask you to install it.

To practice for this on a VM, download a pkg into a dir and then install it locally:
```bash
# for example download tree (if you don't have it already)
sudo dnf download --destdir=/tmp tree
```
```bash
# verify it's there
ls -la /tmp/*.rpm
```
> You'll see something like /tmp/tree-1.8.0-10.el9.x86_64.rpm

- there are two methods of doing this:
    - `dnf`: dnf is always preferred because if the local package requires other software dependencies, dnf will automatically scan your repositories, download the missing pieces, and install everything cleanly.
    ```bash
    sudo dnf install /path/to/pkg-name.rpm
    ```
    - `rpm (lower-lvl tool)`: The rpm command installs a package directly, but it will fail if there are unmet dependencies, throwing an error instead of fetching them for you.
    ```bash
    # Install or upgrade a local RPM package
    sudo rpm -ivh /path/to/file/package_name.rpm

    # Useful rpm flags:
    # -i : Install
    # -v : Verbose output
    # -h : Hash marks (progress bar)
    # -U : Upgrade an existing package (or install if not present)
    # -q : Query a package (e.g., rpm -qPi file.rpm to inspect it)
    ```

## Modify the system bootloader ✅
On RHEL, managing persistent kernel options and GRUB settings used to mean editing /etc/default/grub and running grub2-mkconfig. While that still works, Red Hat's standard, official tool for RHCSA is grubby. grubby makes direct, safe bootloader changes without risking broken syntax in complex configuration files.

### grubby
- Inspecting Bootloader Settings (grubby)

Before modifying anything, always check what kernel is currently set as default and what arguments it uses.
```bash
# Check the default kernel path
sudo grubby --default-kernel

# Display detailed info (kernel path, index, args) for the default kernel
sudo grubby --info=DEFAULT

# List information for ALL installed kernels
sudo grubby --info=ALL
```

- Adding or Removing Kernel Parameters

A common exam task is adding a persistent boot parameter (like enabling serial console logs, disabling quiet boot, or adding custom security flags).

Common args:
- `console=ttyS0` or `console=tty0`: Directs boot messages to a specific serial port or screen output.
- `quiet`: Suppresses non-critical kernel log messages during boot (gives a clean boot screen).
- `rhgb`: Enables Red Hat Graphical Boot (shows the splash screen instead of scrolling text).
- `systemd.debug-shell=1`: Spawns a root shell on terminal 9 (Ctrl+Alt+F9) for debugging boot failures.
- `selinux=0` or `enforcing=0`: Disables or sets SELinux state at boot.
- `audit=1`: Enables kernel-level auditing.

Adding Parameters (--args), to add a new parameter to the default kernel:
```bash
sudo grubby --update-kernel=DEFAULT --args="console=ttyS0"
```
Removing Parameters (--remove-args), to remove an existing parameter from the default kernel:
```bash
# if u use 'ALL', it'll update for all installed kernels
sudo grubby --update-kernel=ALL --remove-args="quiet rhgb"
```
Verification step:
```bash
# look at args and see if changes were applied
sudo grubby --info=DEFAULT
```

- Changing the Default Kernel

If a kernel update broke system functionality or the exam asks you to set an older/newer installed kernel as default:
```bash
# First, list all kernels to find the full path of the target kernel
sudo grubby --info=ALL | grep ^kernel

# Set the default kernel using its full path
sudo grubby --set-default=/boot/vmlinuz-5.14.0-xxx.el9.x86_64
```

### /etc/default/grub
While grubby handles kernel flags and defaults, if you need to change the **boot menu timeout** (how many seconds GRUB waits before auto-booting), you edit /etc/default/grub:
1. Open /etc/default/grub:
```bash
sudo vi /etc/default/grub
```
2. Modify or add the timeout line:
```
GRUB_TIMEOUT=10
```
3. Rebuild the GRUB configuration to save changes persistently:
```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
> Tip: On RHEL 8/9/10, `grub2-mkconfig -o /etc/grub2.cfg` works on both BIOS and UEFI because `/etc/grub2.cfg` is a **symlink** to the correct location.

> Note: `grub2-mkconfig` is only required after modifying `/etc/default/grub` to make the changes persistent, not after using `grubby` (grubby writes the changes directly into the active GRUB configuration file on disk immediately.)
