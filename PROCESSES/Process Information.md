
**man 1 ps**

#### The `/proc` filesystem

**Process Information** `/proc` is a virtual or pseudo filesystem which provides an interface to kernel data structures and is **populated per PID namespace**.

`ps`: displays information about a selection of the active processes. 

For a short summary of active processes:
**$ ps aux**

To show processes and the exact command that started it:

``` bash
ps -ef
```

To show hierarchical overview of proc

``` bash
ps fax
```

To list processes run by real and effective user:

``` bash
ps -U "USER" -u "USER" u
```

View the full process path and the arguments it was started with

``` bash
ps -p  1 -o args=
# /usr/lib/systemd/systemd --switched-root --system --deserialize 22
```

See the command behind a process

``` bash
ps -p 1 -o comm=
# systemd
```

-U: real user
-u: effective user

Show a process tree
```
[root@nfsserver ~]# pstree -sp 4089
systemd(1)─┬─NetworkManager(649)─┬─{NetworkManager}(666)
           │                     └─{NetworkManager}(668)
           ├─abrt-watch-log(613)
           ├─abrtd(610)
```

```
 PID USER     STAT COMMAND
    1 root     Ss   systemd
    2 root     S    kthreadd
    4 root     S<   kworker/0:0H
    46 root     SN   ksmd
	568 root     S<sl auditd
	31613 root     R+   ps
```

* R: currently running or waiting in the run queue
* l: process is multithreaded
* S: interruptible sleep; the process is waiting for an event to complete
* s: session leader. Sessions are related processes organized as a unit
* I: idle kernel thread
* <: high priority
* N: low priority

Threads are enclosed in curly brackets

#### lsof Command

See what processes are using a directory

``` bash
lsof +D "/DIR"
```

#### Process Priorities

**cgroup**: used to allocate system resources. Have three main areas (slices):

**system**: where all **systemd** processes are running
**user**: where all user processes + root are running
**machine**: optional slice for virtual machines and containers

By default CPU resources are equally divided to all three slices if there is high demand.
Within a slice, process priority can be managed by **nice** and **renice**

To kill a single CPU core
`$ echo 0 > /sys/bus/cpu/devices/cpu1/online`

To bring it back online
`$ echo 1 > /sys/bus/cpu/devices/cpu1/online`

To kill all dd processes:
**$ killall dd**

#### Managing Process Priorities

**man nice**
**man renice**

*How much nice a process is to give up resources to other processes*. 

**nice**: set a process priority at creation
**renice**: modify process priority 

**PID**: Process ID
**PR**: Priority -> Default 20. Lowest 39, Highest 0 - should be avoided.
**NI**: Niceness -> Highest: 19 = PR: 39; Lowest -20 = PR: 0 - should be avoided

**Nice** and **renice** use values ranging -20:19. Default value of **nice** is 0, which means priority 20. 

Regular users can decrease a priority, only **root** can **increase** priority by giving **negative** nice values. Increments of 5 is a good rule of thumb to observe process behavior.

Priorities can be set on individual process, as well as users, which will alter all their own prio

Run a bogus process to simulate CPU load
$ yes > /dev/null &

To set a absolute nice value
\# **renice -n** \[-20, 19] -p *PID1 PID2* -u *USER1 USER2*

To set a relative nice value
\# renice +- \[-20, 19] -p *PID1 PID2* -u *USER1 USER2*

The following command would decrease the priority of the processes with
PIDs  987  and  32,  plus  all processes owned by the users daemon and root, by 1
```
renice +1 987 -u daemon root -p 32
```

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 6865 kimchen   20   0  108100    620    524 R 100.0  0.0   4:41.03 dd
    1 root      20   0  128012   6668   4184 S   0.0  0.4   0:02.29 systemd
```

Decrease to the priority of a running process to the lowest (39)
\# **renice -n 19 -p** *PID*

Increase priority to the highest
\# **renice -n -20** **-p** *PID*

Create a process with a priority of  priority of 25 , nice 5 and send it to the background
$ nice -n 5 dd if=/dev/zero of=/dev/null &

Run a script with a pre-defined nice level
\# nice -n 5 /*SCRIPT*.sh

