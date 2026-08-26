# Manage security ✅


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

# Set temporary umask for the current terminal session
umask 0022

# Set persistent umask for current user
echo "umask 0022" >> ~/.bashrc
```
Set persistent umask for all users (including future ones):
```bash
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

To view the current SELinux status, use either of these commands:

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

1. Open `/etc/selinux/config` in the editor:
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

If you only run `setenforce 0`, the system will revert back to its old mode when the exam grading script reboots the machine!

**Always do BOTH for full credit:**

1. Update `/etc/selinux/config` (for persistence).
2. Run `setenforce <mode>` (so the current session matches immediately without needing a reboot).


## List and identify SELinux file and process context ✅
**Understanding SELinux contexts for files and processes is essential because most SELinux issues on the exam come down to standard tools (like `cp` or `tar`) creating files with the wrong label.**

SELinux uses labels (contexts) attached to processes and files to determine what a process is allowed to do.

### SELinux Context Format

A SELinux label consists of four fields separated by colons:

user:role:type:level/sensitivity

- **Example:** `unconfined_u:object_r:httpd_sys_content_t:s0`

### The Field That Matters for RHCSA: **Type (`type`)**

For the RHCSA exam, **99% of the focus will be on the Type field** (the third field, ending in `_t`, like `httpd_sys_content_t` or `ssh_home_t`).

- **Targeted Policy:** SELinux enforcement primary relies on **Type Enforcement**. If a process running with context type `httpd_t` tries to read a file with context type `user_home_t`, SELinux blocks it!

### Listing and Identifying File Contexts

Standard Linux utilities use the **`-Z`** flag to display SELinux security contexts.

- Viewing File & Directory Contexts (`ls -Z`)
```bash
ls -Z /var/www/html
```
> **Output example:**
```text
unconfined_u:object_r:httpd_sys_content_t:s0 index.html
```
> Here, `httpd_sys_content_t` is the file **type context**.


- View directory context itself (using -d)
```bash
ls -Zd /var/www/html
```
> or use the prev command with `ls -laZ` (the . is dir line)

### Listing and Identifying Process Contexts

Processes also have SELinux contexts. When a process runs, it inherits an operational type context.

- Viewing Running Process Contexts (`ps -efZ` or `ps auxZ`)

```bash
ps -efZ | grep httpd
```
> **Output example:**
```text
system_u:system_r:httpd_t:s0      root  1234  1  0  10:00 ?  00:00:00 /usr/sbin/httpd
```

- The process type is **`httpd_t`**.
- Because `httpd_t` has permission to read files labeled `httpd_sys_content_t`, Apache can serve Web pages stored under `/var/www/html`.

### Viewing Port Contexts (`semanage port -l`)

Services are also restricted to binding only to specific network ports.

```bash
# List all network port contexts
sudo semanage port -l

# Check ports allowed for HTTP web traffic
sudo semanage port -l | grep http_port_t
```
> **Output example:**
```text
http_port_t     tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
```

### Summary

| Command | Purpose |
| --- | --- |
| **`ls -Z /path`** | View file and directory SELinux contexts |
| **`ls -Zd /path`** | View directory SELinux context (without listing contents) |
| **`ps -efZ  grep <service>`** | View SELinux context of a running process |
| **`semanage port -l`** | View SELinux port context mapping |


## Restore default file contexts ✅
SELinux relies on file labels (contexts) to determine access permissions. If a file or directory has the wrong label, often caused by using `mv` instead of `cp`, services like Apache, SSH, or Samba will be blocked, even if standard Linux file permissions (chmod/chown) are correct.

### Understanding the Policy Database vs. Active Labels

SELinux maintains a policy database that defines what label every path on the system should have.

- Active Label: What is currently assigned to a file on disk.
- Policy Database: The master reference list (stored under `/etc/selinux/targeted/contexts/files/`).

Restoring default contexts means telling SELinux to check the master policy database and re-apply those defined default labels back onto your files.

### Key Tools
- View active SELinux context of files/directories
```bash
# View current active label on disk
ls -Z
```
- Query the policy database to see what context should be applied
```bash
# match path context (Check what SELinux policy expects for that path)
matchpathcon <full-path-to-file-or-dir>
```
- Modify or add rules to the SELinux policy database
```bash
# this is using full path (assume custom-web dir is in /)
sudo semanage fcontext -a -t httpd_sys_content_t "/custom-web(/.*)?"
```
> -a: add a rule, -t httpd_sys_content_t: Specify the SELinux type, "/custom-web(/.*)?": Regex ensuring the directory and everything underneath it gets matched (for files just use the full path).
- Apply the policy database rules to actual files/directories on disk
```bash
# restore contexts recursively with verbose output
# if you're restoring context for a single file, you don't need -R
sudo restorecon -Rv /custom-web
```
> -R: recursive (apply to subdirectories and files), -v: verbose

### Common Exam Pitfall: `mv` vs `cp`

When you create a file in `/tmp` or `/root` and move it with `mv`, it retains its original SELinux context (`user_tmp_t` or `admin_home_t`).

```bash
# Scenario: File moved into web directory keeps old label
mv /tmp/index.html /var/www/html/

# System blocks web server access because context is user_tmp_t!
# Fix it immediately with restorecon (it'll get the httpd context
# because it's under /var/www/...)
sudo restorecon -v /var/www/html/index.html
```

> Tip: Using `cp` generates a new file, which automatically inherits the parent directory's default context. Using `mv` carries over the source file's context. Always run `restorecon -Rv` on target directories when moving files around.


## Manage SELinux port labels ✅
Just like SELinux uses file contexts to restrict which files a process can read, it uses port labels to control which network ports a service is allowed to bind to (listen on).

### Why Manage Port Labels?
By default, SELinux allows network services to run only on their standard, well-known ports.
- **Example**: Apache (httpd) is allowed to bind to TCP port 80 or 443 (http_port_t).
- **The Problem**: If you configure Apache to listen on a non-standard port (like port 8088), SELinux will block Apache from starting, throwing a "Permission Denied" error during socket binding, even if firewalld/iptables allows port 8088!

### Key Commands for the Exam

| Command | Purpose |
| --- | --- |
| `semanage port -l` | List all existing port label definitions in SELinux. |
| `semanage port -a -t <type> -p <proto> <port>` | **Add** a new port to a specific SELinux type. |
| `semanage port -m -t <type> -p <proto> <port>` | **Modify** an existing port binding to belong to a new type. |
| `semanage port -d -t <type> -p <proto> <port>` | **Delete** a custom port definition you added. |


### The RHCSA Port Label Workflow
Here is how you tackle SELinux port assignments in the exam step-by-step.

1. **Identify Existing Port Types:** Find the correct SELinux port type.
Check what port types exist for your service (e.g., HTTP, SSH, Samba) using `semanage port -l` filtered with `grep`:
```bash
# Find all SELinux port rules related to HTTP
sudo semanage port -l | grep http_port_t
```
*Output example:*
`http_port_t   tcp    80, 81, 443, 488, 8008, 8009, 8443, 9000`

2. **Assign the New Port to the SELinux Type:** Allow the service on a non-standard port.
Suppose your exam task asks you to configure web services (`httpd`) to listen on **TCP port 8088**. You must inform SELinux about this custom port:
```bash
# Add (-a) TCP (-p tcp) port 8088 to type (-t) http_port_t
sudo semanage port -a -t http_port_t -p tcp 8088
```

3. **Verify the Port Mapping:** Ensure SELinux accepted the update.
Run the list command again to confirm your port is listed under the target type:
```bash
sudo semanage port -l | grep http_port_t
```

#### Extra
> Allowing a port in SELinux **does not** automatically open it in your firewall! On the RHCSA exam, non-standard service configurations usually require **both**:
> 1. `semanage port -a -t ...` (allows the local daemon to bind to the port)
> 2. `firewall-cmd --add-port=... --permanent` (allows external network traffic through)


## Use Boolean settings to modify system SELinux settings ✅
SELinux Booleans are simple on/off switches (true/false, 1/0) built into the SELinux policy. They allow you to tweak security behavior on the fly, like permitting web servers to read user home directories or send emails, without writing custom policy modules or editing complex code.

### Key Commands for the Exam

| Command | Purpose |
| --- | --- |
| `getsebool -a` | List all available Booleans and their current state (on/off). |
| `getsebool <boolean_name>` | View the status of a specific Boolean. |
| `semanage boolean -l` | List Booleans with descriptions explaining what each switch does. |
| `setsebool <boolean> <on-or-off>` | Change a Boolean state temporarily (resets upon reboot). |
| `setsebool -P <boolean> <on-or-off>` | Change a Boolean state permanently (persists across reboots). |

### The RHCSA Boolean Workflow
Here is how you handle SELinux Booleans step-by-step during the exam.

1. Locate the Relevant Boolean

You usually won't remember exact Boolean names. Use getsebool -a paired with grep to find the one matching your service (e.g., httpd, ftp, samba, nfs).
```bash
# Find all web server (httpd) related Booleans
getsebool -a | grep httpd
```
If you aren't sure what a Boolean does, use semanage to view detailed descriptions:
```bash
sudo semanage boolean -l | grep httpd_enable_homedirs
```

2. Inspect Current State

Check whether the target Boolean is currently on or off:
```bash
getsebool httpd_enable_homedirs
```
> Output (it's off by default): httpd_enable_homedirs --> off

3. Enable/Disable the Boolean with `-P` (Permanent)

To allow Apache to serve user home directories, toggle the Boolean to on:
```bash
# The -P flag writes the change permanently to disk
# always use it so the change survives reboot
sudo setsebool -P httpd_enable_homedirs on
```
> Note: Running `setsebool -P` can take 5 to 15 seconds to finish because it recompiles parts of the policy in memory.

4. Verify the Update
```bash
getsebool httpd_enable_homedirs
```
> Output: httpd_enable_homedirs --> on

### High-Frequency Exam Booleans

Keep an eye out for these commonly tested service requirements on the RHCSA exam:

| Service Goal | Boolean Name |
| --- | --- |
| Allow Apache (httpd) to access user /home directories | httpd_enable_homedirs |
| Allow NFS shares to be mounted and written to | nfs_export_all_rw |
| Allow Apache to send outbound network connections/emails | httpd_can_network_connect |
| Allow FTP to write/upload files | ftpd_full_access |
| Allow Samba (smb) to share user home directories | samba_enable_home_dirs |
