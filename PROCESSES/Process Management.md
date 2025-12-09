
Types of processes

* **shell job**: *interactive processes*. Commands associated to the shell they were launched from.
* **daemons**: processes that provide *services*. Launched when the system boots. Often run with root priv
* **kernel threads**: part of the Linux **kernel**. Good to be monitored. KT names are in square brackets **\[ kernel thread ]**. KTs cannot be managed or killed.

**thread**: a task started by a process that a dedicated CPU services. No tools to manage individual threads. Thread management should be taken care of from within the command.
#### Managing Shell Jobs

When commands are executed, **shell jobs** start as a **foreground** process by default. 
Programs that take long time to complete might be a good idea to be run in the background, thus leaving the shell free to use for something else.
To bring a shell job to the background add **&** at the end of the command

``` bash
'COMMAND' &
```

To bring a shell job from the background back to the foreground use **fg**

To list jobs. -l shows job ID

``` bash
jobs -l
```

To bring back a specific job to the background:

``` bash
fg 'JOB_NUMBER'
```

**CTRL + Z**: **pauses** a job, so it can be managed. 
**CTRL + C**: **cancels** the current job and removes it from memory
**CTRL + D**: the jobs stops waiting for further input and terminates the job

```
[kimchen@rhel81 ~]$ jobs
[5]   Running                 sudo tail -f /var/log/secure &
[6]   Running                 sleep 3600 &
[7]   Running                 dd if=/dev/zero of=/dev/null &
[8]+  Stopped                 sleep 7200

```

To resume a stopped job in the background
**$ bg 8**

#### Parent-child Relations

`A process is a child of the shell parent that started it. When a shell is closed, child processes are closed too. `

**Jobs running in the background are not automatically closed when the shell has been closed!**
**When a background process parent shell is terminated, the child process becomes child of the systemd !!!**

To see all shell job processes. Shown at the top
**$ top**

To kill a process, type **k** while in **top**. Enter **PID**, hit **ENTER**

#### Common Command-line Tools for Process Management

```
[kimchen@rhel81 ~]$ ps aux | head
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.7 245252 13696 ?        Ss   08:49   0:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
root           2  0.0  0.0      0     0 ?        S    08:49   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   08:49   0:00 [rcu_gp]
```