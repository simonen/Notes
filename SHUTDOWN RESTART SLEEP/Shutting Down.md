
Permissions for shutting down Linux are controlled by `PolKit` (former Policy Kit) Authorization Manager

**man 8 polkit**

Legacy commands are symlinked to `/bin/systemctl`

```
$ stat /sbin/shutdown
 File: ‘/sbin/shutdown’ -> ‘../bin/systemctl’
  Size: 16              Blocks: 0          IO Block: 4096   symbolic link
Device: 801h/2049d      Inode: 537031      Links: 1
Access: (0777/lrwxrwxrwx)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:bin_t:s0
```

man
#### Shutting Down with systemctl

`systemctl poweroff` and shutdown link to the same poweroff.target in /lib/systemd/system

\# systemctl { halt | poweroff | shutdown | reboot }

* halt: halt the system without powering off the machine
* shutdown, poweroff: poweroff the machine

**systemctl** commands lack the options of the legacy shutdown commands. **Systemctl** does not have the time options

Shutting down without parameters is equal to shutdown -h +1 -> after a minute
\# **shutdown** 

Shutdown immediately with no notification to users
\# **shutdown now**

Shutdown in 10 minutes with notification
\# **shutdown -h +10**

Cancel a delayed shutdown
\# **shutdown -c**

Schedule a shutdown with a custom message
\# **shutdown -h** +*MINUTES* "*CUSTOME SHUTDOWN MESSAGE*"

Schedule a shutdown at an exact time
\# **shutdown -h 22:15**

Reboot

```bash
shutdown -r
```

Halt
\# **shutdown -H**

Clean shutdown and power off the machine
\# **shutdown -P** 

##### Shutting Down and Rebooting With Halt

**halt**: performs clean shutdown, unmounts filesystems, stops services. Does NOT power off the machine.

Reboot the system
\# **halt --reboot**

Force shutdown
\# **poweroff -f -f**
##### Sending System Into Sleep Mode

Put system in suspend mode
\# **systemctl suspend**

#### Creating Scheduled Shutdowns with cron

In **/etc/crontab**

Schedule a shutdown at 22:10 with 20 minute warning
`10 22 * * * root /sbin/shutdown -h + 20`

at weekdays, at fixed time. immediately
`00 23 * * 1-5 root /sbin/shutdown now`

\# **crontab -e**: opens crontab as root so user is not specified

To edit the crontab of a specific user:
\# **crontab -u** *USER* **-e**

#### Automated Startups 

Check out supported ACPI sleep states
cat **/sys/power/state**

For non-systemd systems
cat **/proc/acpi/info**

##### Remote Wake-ups with Wake-On-Lan

**PXE** should be disabled when using remote wake-ups otherwise the system could boot an installation environment and overwrite an existing system

Check if the network interface supports wake-on-lan
```
$ ethtool <INTERFACE>
Supports Wake-on: umbg
        Wake-on: d
```

d: disabled
g: Wake on Magic Packet

To enable wol:
\# **ethtool -s** *INTERFACE* **wol g**