---
tags:
  - systemd
  - centos
  - rhel
---

`init` - Legacy. Minimalistic approach to services and daemons. System modes are called **Run Levels** 
Used in lightweight systems where the functionality of systemd is not needed.

`systemd` - The  base process (PID 1) that spawns all other processes. Modern. Expanded functionality. Aims to standardize modern linux systems. It provides an uniform interface 
that starts **units**. 

Advantages:
* `systemd` Is event-driven. Can respond to system events (new hardware being plugged, traffic starting on a network port)
* concurrent and parallel boot processing
* can respawn processes
* event logging
* tracks processes via kernel **cgroups**

[daemon](https://en.wikipedia.org/wiki/Daemon_(computing)) : one or more processes running on a host. Ex. sshd, httpd, mysqld

Processes starting with "k" are **kernel threads** - special kinds of services performing management tasks in the core of the os.

kworker/0:3-events : thread_name/CPU

#### Identifying System Management Daemon 

Ways of checking if a system uses `systemd` or `SysV`

* The system uses `systemd` if `/run/systemd/system` is present
* `stat /sbin/init` is symlinked to `/lib/systemd/systemd`
* `/sbin/init` is not symlinked -> `SysV`

The command attached to **PID 1** process is the **init** - the first process launched at startup

```
CentOS
[root@nfsserver ~]# ps -p 1 or ps 1
  PID TTY          TIME CMD
    1 ?        00:00:15 systemd
# Fedora
kimchen@kimpc:~ $ ps -p 1
PID TT  STAT    TIME COMMAND
  1  -  ILs  0:00.11 /sbin/init
```

```
#systemd
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0 193884  7012 ?        Ss   08:41   0:11 /usr/lib/systemd/systemd

#SysV
root      1   0.0  0.0 11768 1168  -  ILs  18:17     0:00.08 /sbin/init
```

**systemd**: /proc/1/exe links to **/usr/lib/systemd/systemd**

```
[root@nfsserver ~]# stat /proc/1/exe
  File: ‘/proc/1/exe’ -> ‘/usr/lib/systemd/systemd’
  Size: 0               Blocks: 0          IO Block: 1024   symbolic link
```

**SysV**: **/proc/1/exe** links to **/sbin/init**

```
[root@nfsserver ~]# cat /proc/1/comm
systemd
```

#### SysV, SysV Init, SystemV or just init (Legacy)

- `/sbin/init`: binary
- `/etc/inittab` : conf file
- `/etc/rc0-6.d/` : Contain scripts, starting with K (shutting down) or S (Starting) for the particular runlevel
- `/etc/init.d/` : Systems running `systemd-sysv` will look here if a `systemd` service file cannot be found

``` 
[root@prometheus rc0.d]# ll
lrwxrwxrwx. 1 root root 20 Jan 19 18:49 K50netconsole -> ../init.d/netconsole
lrwxrwxrwx. 1 root root 17 Jan 19 18:49 K90network -> ../init.d/network
```

`systemd-sysv` generated service files look like this

```
[Service]
...
ExecStart=/etc/init.d/postfix start
ExecStop=/etc/init.d/postfix stop
ExecReload=/etc/init.d/postfix reload
```

```
systemctl disable postfix
postfix.service is not a native service, redirecting to systemd-sysv-install
Executing /lib/systemd/systemd-sysv-install disable postfix
```
##### SysV RunLevels

SysV Runlevels are mapped to `systemd` targets using symlinks

Runlevels can be found here: 
- `/usr/lib/systemd/system`

```
lrwxrwxrwx. 1 root    15 Apr 11 09:18 runlevel0.target -> poweroff.target
lrwxrwxrwx. 1 root    13 Apr 11 09:18 runlevel1.target -> rescue.target
drwxr-xr-x. 2 root    50 Apr 11 09:18 runlevel1.target.wants
lrwxrwxrwx. 1 root    17 Apr 11 09:18 runlevel2.target -> multi-user.target
drwxr-xr-x. 2 root    50 Apr 11 09:18 runlevel2.target.wants
lrwxrwxrwx. 1 root    17 Apr 11 09:18 runlevel3.target -> multi-user.target
drwxr-xr-x. 2 root    50 Apr 11 09:18 runlevel3.target.wants
lrwxrwxrwx. 1 root    17 Apr 11 09:18 runlevel4.target -> multi-user.target
drwxr-xr-x. 2 root    50 Apr 11 09:18 runlevel4.target.wants
lrwxrwxrwx. 1 root    16 Apr 11 09:18 runlevel5.target -> graphical.target
drwxr-xr-x. 2 root    50 Apr 11 09:18 runlevel5.target.wants
lrwxrwxrwx. 1 root    13 Apr 11 09:18 runlevel6.target -> reboot.target
```

To get the current runlevel

```bash
runlevel
```

To move to another runlevel. telint 3 in effect executes the multi-user.target

```bash
telint "RUNLEVEL(0-6)"
```

##### Initd Scripts in SysV

initd scripts start, stop or show status of processes. LSB-compliant. 
LSB Linux Standard Base: set of standards

#### Ubuntu Upstart

The init process under Upstart is event-driven

Service daemon
`initctl`

Definition files or Upstart scripts can be found in
`/etc/init`

Managing SysV init.d  services in Ubuntu is done via the `update-rc.d` command. It creates or removes symbolic links in `/etc/rc*.d `

`update-rc.d` options:

start: explicitly state the *runlevels* and startup sequences
stop: explicitly state the sequence and *runlevels* to stop a service
defaults: creates start and stop *symlinks* with default start stop sequences
remove: removes symlinks from runlevel dirs
- `-n`: dry run
- `-f`: force

S20: start priority 20
K80: stop priority of 80

``` bash
update-rc.d "SERVICE" defaults start 
```

The init.d scripts are symbolically linked in /etc/rc*.d and given standard start and stop priorities of 20 and 80.

TO configure the service to start at runlevels 2 and 3 with priority 40

``` bash
update-rc.d "SERVICE" start 23 40
```

To turn off the service in runlevel 2. This adds a K80"SERVICE" symbolic link in /etc/rc2.d 

```
update-rc.d "SERVICE" stop 2
```

To remove a service from all runlevels

```bash
update-rc.d "SERVICE" remove
```

If a service is still present in /etc/init.d, which means the service has not been uninstalled first, add the -f option