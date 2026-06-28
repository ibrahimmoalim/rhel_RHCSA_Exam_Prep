# Understand and use essential tools

## Access a shell prompt and issue commands with correct syntax ✅
- this is basic commands like `ls`, but maka sure to use double-tab (hitting tab key twice) if you're stuck on a command and don't know what comes next (RHEL suggests options), if bash-completion isn't working, install it via: `sudo dnf install bash-completion`

## Use input-output redirection (>, >>, |, 2>, etc.) ✅
- `>, >>, |` are simple enough, `2>` is for redirecting error ouputs
```bash
# redirect it to '/dev/null' (which is nowhere, it's basically hidden)
cta dkjbvgrekj 2> /dev/null
# or redirect it to a specific file
cta rkgnkdn 2> error.txt
# you can append it to the end of a file too
rgergrg 2>> error.txt
# redirect both standard output and error
<command> &> file.txt
```

## Use grep and regular expressions to analyze text
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

