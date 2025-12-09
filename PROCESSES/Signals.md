---
tags:
---

man 7 `signal`
man 5 `systemd.kill`

Every process has a parent process. In modern Linux, if a parent process is killed, the child process becomes child of **systemd**

Signals can be sent to processes.

`SIGTERM (15)`:  Asks a process to stop ( graceful , should be tried first)
`SIGKILL (9)`: Forces a process to stop (nuclear option)
`SIGHUP (1)`: Hangs up a process. Makes the process reread its configuration files. Useful when making changes to config files

On `systemd` systems it is better to use `systemctl kill`  instead of `kill PID` as it terminates all processes that belong to a service and does not leave orphan processes

``` bash
systemctl kill 'PROCESS_NAME'
```

#### The kill Command

The command `kill` sends the specified signal to the specified processes. Defaults to `SIGTERM (15)`

To show a list of available signals:

``` bash
kill -l
```

`kill PID`: sends a `SIGTERM` signal to terminate a process and close all its open files.
`kill -9 PID`: sends a `SIGKILL` signal to forcefully terminate a process.
`kill -s SIGNAL_NAME_or_NUMBER PID`

>Using **kill -9** risks losing data and killing other processes that depend on the target process

`killall P_NAME`: kill multiple processes using the same name. 

For a list of all `killall` signals: 

``` bash
killall -l
```

`pkill NAME`: Uses name rather than the PID of a process

To generate an I/O stream as a background job:
**$ dd if=/dev/zero of=/dev/null &**

```
[root@headoffice ~]# ps fax | grep -B5 dd
861 ?        Ss     0:00 /usr/sbin/cupsd -l
    863 ?        Ss     0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
   2189 ?        Ss     0:00  \_ sshd: kimchen [priv]
   2195 ?        S      0:00      \_ sshd: kimchen@pts/1
   2200 pts/1    Ss     0:00          \_ -bash
   2259 pts/1    R      0:07              \_ dd if=/dev/zero of=/dev/null
   2260 pts/1    R      0:06              \_ dd if=/dev/zero of=/dev/null
   2261 pts/1    R      0:06              \_ dd if=/dev/zero of=/dev/null
   2262 pts/1    R      0:06              \_ dd if=/dev/zero of=/dev/null
   2263 pts/1    R      0:06              \_ dd if=/dev/zero of=/dev/null
```

To kill the parent process:

``` bash
kill -9 2200
```

This will kill the parent process (the user shell) and send the child processes to **systemd**.
To kill all dd processes:

``` bash
killall dd
```

Using pkill

`pkill PROCESS_NAME` : NOT RECOMMENDED!!! This will possibly try to kill other processes that might have overlapping names! Always enforce full names when using `pkill`!!!

Use full process name matches only:

``` bash
pkill -fe 'PROCESS_NAME'
```

#### Zombie Processes

`zombie process`: A process that has terminated but the parent process has not yet read its exit status (using `wait()` or similar). In this state, the process has finished but still occupies an entry in the process table until the parent collects its status. 
Cannot be killed using the `kill` command like a regular process.

To list zombie processes:

``` bash
ps aux | grep defunct or Z
```

```
[kimchen@rhel9 rhcsa]$ ps fax | grep -B5 zombie
   2296 ?        S      0:03  |   \_ sshd: kimchen@pts/0
   2301 pts/0    Ss+    0:00  |       \_ -bash
   2555 ?        Ss     0:00  \_ sshd: kimchen [priv]
   2560 ?        S      0:00      \_ sshd: kimchen@pts/1
   2564 pts/1    Ss     0:00          \_ -bash
   2624 pts/1    S      0:00              \_ ./zombie
   2625 pts/1    Z      0:00              |   \_ [zombie] <defunct>
```

Once the parent acknowledges the exit status, the process is fully **reaped**, and all resources (such as memory) are released. The process is then removed from the system and is no longer visible in the process table.

To send a signal to the parent process to remove its child processes

``` bash
kill -SIGCHLD 'PARENT_ID'
kill -9 'PARENT_ID' # if upper doesn't work
```

#### Using TOP

**top** states:

* **R**: process is active, running, using CPU time or waiting in the queue to get serviced
* **S**: sleeping. The process is waiting for an event to finish
* **D**: uninterruptible sleep. When a process is waiting for **I/O**. A.k.a **blocking state**
* **T**: stopped. Usually by the CTRL+Z sequence
* **Z**: zombie. The process has been stopped and cannot be removed by its parent process. In unmanageable state

Processes can be killed while in `top` mode by pressing the K key and entering the PID

Get info about load average:
