# RHCSA RHEL 9 Targeted Study Guide
**Subject:** Red Hat Certified System Administrator (EX200) Preparation
**Topics Covered:**
- System Boot & Emergency Password Recovery
- LVM & Physical Extent-based Storage Configuration
- Container Management (Rootless Podman & Linger)
- Network Interface Configuration via NetworkManager (nmcli)
- Stratis Local Storage Management
- Special Permissions (SGID, SUID, Sticky Bit)
- SELinux Port Labeling and Firewall Control
- Local DNF Software Repository Setup
- Advanced File Searching and Command Execution
- LVM-based Swap Management and Persistence

## Summary
This study guide focuses on core RHEL 9 system administration tasks essential for passing the RHCSA (EX200) exam. In the actual hands-on test, you are required to perform operations that persist across reboots. This guide systematically addresses critical areas of vulnerability—such as exact syntax for recovery environments, storage virtualization (LVM and Stratis), SELinux security parameters, system persistence, and networking.

## Key Concepts

### 1. Root Password Recovery via rd.break
- The Boot Interruption: Appending rd.break to the kernel parameters line in GRUB intercepts the boot process before systemd loads, dropping you into an emergency initramfs shell.
- The Mount Cycle: At this stage, the real system root is mounted as read-only on /sysroot. You must remount it as read-write to modify /etc/shadow:
mount -o remount,rw /sysroot
- Chroot Environment: Shift the temporary boot root to the actual system root using chroot /sysroot.
- SELinux Relabeling: Because modifying /etc/shadow changes its security context, you must trigger an SELinux filesystem relabel on the next boot by running touch /.autorelabel.

### 2. Physical Extents & Custom Volume Groups
- Extent Sizing: By default, Physical Extents (PEs) are 4 MiB. You can customize this size when creating a Volume Group using the -s flag.
    - Example: `vgcreate -s 16M vg_storage /dev/sdb1` (sets the PE size to 16 MiB).
- Targeted Extent Allocation: When creating Logical Volumes based on exact extents rather than raw size, use the -l flag (lowercase L) instead of -L.
    - Example: `lvcreate -l 20 -n lv_data vg_storage` (allocates exactly 20 extents).

### 3. Persistent Rootless Containers
- User Linger: Normally, user-space systemd instances terminate when the user logs out. For a rootless podman container to persist as a background service, "linger" must be enabled.
    - Command: `loginctl enable-linger <username>`
- Systemd Integration: This allows systemd to start the user's unit files at system boot, even if they have not authenticated via SSH or console.

### 4. Static IP Management via NetworkManager
- nmcli Modification: Always use the persistent configuration tool (nmcli connection modify) instead of temporary runtime commands.
    - Syntax: `nmcli con mod <con-name> ipv4.addresses <IP/CIDR>`
- Activation: Changes do not apply immediately. You must reload or cycle the connection to read the updated profile:
    - Syntax: `nmcli con up <con-name>`

### 5. Stratis Storage Pool and Filesystems
- Pool Creation: Stratis aggregates one or more block devices into a storage pool.
- Thin Provisioning: Stratis filesystems are dynamically allocated and grow automatically. You create them on top of a pool:
    - Command: `stratis filesystem create <pool_name> <fs_name>`

### 6. Collaborative Directory Permissions (SGID)
- Collaborative Directory Pattern: To allow multiple users in a group to write to a shared folder while keeping files owned by the group, implement SGID (Set Group ID) and set appropriate read/write permissions.
    - Command: `chmod 2770 /path/to/dir`
    - Explanation: 2 sets the SGID bit; 7 gives the owner rwx; 7 gives the group rwx; 0 denies others. Any new file created inside will inherit the directory's group rather than the creator's default group.

### 7. SELinux Security vs. System Firewalls
- Firewall vs. SELinux: Opening a port in firewalld permits network traffic to reach the OS. However, if SELinux is in Enforcing mode, it will block a daemon (like httpd) from binding to a non-standard port unless that port is labeled correctly in the policy.
    - Command: `semanage port -a -t http_port_t -p tcp 8080` (allows Apache to bind to port 8080).

### 8. Local DNF Repository Configuration
- Repository Definitions: Repo files exist in /etc/yum.repos.d/ and must end in .repo.
- Local Protocols: When pointing to a local directory or mounted ISO, use the file:/// protocol (note the triple slash: two for protocol, one for absolute root path).
    - Entry: `baseurl=file:///mnt/rhel9/AppStream`

## Vocabulary List
- rd.break: A kernel boot parameter that pauses the boot process within the RAM disk (initramfs) before control is passed to the real systemd initialization process.
- chroot: Change Root. A operation that changes the active root directory for the current shell session, isolating it to a subdirectory (like /sysroot).
- Linger: A systemd-logind feature that allows user-specific services and containers to spawn at system boot and persist after user logout.
- SGID (Set Group ID): A special permission bit. On a directory, it ensures that any files/subdirectories created within it inherit the parent directory's group ID rather than the creator's primary group ID.
- semanage: SELinux Policy Management tool used to configure and view SELinux policy settings, such as file contexts, port mappings, and booleans.
- Stratis: A Red Hat local storage management solution that simplifies volume management, thin provisioning, snapshotting, and monitoring via a daemon.
- Physical Extent (PE): The smallest unit of data block (default 4 MiB) allocated within a Physical Volume in LVM storage.

## Key Questions for Active Recall
1. Which specific files or environments must be updated to ensure a new swap space mounts persistently at boot?
    - Answer: The device path (or UUID) must be registered with the filesystem type swap and mount options defaults within the /etc/fstab configuration file.
2. What happens if you change a root password in chroot /sysroot but forget to run touch /.autorelabel before booting?
    - Answer: Systemd will boot, but SELinux will block access to the newly modified /etc/shadow file because of an incorrect security context, preventing you from logging in as root.
3. What is the difference between -L and -l flags when creating a Logical Volume using lvcreate?
    - Answer: -L is used to specify size in absolute units (e.g., 10G, 512M), whereas -l is used to specify the exact number of Logical Extents (e.g., 20).
4. When configuring a network interface statically using nmcli, why must you also specify ipv4.method manual?
    - Answer: If you do not change the method from auto to manual, the system will still attempt to lease an address via DHCP alongside your assigned static IP.
5. Which command combination dynamically finds all files larger than 10MB belonging to user bill and moves them to /tmp/bill_files/ while ignoring error outputs?
    - Answer: find / -user bill -size +10M -exec mv {} /tmp/bill_files/ \; 2>/dev/null
