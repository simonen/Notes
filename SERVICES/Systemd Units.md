---
tags:
  - systemd
  - centos
  - rhel
---
[systemd units](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)

**unit** - an entity managed by **systemd**. Can be a:
- service
- socket
- device
- mount point
- automount point
- swap file or partition
- startup target
- timer
- slice
- scope

To print all unit types

``` bash
systemctl -t help
```

In **systemd**, the behavior of each unit is defined and configured by a **unit file**.

In the case of a service, the unit file specifies:
- location of the executable file for the daemon
- tells **systemd** how to start and stop the service
- identifies other units the service depends on

**CONVENTIONS**

Unit files have a suffix that matches the type of unit.
- unit.service
- unit.timer
- unit.target
#### Systemd Service Units

**man 5 systemd.service**

A service unit is used to start a process.

```
[Unit]
Description=Vsftpd ftp daemon
After=network.target
[Service]
Type=forking
ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf
[Install]
WantedBy=multi-user.target
```

Unit file sections
* `[Unit]`: describes the unit and defines dependencies. Contains the: `Before` and `After` statements which indicate whether the unit should start before or after a specified dependency.

* `[Service]`: Describes how to start and stop the unit and request status installation. 
	* `ExecStart` - indicates how the unit is started, 
	* `ExecStop` - how it is stopped. 
	The Type option indicates how the process should start. 
	* Types: `forking`, `oneshot`, `simple`, `dbus`, `notify`, `idle`

* `[Install]`: indicates in which target this unit has to be started. In the above case, a symlink of the service file will be created in `/etc/systemd/system/multi-user.target.wants `when the service is enabled

##### Service Drop-In Files

To **override** or **extend** **systemd-managed** services without changing the original service files, drop-in files can be used. 

```
/etc/systemd/system/<service>.d/*.conf
/etc/systemd/system/docker.service.d/override.conf
```

`systemd` reads these files after the main service file and applies any overrides.

>This keeps customizations separate from the package-managed unit. This means that `yum/dnf` updates won't overwrite and customization.

Create a drop-in directory for the service and a drop-in configuration file. Example with docker service override

`etc/systemd/system/"SERVICE".service.d/override.conf`
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
```

- The empty `ExecStart=` clears the original `ExecStart`. Without it, the new command will be appended.
- The new `ExecStart` line defines the new startup command. 

`systemctl` now shows that a drop-in configuration file is being used.

```bash
systemctl status docker
```

```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; preset: enabled)
    Drop-In: /etc/systemd/system/docker.service.d
             └─override.conf
```
#### Systemd Mount Units

Mount units are an alternative to mounting via `/etc/fstab`

`tmp.target`
```
# /usr/lib/systemd/system/tmp.mount

[Unit]
Description=Temporary Directory
ConditionPathIsSymbolicLink=!/tmp
DefaultDependencies=no
Conflicts=umount.target
Before=local-fs.target umount.target

[Mount]
What=tmpfs
Where=/tmp
Type=tmpfs
Options=mode=1777,strictatime

[Install]
WantedBy=local-fs.target
```
#### Systemd Socket Units

A **socket** creates a method for applications to communicate. A **socket** could be a file or a port on which **systemd** will be listening for incoming connections. That way, a service will only start when there is a connection coming in on that socket. Every socket needs a corresponding service file.

Example:

`/usr/lib/systemd/system/sshd.socket`
```
[Unit]
Description=OpenSSH Server Socket
Documentation=man:sshd(8) man:sshd_config(5)
Conflicts=sshd.service

[Socket]
ListenStream=22
Accept=yes

[Install]
WantedBy=sockets.target
```

- `ListenStream` -> TCP
- `ListemDatagram` -> UDP
#### Systemd Target Units

**target**: a group of units, loaded in the right order, executed at the right time. Can have dependencies on other targets. Does not contain information about units themselves, just what units and services it can coexist with and the order or execution. That is included in the Install section of the different unit files.

When a unit is added to a target, a symbolic link is created in the target directory in `/etc/systemd/system/unitfile.target` pointing to the unit file in `/usr/lib/systemd/system/unit.service`

This symbolic link is a.k.a a `want` - defines what the target wants to start.

To get | set the default target

``` bash
systemctl {get | set}-default
```

To isolate, enter a target mode. TAB TAB to list all isolatable modes.

```bash
systemctl isolate multi-user.target
```

#### Systemd Timers

[[LINUX/SCHEDULING TASKS/Scheduling Tasks]]
