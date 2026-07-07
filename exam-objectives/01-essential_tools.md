# Understand and use essential tools ✅

## Access a shell prompt and issue commands with correct syntax ✅
- this is basic commands like `ls`, but make sure to use double-tab (hitting tab key twice) if you're stuck on a command and don't know what comes next (RHEL suggests options), if bash-completion isn't working, install it via: `sudo dnf install bash-completion`

## Use input-output redirection (>, >>, |, 2>, etc.) ✅
- `>, >>, |` are simple enough, `2>` is for redirecting error ouputs
```bash
# redirect it to '/dev/null' (which is nowhere, it's deletes it forever)
cta dkjbvgrekj 2> /dev/null
# or redirect it to a specific file
cta rkgnkdn 2> error.txt
# you can append it to the end of a file too
rgergrg 2>> error.txt
# redirect both standard output and error
<command> &> file.txt
```

## Use grep and regular expressions to analyze text ✅
- using grep with regex, find only actual config lines in a config file
```bash
# -v: shows lines that do not match (so ignores anything that
# starts with # (^ means starts with) and any empty lines
# because $ means ending, and so if combined ^$ it means
# line that starts and ends immidietly (empty line))
# -e: is used to search for multiple different keywords at the same time
# it's like an "OR" operator, so in this case, ignore commented out lines
# and empty lines.
# -v is only used once and -e multiple times, depending on the patterns
grep -ve '^#' -e '^$' /etc/ssh/sshd_config
```
[more grep with regex notes](https://github.com/ibrahimmoalim/rhel_RHCSA_Exam_Prep#grep)

## Access remote systems using SSH ✅

## Log in and switch users in multi-user targets ✅
- this is basically switching users with `su - <user-name>` command and logging out with `exit`, but you can also switch the targets instantly without needing reboot with the command:
```bash
sudo systemctl isolate <target-name>
# for example switch to terminal only environment immediately
sudo systemctl isolate multi-user.target
# switch back to GUI
sudo systemctl isolate graphical.target
```
[more notes on targets](https://github.com/ibrahimmoalim/rhel_RHCSA_Exam_Prep#systemctl-targets)

## Archive, compress, unpack, and uncompress files using tar, gzip, and bzip2 ✅
- [tar notes](https://github.com/ibrahimmoalim/rhel_RHCSA_Exam_Prep#tar)

## Create and edit text files ✅
- create with `touch` command and edit with `vi` or `sudo vi`

## Create, delete, copy, and move files and directories ✅
- touch or mkdir for creating
- `rm`(for removing file) or `rm -rf`(for removing dirs)
- `cp` or `cp -r`(for dirs)
- `mv` to move files or dirs form a place to place or to rename them in the cwd

## Create hard and soft links ✅
- [hard and sof link notes](https://github.com/ibrahimmoalim/rhel_RHCSA_Exam_Prep#ln-links)

## List, set, and change standard ugo/rwx permissions ✅
- `chown` and `chgrp` to change ownership of fiesl or dirs
```bash
# change owner to ali
sudo chown ali file.txt
```
```bash
# change group to 'devs'
sudo chgrp devs tests/
```
```bash
# change both (you don't need sudo if you are root acc)
chown ali:devs project/
```
```bash
# change ownership recursively for a dir
chown -R ali:devs /home/shared_dir/
```
- [chmod notes](https://github.com/ibrahimmoalim/rhel_RHCSA_Exam_Prep#chmod)

## Locate, read, and use system documentation including man, info, and files in /usr/share/doc ✅
for this you have to use `man` pages, `info` and `/usr/share/doc/` are important too but if you're stuck on something and `man` can't help you out, just move to the next task and come back for it later and use `ls /usr/share/doc/<pkg-name-orkeyword>` and read the .md files to see if they can help (look for a README.md)
- update the man page database (sometimes you don't find the item you're looking for and the man page has it but it's db isn't updated)
```bash
sudo mandb
```
- search for a keyword if you don't know the name of a man page
```bash
man -k <keyword>
```
