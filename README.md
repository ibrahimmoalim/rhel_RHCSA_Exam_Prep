## troubleshooting virt-manager

- if you get network 'default' is not active (the virtual network switch that your VM relies on to connect to the outside world isn't running) when you try to run a VM, open the terminal and run:
    ```bash
    sudo virsh net-start default
    ```
    Set it to autostart (so you don't have to deal with this error again after a reboot):
    ```bash
    sudo virsh net-autostart default
    ```

## terminal shortcuts

- ctrl+a => go to the start of line (command)
- ctrl+e => go to the end
- ctrl+u => clear line
- ctrl+l => clear terminal

- when viewing pages like `man` pages in the terminal, use these keys or shortcuts to move around:
    - g => go to top
    - shift+g => go to bottom
    - up and down arrow keys => move up and down
    - hold enter => go down
    - space move down a whole page
    - /(pattern) => search for a string, like /list (shows all 'list')
    - n => go to the next search result item (top to bottom)
    - shift+n => go to the prev search result item (top to bottom)
    - ?(pattern) => same as '/' but it's reversed so 'n' is go to next item (bottom to top)
    - q => quit

## terminal commands

- `tree -d` => `tree` command but only shows directories
- `tree -d -L 1` =>  `tree -d` but only shows 1 **level** (the first directories, like doing ls but only seeing directories, you can use higher numbers like 3 to see 3 levels)
```bash
~/mobile_project_demo$ tree -d -L 3
.
├── backend_api
│   ├── src
│   │   ├── main
│   │   └── test
│   └── target
│       ├── classes
│       ├── generated-sources
│       ├── generated-test-sources
│       ├── maven-archiver
│       ├── maven-status
│       ├── site
│       ├── surefire-reports
│       └── test-classes
├── docker
└── mobile_app
    ├── app
    │   ├── build
    │   └── src
    └── gradle
        └── wrapper

21 directories
```
- `tree -d -L 2 ./mobile_project_demo/` => using the tree command target a specific dir in cwd
- less => cat command but uses pages so you can scroll down by holding enter and use terminal shortcuts for man pages since they are similar
- more => same as less but it's older (less has more features)
- tail -f (file-path) => best for viewing live logs (-f means follow, so any new lines in that file is shown live)

### grep
- grep with regex
    - `... | grep '^2026-06-17'` => greps every line that starts with that string (^ means line starts with), this is useful for logs where you want to target a specific data, if the logs start with date you can use that, but it's used for anything not just logs and dates.
    - `... | grep 'removed$'` => greps everyline that ends with 'removed' (used $ at the end of arg)
    - `... | grep '....x86_64...'` => this will show every line that has x86_64 in it and highlight 3 chars before 'x86_64' and 2 chars after (the closest dots that are before and after 'x86_64' are the arg providers, the other dots are the amount of chars). It can be also used like:
      `... | grep '11:..:..'` => this will show every line that includes 11th hour from 11:00:00 to 11:59:59. You can also check what happened in a specific time frame like 10:30:00 to 10:50:00:
      `... | grep '10:[3-5]'` => this will only target any line that includes 10:3... , 10:4... and 10:5... (you can do '10:[3-5]0' , it'll only show 30, 40 and 50 mins exactly)

- **`grep -ri 'ibrahim' java_notes/`** => this will find any directory or file name in java_notes/ that contain the string 'ibrahim' (-r recursive -i ignore-case), showing what lines in what files
- `grep -vi 'system' java_notes/src/com/java/basics/Methods.java` => (-v means exclude) this will only show lines that don't contain 'system' (case is ignored with -i)
- `grep -in 'String' java_notes/src/com/java/basics/Arrayss.java` => (-n numbered) will show line numbers associated with the found lines, result is like this:
```java
7:    public static void main(String[] args) {
10:        // for example this string variable can be turned into an array
11:        // to hold more strings
12:        // String fruit = "banana";
14:        String[] fruits = { "orange", "banana", "apple" };
19:        // ([Ljava.lang.String;@2a139a55)
34:        // sort the strings in the array alphabetically
41:        for (String fruit : fruits) {
61:        String[] foods = new String[4];
72:        for (String food : foods) {

```
- `grep -ril 'for' java_notes/src/com/java/` => (-l means files with matches) this will grep recursively through the give dir and show files that contain 'for' in their name or inside the file it self, but it doesn't show text lines inside files, just the file names.
- `grep -o 'for' java_notes/src/com/java/basics/Arrayss.java` => shows only how many times 'for' appears, not full lines, only the string is shown:
```java
for
for
for
for
for
for
for
```

### tar
- archive a file or a dir
```bash
tar -czvf <nameOfOutputFile>.tar.gz <nameOfFileOrDirToBeArchived>
```
> -c: create, -z: zip format. -v: verbose, -f: file
- unarchive/unzip to a specific dir with -C (default unzips to cwd)
```bash
tar -xzvf <nameOfTheArchive>.tar.gz -C /home/<user>/
```
> -x: extract
> e.g `tar -xzvf someBackup.tar.gz -C /home/ibrahim/tests/`, this will extract to the tests dir in home dir

### ln (links)
- create a hard link (files will share same Inode)
> You can check Inode with `stat` command: `stat <file>`
```bash
ln file.txt path/to/file-hard-link.txt
```
> hard links don't break when either of the files is deleted,
> hard links point to Inode while sym links point to original file's path

> Hard Link: Link → Inode → Data Blocks
- create a symbolic link/soft link (files will have unique Inodes)
>
```bash
ln -s file.txt path/to/file-sym-link.txt
```
> If you edit or modify the contents of a symlink, you are actually modifying the original file. Think of a symlink like a wormhole. If you open a symlink in a text editor and make changes, the system automatically follows that link and writes the data directly into the original file.
>However, deletion behaves differently: If you delete the symlink, the original file is safe. If you delete the original file, the symlink stays behind but becomes broken (it points to a path that no longer exists).

> Symlink: Link → Path String (/path/to/file) → Target Filename → Inode → Data Blocks

### chmod
- change mode (permissions) for a file or a dir
```bash
chmod -v go+w file.txt
```
> this adds write permissions for group and others, '-v' or '--verbose' tells you what changed
- add read and write for all
```bash
# octal way
chmod -v 0666 file.txt
```
Or
```bash
#symbolic way
chmod -v ugo+rw file.txt
```
- remove write permissions for group and others
```bash
chmod -v g-w,o-w file.txt
```
> the comma isn't required, you can use "go-w"
- give group and others only write access
```bash
# if you do 'o=', others will have no access (if nothing after the equal sign)
chmod -v g=w,o=w file.txt
```
> this will make it so that if group had read,write and execute access before, it'll now only have write. '=' only gives it what is provided in the command, it's like using numbers (octal way) '0622' (2 is write, 6 means user has read and write which usually the case)

### umask (user mask)
- check default permissions
```bash
umask
```
Or
```bash
#symbolic
umask -S
```
> You'll see something like: `0022` or `u=rwx,g=rx,o=rx`(for -S).
> Ignore the first 0, `022` is for user-group-others,
> `umask` is more of a strip permissions instead of give permissions, so the 0 means by default user has full access to new files or directories, 2 means write permission is stripped, so we're left with read and execute for group and others. `0022` or `0002` is the default for Linux distros, `0002` meaning group has same permissions as user by default because each user has a group created with them.
- change default permissions
```bash
umask 0022
```
Or
```bash
umask 022
```
> if it was 0002 before the command, it'll now become 0022

### find
- find files or dir in the cwd by name, ignoring case
```bash
find . -iname ".vim*" -type d
```
> the 'i' in 'iname' is the arg for ignoring case, if you just do '-name', it'll only find directories or files with the exact given name (case sensitive). '.' only searches in cwd. '-type' only searches for given type, 'd' for dir and 'f' for files. You can also use '-user <user-name>' to only find items owned by a specific user (including root). The command finds any file or dir that starts with '.vim' (* means anything that comes after)
- find items in a specific dir
```bash
find /var/log/ -mmin -10
```
> this finds all logs modified in the last 10 mins, '-mmin' means modified minutes. Use '-mmin +60' to see all dirs or files modified more than 60 mins (1 hour) ago.
- find a dir or a file, then execute commands on them
```bash
find . -iname "*.txt" -user ibrahim -exec chmod 777 {} \;
```
> this finds any file or dir owned by 'ibrahim' ending with '.txt' and gives them all permissions to user, group, and others.

> {} = puts the items found inside a {} one by one so -exec can run commands on them, it finds the first file, drops it into {}, runs command, and then completely empties {} before moving to the next file.
> \; = end of execution.
- find by size
```bash
find ~ -size +1G
```
> This finds any dir or file bigger than 1GB, 'M' for MB, 'k(lower-case)' for 'KB'. You can of course use '-1G' to find anything less than 1GB. '~' targets homde dir.
```bash
find / -size +2G -exec du -sh {} \;
```

### managing users
#### who
Checks who's logged in (will show seat0 and tty2 which just means graphical and terminal)
#### last
Shows who logged in last and how long they been using the system (last on top)
#### id
id (user-name) => Shows userID, groupID and what groups that user is in (wheel group means user has adminstrative privileges)
#### passwd and shadow
- cat /etc/(passwd or shadow) => Used for checking users and their IDs
- cat /etc/login.defs => contains config for user creation like whether homde dir will be created for new users or not and much more
#### skel (skeleton)
This is a folder in /etc/skel that contains:
```bash
total 36K
drwxr-xr-x   2 root root 4.0K May 17 09:22 .
drwxr-xr-x 149 root root  12K Jun 23 14:03 ..
-rw-r--r--   1 root root  220 Mar  8 18:21 .bash_logout
-rw-r--r--   1 root root 3.5K Mar  8 18:21 .bashrc
-rw-r--r--   1 root root 5.2K Aug 28  2025 .face
lrwxrwxrwx   1 root root    5 Aug 28  2025 .face.icon -> .face
-rw-r--r--   1 root root  807 Mar  8 18:21 .profile

```
> These files or directories inside /etc/skel will be given by default to any new user, so you can modify this (add dir to skel/ or a file) and those changes will be seen in new users home dir.
#### useradd
You can also use `adduser` but `useradd` is universal and used by every distro
```bash
sudo useradd -m -s /bin/bash <new user's name>
```
> -m gives it home dir and pull files from `/etc/skel`, -s defines shell (default is /bin/sh)
#### passwd
When you create a user you have to give them a password to switch to that user
`sudo passwd <user-name>` => change password for a user
> Note: only change password through this command, and not through `usermod` because `usermod` doesn't hash the pass.
#### usermod
You can use this as a settings for user management if you forgot some options in `useradd` command, you can do them here, e.g:
```bash
sudo usermod -s /bin/bash <user-name>
```
Or change UID of a user:
```bash
sudo usermod -u 1500 <user-name>
```
- lock a user's password
```bash
# or -L
sudo usermod --lock <user-name>
```
> when you run this command successfully and you try to switch to the locked user, after you enter the password, you'll get something like 'authentication failed' even if the password was correct.
- unlock a user's password
```bash
# or -U
sudo usermod --unlock <user-name>
```
#### deluser
`sudo deluser <user-name>` => deletes user from system
#### chage (change age)
- `sudo chage -M 60 <user-name` => sets the maximum number of **days** a password is valid to 60.
    - If their password is 10 days old: They will be forced to change it in 50 days.
    - If their password is 70 days old: They will be prompted to change it the very next time they try to log in.
- `sudo chage -l <user-name>` => check user's current password aging status

### manage groups
#### groupadd
- create a group and add users to it
```bash
sudo groupadd -U <first-user>,<second-user> <group-name>
```
> e.g sudo groupadd -U ahmed,ali devs (-U means --users)
Or first create a group with no -U, then add users to it with `usermod`
```bash
# this only modifies one user per command
# make sure you give the -a (append) or user will be removed
# from the prev groups they were in
sudo usermod -aG <existing-group-name> <existing-user-name>
```
`gpasswd` is safer:
```bash
# if you miss the '-a' the command fails with syntax error
# no damage done
sudo gpasswd -a <user-name> <group-name>
```
- add multiple users to a group using gpasswd with a for loop
```bash
for u in ahmed ali umar; do sudo gpasswd -a "$u" admins; done
```
- remove a user from a group
```bash
sudo gpasswd -d <user-name> <group-name>
```

> It's best practice to add a group named 'admins' and put it in the sudoers file in /etc/sudoers, remove users from that file aside from root, add users you want to be able to use sudo to the admins group, you can later add more users easily and remove users from the admins group easily as well without ever touching the sudoers file, to add a group named `admins` to the sudoers file, do: `%admins ALL=(ALL:ALL) ALL` (for a user you wouldn't use the '%', thats the only difference)

- delete a group
```bash
sudo groupdel <group-name>
```

### especial permissions (rws)
> Note: If you see a capitalized S (like -rw-r-S---), it means the SUID/SGID bit is set, but the underlying file/directory is missing the execute (x) permission. A lowercase s means everything is configured correctly.
- SUID (Set User ID)
Runs binary as the file owner
```bash
# or `chmod 4755 <file>`
sudo chmod u+s <file>
```
- SGID (Set Group ID)
This is done for shared directories, where multiple users in the same group own a directory (Files inherit parent directory's group)
```bash
# or `chmod 2775 /path/to/dir`
sudo chmod g+s /path/to/dir
```
> You'll see something like:
```bash
drwxrws--T   2 root    sharedGroup 4096 Jun 24 13:59 sharedDir
```
> Focus on the group part `rws`, Anyone in sharedGroup can read, write, and enter. (The 's' means new files inherit this group, so any directories (it automatically inherits the SGID bit (s) too for directories) or files created inside that sharedDir will have be owned by `sharedGroup`).
- Sticky Bit
Used for configuring shared directories where users can't delete each other's files (doesn't do anything when applied to individual files)
```bash
# or `chmod 1777 /path/to/dir`
sudo chmod o+t /path/to/dir
```
> You'll see this on that folder:
```bash
# the 'T' means users in this dir won't be able to delete each others
# files even though they share the same group (only root/sudoers/admins
# and file owner can delete the file)
drwxrws--T   2 root    sharedGroup 4096 Jun 24 13:59 sharedDir
```
- find all SUID and SGID files on the system
```bash
find / -perm /4000 -type f 2>/dev/null
```
```bash
find / -perm /2000 -type f 2>/dev/null
```
```bash
# for directories
find / -perm /2000 -type d 2>/dev/null
```

### ip
`ip a` or `ip addr` => show ip address and networks
`ip neighbo show` => show networks recently talked to
`ip route` => show gateway address

### nmcli (network manager command-line interface)
- show all network interfaces (wifi, eth, loopback) and whether they are connected or disconnected
```bash
nmcli device status
```
- list all saved network profiles on the system
```bash
nmcli connection show
```
- scan and list all available wifi networks in the physical area
```bash
nmcli dev wifi list
```
- show detailed connection about a network using it's `connection` name you get from `nmcli device status`
```bash
nmcli connection show <wifi-connection-name/eth-connection-name>
```
- set static ip address
```bash
# network interface name is what you see when you do 'ip a'
# either starts with 'en..' if ethernet or 'wl..' if wifi
# subnet must match the original, this is 192.168.1.x/24
# dns servers are seperated by a comma (don't add more than 3)
sudo nmcli connection modify <network-interface-name> ipv4.addresses 192.168.1.120/24 ipv4.gateway 192.168.1.1 ipv4.dns "8.8.8.8,8.8.4.4,1.1.1.1" ipv4.method manual
```
- apply the change
```bash
# con is short for connection and it works
sudo nmcli con up <network-interface-name>
```


## vim notes

- `:r !cat vimrc` => this is used to copy contents of a file into the file you currently have open in vim, in this instance `vimrc` contents will be put into the current file open in vim. You can use any command and get it's output copied into the editor where your cursor is (here we are using cat)
- gg => go to top of file (while is `esc` mode)
- G => go to bottom of file
- yy => copy line
- p => paste line
- dd => delete line
- u => undo
- :wq => save and quit
- :q! => quit without saving

## extra terminal notes

- A **#** infront of a command makes it do nothing, comments it out
