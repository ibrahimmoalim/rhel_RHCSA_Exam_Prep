# Manage users and groups ✅

## Create, delete, and modify local user accounts ✅
[user management notes](https://github.com/ibrahimmoalim/rhel_RHCSA_Exam_Prep#skel-skeleton)

## Change passwords and adjust password aging for local user accounts ✅

- set or change a user's password
```bash
sudo passwd ali
```
> As root, you do not need to know the old password and can bypass minimum strength requirements

- force a user to change password on next login
```bash
# -e expires the password immediately
sudo passwd -e ali
# or
sudo chage -d 0 ali
```

- inspect password aging rules for a user
```bash
sudo chage -l ali
```
sample output:
```
Last password change					            : Jul 27, 2026
Password expires					                : Oct 25, 2026
Password inactive					                : Nov 08, 2026
Account expires						                : Dec 31, 2026
Minimum number of days between password change		: 5
Maximum number of days between password change		: 90
Number of days of warning before password expires	: 7
```

- set maximum password age and warn user before password expires
```bash
# -M means maximum, -W means warn
sudo chage -M 90 -W 7 ali
```
> this starts counting from the last time the password was changed, so if ali's pass was changed 7 days ago (you can find this with `sudo chage -l ali`), then that 7 days will be substracted from the 90 that's set in the command, and so ali will now be forced to change password in 83 days.

- set minimum password age
```bash
sudo chage -m 5 ali
```
> ali will not be able to change password if last password change was within last 5 days.

- set account termination date
```bash
sudo chage -E 2028-04-16 ali
```
> if ali or anyone else tried to login into user ali, they'll get "Your account has expired; please contact your system adminstrator.", but home dir for ali will still be there. Admin will have to use the command above to set a date in the future like 2030 or `sudo chage -E -1 ali`(sets acc to never expire) for user ali to be able to login again.

- set inactivity grace period
```bash
sudo chage -I 10 ali
```
> ali's account will be locked 10 days after passowrd expires if unused, this countdown starts only after the password has already expired.

- set global password defaults

If you want default policies to apply automatically to **newly created users**, modify `/etc/login.defs`:
```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
```
> Note: Changing `/etc/login.defs` only affects users created after the change, not existing accounts

## Create, delete, and modify local groups and group memberships ✅

In RHEL/Linux, every user has:
- **Primary Group**: Stored in `/etc/passwd`. Every file the user creates is owned by this group by default. A user can only have one primary group at a time.
- **Secondary (Supplementary) Groups**: Stored in `/etc/group`. A user can belong to multiple secondary groups to gain access to shared folders or elevated permissions (like wheel or docker).

- create a standard group
```bash
sudo groupadd devs
```
- create a group with a specific GID
```bash
sudo groupadd -g 3000 sysadmins
```
- rename an existing group
```bash
# rename 'devs' group to 'developers'
sudo groupmod -n developers devs
```
- delete a group
```bash
sudo groupdel developers
```
> Note: You cannot delete a group if it is currently set as the primary group of any existing user. You must change the user's primary group first.

- change a user's primary group
```bash
# set 'devs' as ali's primary group
sudo usermod -g devs ali
```
- add user to secondary groups
```bash
# add ali to sysadmins
sudo gpasswd -a ali sysadmins

# remove ali from sysadmins
sudo gpasswd -d ali sysadmins
```
- verify group memberships
```bash
# Check all groups 'ali' belongs to (Primary listed first)
id ali

# Search for the group definition in /etc/group
# (shows who's in 'sysadmins' and GID)
getent group sysadmins
```

## Configure privileged access ✅
#### The `wheel` Group
In RHEL/CentOS/Rocky, members of the wheel secondary group are automatically granted full root privileges through a default rule in `/etc/sudoers` which is edited with `visudo` (The safe editor that locks and validates `/etc/sudoers` syntax before saving). **DO NOT EDIT /etc/sudoers directly**:
```
%wheel ALL=(ALL) ALL
```

To grant a user full administrative privileges, simply add them to the wheel group:
```bash
sudo gpasswd -a ali wheel
```
> Once added, ali can execute any command as root by prefixing it with sudo

#### Editing Rules safely: visudo
Running visudo opens your default text editor, but checks syntax on save. If you make a typo, visudo will warn you and prevent saving, saving you from locking yourself out of root privileges.
```bash
# Safely edit main sudoers file
sudo visudo

# Safely edit or create a specific drop-in file
sudo visudo -f /etc/sudoers.d/custom_rule
```

Instead of editing /etc/sudoers with `visudo` create drop-in rule files inside `/etc/sudoers.d/`. This keeps custom rules organized and avoids losing changes during system updates.

Syntax Rule Structure:
```
WHO   WHERE=(AS_WHOM)   [TAGS:] COMMANDS
```
Example:
```
# for groups: use "%", for users: use name only
# use "ALL" at the end for all commands (for specific commands, full path must be used)
%webdevs ALL=(ALL) NOPASSWD: /usr/bin/dnf
```

- grant full admin access without requiring a password prompt:
```bash
# -f means file, it creates the file if it doesn't exist
# and lets you edit it, just like using 'sudo vi'
# you don't have to use -f in modern distributions
sudo visudo -f /etc/sudoers.d/sysadmins
```
Add line:
```
%sysadmins ALL=(ALL) NOPASSWD: ALL
```
- grant limited permissions (e.g only manage services)
```bash
sudo visudo /etc/sudoers.d/webdevs
```
Add:
```
yahya ALL=(ALL) /usr/bin/systemctl status httpd, /usr/bin/systemctl restart httpd
```
> if you don't know full path, use `which httpd`
- allow a group to manage packages only without being asked for password
```bash
sudo visudo /etc/sudoers.d/pkg_managers
```
Add:
```
%pkg_managers ALL=(ALL) NOPASSWD: /usr/bin/dnf
```
> users in group 'pkg_managers' will be able to use 'sudo dnf ...' without entering their passwords.

- verify configured privilages without changing users
```bash
# List all allowed (and forbidden) commands for 'ali'
sudo -l -U ali

# Test running a command as another user (or root)
sudo -u root whoami
```