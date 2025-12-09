
#### rsyslogd Modules

modules add to **rsyslog** functionality


**syslog**: the original logging solution for UNIX. 
**rsyslog**: enhanced version of syslog. Backward compatible

##### Connecting journald to rsyslog

`/etc/systemd/journald.conf`
```
$OmitLocalLogging on
$IMJournalStateFile imjournal.state
```
