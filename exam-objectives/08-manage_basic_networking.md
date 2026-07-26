# Manage basic networking ✅

## Configure IPv4 and IPv6 addresses ✅
On RHEL, network configuration is managed by NetworkManager using the nmcli command.

- check the exact name of active connection profile
```bash
nmcli connection show
```
> Look at the NAME column (e.g., ens3, eth0, or Wired connection 1). Use that exact name in the commands below.

### configure a static ipv4 address
To assign a static IPv4 address, subnet mask (in CIDR notation), default gateway, and DNS servers:
```bash
sudo nmcli connection modify <connection-name> ipv4.addresses 192.168.120.100/24 ipv4.gateway 192.168.120.1 ipv4.dns "1.1.1.1 8.8.8.8" ipv4.method manual
```
Breakdown of Flags:
- `ipv4.addresses 192.168.120.100/24`: Sets the IP and /24 subnet mask (255.255.255.0).
- `ipv4.gateway 192.168.120.1`: Sets the default router/gateway IP.
- `ipv4.dns "8.8.8.8 1.1.1.1"`: Specifies DNS servers (space-separated inside quotes).
- `ipv4.method manual`: Switches the connection from DHCP (auto) to Static (manual).

#### Apply the Changes
Modifying a connection with nmcli updates the configuration file on disk, but it does not apply to the running network adapter immediately. You must reactivate the profile:
```bash
sudo nmcli connection down <connection-name> && sudo nmcli connection up <connection-name>
```
> `sudo nmcli connection up <connection-name>` alone works too, but if it doesn't, do `...down..` first.

### configure a static ipv6 address
**The exam prompt will explicitly give you the exact IP, Gateway, and DNS values to use** (e.g., "Configure enp1s0 with IPv6 address 2001:db8:1::10/64, Gateway 2001:db8:1::1, and DNS 2001:db8:1::254").

The syntax for IPv6 uses the exact same structure with ipv6 parameters:
```bash
sudo nmcli connection modify <connection-name> ipv6.method manual ipv6.addresses 2001:db8:1::10/64 ipv6.gateway 2001:db8:1::1 ipv6.dns "2001:db8:1::254"
```
```bash
# apply changes
sudo nmcli connection up <connection-name>
```

### Verify Your Configuration
Always double-check your IP configuration using these verification commands:
```bash
# Check IPv4 address and interface state
ip -4 addr show <connection_name>

# Check IPv6 address
ip -6 addr show <connection_name>

# Check active default gateways
ip route show

# Check route for IPv6
ip -6 route show
```

- Switching Back to DHCP (If Required by an Exam Prompt)

If the exam asks you to set an interface back to acquire IPs dynamically via DHCP:
```bash
sudo nmcli connection modify <connection_name> ipv4.method auto ipv6.method auto

sudo nmcli connection down <connection_name> && sudo nmcli connection up <connection_name>
```

## Configure hostname resolution ✅
Hostname resolution is how Linux translates human-readable hostnames (like server1.example.com) into IP addresses (like 192.168.122.100). On the exam, you need to handle both system-level hostname assignment and local name resolution mapping.

### Setting the System Hostname (hostnamectl)
To persistently set or change the hostname of your machine:
```bash
# View current hostname details
hostnamectl status

# Set the fully qualified domain name (FQDN)
sudo hostnamectl set-hostname server1.example.com
```
```bash
# run this to update immediately into the new hostname without
# rebooting or logging-out and back in
exec bash
```

### Local Hostname Resolution (/etc/hosts)
When DNS is unavailable or you need to force a local IP-to-hostname mapping on the machine itself, you edit `/etc/hosts`, this allows you to run `ssh username@hostname` instead of `ssh username@ip-addr`.

Usually, an exam prompt will state something like: **"Configure local name resolution so that 192.168.122.200 resolves to server2.example.com with an alias of server2."**

Open `/etc/hosts` file with sudo:
```bash
sudo vi /etc/hosts
```
Add this line:
```bash
# the format is:
# ip-addr   full-domain         short-name(alias)
192.168.x.x server2.example.com server2
```

### Verifying Hostname Resolution
To test if your resolution configuration works:
```bash
# Test local query resolution
getent hosts server2

# Test reachability (-c is count, it'll send two pings)
ping -c 2 server2
```

## Configure network services to start automatically at boot ✅
This objective ensures that essential networking daemons and individual connection profiles persist and initialize properly whenever the system boots.

- enable network services via systemd

System services (like NetworkManager, sshd, or firewalld) must be enabled in systemd to start at boot time.
```bash
# check if a service is enabled to start at boot
systemctl is-enabled NetworkManager

# enable and start NetworkManager immediately
sudo systemctl enable --now NetworkManager

# do same for SSH service
sudo systemctl enable --now sshd
```

- set network connections to auto-connect at boot via nmcli

Even if NetworkManager is enabled at boot, individual network adapters/connections must have auto-connect active, or the interface won't bring itself up when the machine powers on.
```bash
# check if a connection is set to auto-connect at boot
nmcli connection show <con-name> | grep connection.autoconnect

# set a connection to start automatically at boot
sudo nmcli connection modify <con-name> connection.autoconnect yes

# disable auto-connect (if asked by exam) for unused connections
sudo nmcli connection modify <con-name> connection.autoconnect no
```

- wait for Network at boot (NetworkManager-wait-online)

On servers running network-dependent services (like NFS mounts or web servers), the system must wait until the network interface is **fully online** before starting dependent services.
```bash
# Enable the wait-online service so network-dependent
# units don't fail at boot
sudo systemctl enable NetworkManager-wait-online.service
```

## Restrict network access using firewalld and firewall-cmd ✅
RHEL uses firewalld as its dynamic firewall daemon, which is managed using the `firewall-cmd` CLI utility. It uses the concept of zones (default is usually public) to manage allowed traffic.

Firewall rules created without `--permanent` only exist in temporary memory and will vanish upon reboot.

Always run your firewall-cmd commands with `--permanent`(when you're adding or removing a service/port/rich-rule), and then run `sudo firewall-cmd --reload` to apply them to the runtime configuration!

- Before adding or removing rules, inspect what is currently allowed:
```bash
# Check if firewalld is running
sudo firewall-cmd --state

# Show the default active zone and all allowed services/ports
sudo firewall-cmd --list-all
```

[More firewall-cmd Notes](https://github.com/ibrahimmoalim/rhel_RHCSA_Exam_Prep/tree/main#firewall-cmd)

If an exam task asks to restrict access to a specific IP or subnet (e.g., "Allow SSH access ONLY from 192.168.122.50"), use a Rich Rule:
```bash
# first remove the global SSH service if it's active on the default zone
sudo firewall-cmd --permanent --remove-service=ssh
```
```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.122.50" service name="ssh" accept'

# reload
sudo firewall-cmd --reload
```
