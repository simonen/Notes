---
tags:
  - systemd
  - centos
  - rhel
---

**systemd target**: a group of units that belong together. 
**isolatable target**: define the state the system is booting into. Have specific property: can be isolated. Has the `AllowIsolate=yes` directive in its unit file

* **emergency.target**: boots the system with minimal set of units for disaster recover
* **rescue.target**: starts all units except for unessential units.
* **multi-user.target**: starts everything needed for fully operational system. Used by servers
* **graphical.target**: full functionality plus graphical user interface

#### Working with Targets

Targets do:
* adding units to be automatically started
* setting a default target
* running non-default target to enter troubleshooting mode

`systemctl enable / disable` adds or removes services to / from targets.

#### Target Units

A target unit has:
* `/etc/systemd/system/unit.wants`: Contains reference to all files that need to be loaded
* target unit file:

```
[kimchen@rhel9 timers.target.wants]$ systemctl cat multi-user.target

[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes
```

The target does not contain info about the units that should be included
A target unit file contains what it requires and which services it cannot coexist with. Defines load ordering in the After statement in the Unit section.

#### Wants

*Wants* in Systemd define which units Systemd wants when starting a specific target
Wants are created as directories when using the `systemctl` enable** command, containing wants as symbolic links to the services that need to be started
`/etc/systemd/system/`: Contains the **wants** dirs of different targets
#### Managing Targets

The **\[Install]** section of a unit file defines the target in which a service should be started
```
[kimchen@rhel9 multi-user.target.wants]$ systemctl cat vsftpd.service
# /usr/lib/systemd/system/vsftpd.service
[Unit]
Description=Vsftpd ftp daemon
After=network-online.target

[Service]
Type=forking
ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

[Install]
WantedBy=multi-user.target
```

The `systemctl enable` command creates a symlink 

`vsftpd.service` -> `/usr/lib/systemd/system/vsftpd.service` in the `/etc/systemd/system/multi-user.target.wants` directory

Changing the default target works similarly:
`Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/multi-user.target.`

or

``` bash
systemctl set-default "TARGET"
```
#### Isolating Targets

Isolating a target is similar to SysV runlevels. Used to put the system in a particular state.

```
[root@localhost i386-pc]# systemctl cat graphical.target
# /usr/lib/systemd/system/graphical.target

[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
AllowIsolate=yes
```

`AllowIsolate` directive defines if a target can be isolated.

``` bash
systemctl isolate "TARGET"
```

```
[root@prometheus grub]# systemctl cat rescue.target
# /usr/lib/systemd/system/rescue.target

[Unit]
Description=Rescue Mode
Documentation=man:systemd.special(7)
Requires=sysinit.target rescue.service
After=sysinit.target rescue.service
AllowIsolate=yes

[Install]
Alias=kbrequest.target
```

The `Alias` option in the \[Install] section means that we can symlink kbrequest.target to rescue.target

``` bash
[root@prometheus grub] systemctl enable rescue.target
Created symlink from /etc/systemd/system/kbrequest.target to /usr/lib/systemd/system/rescue.target.
```

`kbrequest.target` is a special `systemd` unit that is started by pressing ALT+UpArrow which should enter rescue mode
