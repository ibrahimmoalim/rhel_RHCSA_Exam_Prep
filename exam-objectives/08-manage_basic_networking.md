# Manage basic networking

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