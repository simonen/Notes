
`logrotate` conf file
`/etc/logrotate.conf`

Drop-ins in:
`/etc/logrotate.d`

`man logrotate`

`Log rotation` is the process of periodically copying the log file and adding a suffix like the date or an increment number. The `rsyslog` daemon then logs to a new file. Rotated log files are usually kept for a fixed period of time.

Example conf for apt log rotation

`/etc/logrotate.d/apt.term.log`
```
/var/log/apt/term.log -> Log file(s) or dirs to rotate
options
{ 
rotate 12 -> 12 rotations before deletion -> keep 12 files at a time
monthly -> rotations once a month
compress -> immediately compress rotated files
missingok -> If the log file is missing logrotate moves on to the next w/o error
notifempty
dateext -> add date as suffix
size SIZE -> Logs are rotated when they reach a the specified size
}
```

The `logrotate` command defaults to the `/etc/logrotate.conf`. Can take another conf file as argument

``` bash
logrotate "LOGROTATE.conf"
```