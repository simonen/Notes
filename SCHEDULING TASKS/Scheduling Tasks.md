man **systemd.time**

**at** - for future execution of one-time tasks only
**cron** - for recurring task execution. Legacy
**systemd timers**: default solution for starting jobs at a certain time

#### Systemd Timers

**A systemd timer is always used with a service file, names should match**

logrotate.timer -> logrotate.service

The service unit files define how the unit should be started
The timer unit file define when.

To list active timers

```bash
systemctl list-timers
```

```
[kimchen@rhel9 ~]$ systemctl cat logrotate.timer
# /usr/lib/systemd/system/logrotate.timer
[Unit]
Description=Daily rotation of log files
Documentation=man:logrotate(8) man:logrotate.conf(5)

[Timer]
OnCalendar=daily
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target
```

**OnCalendar**: defines when the timer should execute. Options: daily; *:00/10* - every 10 minutes, `*-*-*`- daily, 
**AccuracySec**: defines the time window in which the timer should execute. For best accuracy 1us
**Persistent**: a modifier to **OnCalendar**, stores the last execution on disk and ensures it will be executed exactly one day later.

**Systemd** **timer** options:

**OnActiveSec**: defines a timer relative to when the timer is activated
**OnBootSec**: defines a timer relative to when the machine was booted
**OnStartupSec**: defines a timer relative to when the service manager was started. Same as OnBootSec except for when user service units are used
**OnUnitActiveSec**: defines a timer relative to when the unit that the timer activates was last activated
**OnCalendar**: defines a timer based on calendar events, such as daily. **man systemd.time**

Custom units like timers can be put as drop-in files in **/etc/systemd/system/**NAME.timer
NAME.service -> NAME.timer
