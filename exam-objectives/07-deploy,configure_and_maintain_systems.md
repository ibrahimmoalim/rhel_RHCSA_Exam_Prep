# Deploy, configure, and maintain systems

## Schedule tasks using at, cron and systemd timer units
Automating task execution is a core administration skill. On the exam, you'll be expected to schedule jobs using three distinct mechanisms: one-off execution (`at`), traditional recurring schedules (`cron`), and modern service triggers (`systemd timers`).

### at
Use `at` when you need a job to execute exactly once at a specific time in the future.
- install `at` if missing
```bash
sudo dnf install at -y
```
- enable and start it
```bash
sudo systemctl enable --now atd
```
- create an `at` job
specify the time, enter interactive mode, type commands, and exit with `ctrl+d`
```bash
# this will open interactive mode
# do 'sudo at 12:00' if you want to run commands as root in the future
at 12:00
# type commands to run at 12:00
echo "hello" > hello.txt
# save and exit with [ctrl+d]
```
- handy time syntax examples:
    - `at 02:00 PM`
    - `at now + 5 minutes`
    - `at now + 3 days`
    - `at 9:00 AM tomorrow`
    - `at 4:00 PM Jul 25`
- managing `at` jobs (do sudo before these to manage jobs owned by root)
```bash
atq                # list all pending jobs (shows job ID, date, time)
at -c <job_id>     # view the script/commands inside that job
atrm <job_id>      # delete a scheduled job
```

### cron
`cron` is used for recurring scheduled jobs (e.g., every midnight, every Monday at 3 AM).

To edit your user's crontab safely, always use `crontab -e` (never edit `/etc/crontab` directly unless explicitly asked).
- run `crontab -e` to schedule cron tasks for current user
It'll be opened in an editor
```bash
# ┌───────────── minute (0 - 59)
# │ ┌─────────── hour (0 - 23)
# │ │ ┌───────── day of month (1 - 31)
# │ │ │ ┌─────── month (1 - 12)
# │ │ │ │ ┌───── day of week (0 - 6) (0 is Sunday)
# │ │ │ │ │
# * * * * * <command_to_execute>
```
- practical examples:
    - run every day at 2:30 AM
    ```bash
    30 2 * * * /usr/bin/tar -czf /var/backups/data.tar.gz /data
    ```
    - run every 15 minutes during work hours (8 AM - 4 PM) on weekdays
    ```bash
    */15 8-15 * * 6,0-4 <command>
    ```
    - run hourly/daily/weekly/monthly/yearly
    ```bash
    @daily <command>
    ```
- managing crontabs
```bash
crontab -e       # edit current user's crontab
crontab -l       # list current user's cron jobs
crontab -r       # wipe all cron jobs for the current user

# edit another user's crontab as root
sudo crontab -u ali -e
sudo crontab -u ali -l
```

### modern scheduling with systemd timers
Red Hat is pushing heavily toward systemd timer units because they offer better logging via **journalctl**, can trigger on system events, and don't require an active user session.

A systemd timer always requires **two files** in `/etc/systemd/system/`
1. A **service unit** (`.service`), which defines **what** to execute
2. A **timer unit** (`.timer`), which defines **when** to trigger the service

- if you were to run: `/usr/bin/logger "hello"` every sunday at 8:00 AM, you'd follow these steps:
- create the service unit (`/etc/systemd/system/job_test.service`)
```Ini
[Unit]
Description=A Hello Log

[Service]
Type=oneshot
ExecStart=/usr/bin/logger "hello"
```
- create the matching timer unit (`/etc/systemd/system/job_test.timer`)
> Note: the filename prefix MUST match the service name (e.g., job_test.timer targets job_test.service).
```Ini
[Unit]
Description=Run Job Test Every Sunday Morning

[Timer]
# Format: DayOfWeek Year-Month-Day Hour:Minute:Second
OnCalender=Sun *-*-* 08:00:00
# if the server is down when time is reached, 'Persistent=true' ensures
# systemd triggers the missed job immediately the next time the machine boots up.
Persistent=true

[Install]
# Allows the timer to start automatically when the system boots.
WantedBy=timers.target
```
- enable and start the timer
```bash
# reload systemd to recognize the new unit files
sudo systemctl daemon-reload

# enable and start ONLY the .timer unit (NOT the .service)
sudo systemctl enable --now job_test.timer
```
> If you get an error, make sure syntax is correct in both files and validate the OnCalendar syntax with:
```bash
# this is sat-fri but because this calendar starts weekdays on mon, we have to do mon..thu first
# (the two dots mean mon through thu) then we do sat and sun. this only skips fri.
# we cannot say sat..thu directly.
systemd-analyze calendar "Mon..Thu,Sat,Sun *-*-* 08:00:00"
```
- verify timers
```bash
# list all active timers
systemctl list-timers

# check timer status
systemctl status job_test.timer

# check if the execution output succeeded via journalctl
sudo journalctl -u job_test.service
```