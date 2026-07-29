# Manage security


## Configure firewall settings using firewall-cmd/firewalld ✅

### Firewalld Zones Overview
Firewalld uses zones to assign trust levels to network interfaces.
- **public**: Default zone. Disallows incoming connections to services unless explicitly allowed (allows pings).
- **trusted**: Accepts all incoming network connections.
- **drop**: Drops all incoming network packets without replying (silent drop, times out clients).
- **block**: Rejects all incoming network connections including basic pings (returns ICMP rejected message).

### Managing Active Zones & Interfaces
You can bind specific network interfaces or IP source ranges to different zones depending on security requirements.
```bash
# Check current default zone and active interfaces
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --get-active-zones

# Change default zone
sudo firewall-cmd --set-default-zone=public

# Assign a specific interface (e.g., eth1) to a stricter zone
# everything that comes through that interface will be dropped
sudo firewall-cmd --permanent --zone=drop --add-interface=eth1
sudo firewall-cmd --reload
```

### Restricting Access by IP Address or Subnet
Security prompts often ask to either block a specific IP address or allow traffic ONLY from a specific IP address.
#### Explicitly Block/Reject a Host (Using drop or block Zone)
If asked to block all traffic from 10.0.0.5:
```bash
sudo firewall-cmd --permanent --zone=drop --add-source=10.0.0.5
sudo firewall-cmd --reload

# verify
sudo firewall-cmd --zone=drop --list-all
```
> now if a connection comes from that ip, it'll be dropped because we have added it to the 'drop' zone, it won't matter if the default zone is anything else like public.

#### Restrict Service Access to an IP (Rich Rules)
If asked to allow web traffic (port 80) only from subnet 192.168.1.0/24:
```bash
sudo firewall-cmd --permanent --add-rich-rule'rule family="ipv4" source address="192.168.1.0/24" port port="80" protocol="tcp" accept'
```
simpler way to do it:
```bash
sudo firewall-cmd --permanent --add-rich-rule'rule family="ipv4" source address="192.168.1.0/24" service name="http" accept'
```
reload:
```bash
sudo firewall-cmd --reload
```

### Port Forwarding / Masquerading
For security hardening, you might be asked to redirect incoming traffic on one port to another internal port (e.g., forward incoming port 80 to port 8080).
```bash
# Enable masquerading (required for port forwarding)
sudo firewall-cmd --permanent --add-masquerade

# Forward incoming TCP traffic from port 80 to port 8080
sudo firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080

# apply changes
sudo firewall-cmd --reload
```
> You can test and practice with this by installing an apache server(httpd pkg), then setting it to listen on port 8080 (default is 80) by changing apache config in `/etc/httpd/conf/httpd.conf` then restarting `httpd.service`. Use the above commands to do the port forwarding and test connection from a remote device that you have a connection to by doing `curl <host-device-ip-addr>` on remote device.


## Manage default file permissions ✅

The umask (user file-creation mask) defines which permissions are subtracted from the base maximum when a new file or directory is created.

Default max file permissions are 666 which is rw-rw-rw (you have to add 'x' to a file if you want it to be executable)
Default max directory permissions are 777 which is rwxrwxrwx ('x' here allows 'cd' to work)

umask calculation:
Directory Permission = 777 - umask
File Permission = 666 - umask

Example: **umask 027**
- Directory: 777 - 027 = 750 (rwxr-x---)
    - owner gets rwx(7), group gets r-x(5), others get ---(0).
- File: 666 - 027 = 640 (rw-r-----)
    - owner gets rw-(6), group gets r--(4), others get ---(0).
> Note: If subtraction results in an odd number for files, execution is stripped because files never start with execution privileges

You subtract umask values digit-by-digit (positionally) for each permission category, not as a standard 3-digit decimal number.

So the calculation for 666-027 went like this:
- User Position (First Digit): 6 - 0 = 6
- Group Position (Second Digit): 6 - 2 = 4
- Others Position (Third Digit): 6 - 7
    - In binary/octal permissions, 6 is rw- (4 + 2 + 0).
    - The umask 7 is rwx (4 + 2 + 1), which masks/removes all three permissions.
    - Subtracting rwx from rw- leaves **0** (---).

- commands to manage `umask`
```bash
# Check current umask in octal
umask

# Set temporary umask for your current terminal session
umask 0022

# Set persistent umask for current user
echo "umask 0022" >> ~/.bashrc
```
Set persistent umask for all users (including future ones):
```bash
#
sudo vi /etc/bashrc
```
add this line at the end to set default permissions system-wide:
```bash
umask 0022
```

[Notes for Special Permissions (SUID, SGID, Sticky Bit)](https://github.com/ibrahimmoalim/rhel_RHCSA_Exam_Prep#especial-permissions-suid-sgid-sticky-bit)


## Configure key-based authentication for SSH ✅

- Generate `ssh` key on client machine
```bash
# hit enter to accept all defaults
ssh-keygen
```
> this creates a private key and a public key (ending in .pub) inside ~/.ssh/

- Copy the public key to the remote server
remote server must be reachable via SSH with the use of password, and user must already exist in remote server (this key will make you connect without password in the future)
```bash
ssh-copy-id user@remote-server-ip
```
> you will be prompted to type the remote user's password one last time to authorize the transfer, the pub key will be added to remote servers authorized keys.

- Test it
```bash
ssh user@remote-server-ip
```

- Disable Password Authentication (Server Hardening)

Exam questions often ask you to enforce SSH key authentication by disabling password logins altogether on the server.

Open the main SSH daemon config:
```bash
# don not confuse it with another file: /etc/ssh/ssh_config
# the real one has 'd' "sshd_config"
sudo vi /etc/ssh/sshd_config
```
Ensure the following directives are set:
```bash
PasswordAuthentication no
PubkeyAuthentication yes
```
Apply Changes:
```bash
# test config syntax first
# if nothing is returned you're all good.
sudo sshd -t

# restart the SSH service
sudo systemctl restart sshd
```

- If you encounter any erros:
    - make sure the `~/.ssh/` dir has drwx------
    - make sure the `~/.ssh/authorized_keys` file has -rw-------
    - check if the remote server has SSH on a different port, if it has, then use: `ssh-copy-id -p <port> user@remote-ip`


## Set enforcing and permissive modes for SELinux ✅

### The Three SELinux Modes

| Mode | Behavior | Exam Impact |
| --- | --- | --- |
| **`Enforcing`** | Enforces security policy. Access is denied and logged if a violation occurs. | **Default RHEL state.** Required on the exam unless stated otherwise. |
| **`Permissive`** | SELinux policy is **not** enforced, but violations are still logged. | Great for troubleshooting issues without turning off SELinux completely. |
| **`Disabled`** | SELinux is completely turned off. | **Never do this on the exam!** Disabling requires a reboot to re-enable, and relabeling the filesystem takes time. |

### Checking the Current Mode

To view your current SELinux status, use either of these commands:

```bash
# Quick check (returns: Enforcing, Permissive, or Disabled)
getenforce

# Detailed report (shows current mode, config file mode, policy name, etc.)
sestatus
```

### Changing Modes Temporarily (Runtime Only)

To switch modes immediately without rebooting (resets upon system restart):

```bash
# Set mode to Permissive (0)
sudo setenforce 0
# OR
sudo setenforce Permissive

# Set mode to Enforcing (1)
sudo setenforce 1
# OR
sudo setenforce Enforcing
```

### Changing Modes Persistently (Survives Reboot)

For changes to survive a system reboot, you **must edit the configuration file**.

#### Configuration File Path

`/etc/selinux/config` *(Note: `/etc/sysconfig/selinux` is a symlink to this file).*

#### How to Edit

1. Open `/etc/selinux/config` in your editor:
```bash
sudo vi /etc/selinux/config
```

2. Modify the `SELINUX=` line:
```text
SELINUX=enforcing
# OR
SELINUX=permissive
```

3. Save and exit.

4. Check SELinux Status with `sestatus`, check the lines **Current mode** (shows what you set with `sudo setenforce`and currently using) and **Mode from config file** (shows persistent mode, which will be used after reboot even if not currently set).

### Exam Tip
A very common RHCSA task asks you to set SELinux to **`permissive`** or **`enforcing`**.

If you only run `setenforce 0`, the system will revert back to its old mode when the exam grading script reboots your machine!

**Always do BOTH for full credit:**

1. Update `/etc/selinux/config` (for persistence).
2. Run `setenforce <mode>` (so the current session matches immediately without needing a reboot).
