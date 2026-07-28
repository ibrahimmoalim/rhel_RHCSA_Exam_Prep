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

## Manage default file permissions

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
