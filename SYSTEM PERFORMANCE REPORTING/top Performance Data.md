**load average**: indicates average number of processes waiting in the queue to be served over the last 1, 5 and 15 minutes
This number should not be much higher than the CPU core count

`1:Def - 13:59:01 up 37 min,  1 user,  load average: 0.00, 0.01, 0.05`
##### CPU Performance Data
`%Cpu(s):  0.0 us,  0.7 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st`

* **us**, **user**: percentage of time the CPU spends handling un-niced processes started in user mode that do not interact with the kernel
* **sy**, **system**: percentage of time the CPU spends in kernel mode handling **system calls** and dev drivers
* **ni**, **nice**: percentage of time the CPU spends for **niced** processes
* **id, idle**: percentage of time the CPU spends in idle loop, i.e CPU is available
* **wa, I/O-wait**: time the CPU spends waiting for non-interruptible I/O, such as requests to disks, hard mounted NFS and tape units. 
	* **High value means slow performing storage**
* **hi, hardware interrupts**: time the CPU spends handling **hardware interrupts**. 
	* **High value may indicate hardware failure**
* **si, software interrupts**: time the CPU spends handling **software interrupts**
* **st, stolen**: percentage of stolen time. Show in a virtualized environment where VM's steal CPU time from the hypervisor

To track a selected, comma-delimited list of processes

\$ **top -p** *PID, PID, PID*
Press \= to return to the full list

##### Memory Usage

`KiB Mem :  1016860 total,   546224 free,   123616 used,   347020 buff/cache`

##### Swap Usage

If shortage of memory occurs, Inactive(anon) memory can be relocated to the swap space

**$ free -m**
```
[root@server1 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:            993         162         309           6         520         594
Swap:           819           0         819
```

**total** = used + free + cache
**available** = free + inactive file memory (cached files not having been used recently)

```
[root@server1 ~]# swapon -s
Filename                                Type            Size    Used    Priority
/dev/dm-1                               partition       839676  0       -1
```

/dev/dm-1 in this case is a LVM volume, not a partition

##### Process Memory Usage

Use the < and > keys to sort by CPU or MEM usage

```
1  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 12862 root      20   0  146148   2064   1432 R  0.7  0.2   0:00.91 top
```

**VIRT**, virtual memory: a set of pointers that set to memory addresses in the virt addr space. Total available virtual addresses on a x64 bit machine is 32TB. Each process can set uniquely reserved memory pointers
**RES**, resident memory: real memory that need to allocated in RAM
**SHR**: shared memory

##### Customizing top

'f' key: displays all stats available to top
W key:  writes the current top setup in **~/.toprc**

##### iostat, vmstat pidstat

Package:
`sysstat`

The **sysstat** package contains some monitoring tools


**iostat**: gives per device I/O statistics
	**tps**: transactions per second
	-c: shows CPU stats
	-d: shows device stats
	**$ iostat -d** *\<DEVICE>* *\<INTERVAL_SEC>* *\<POLLING_LOOPS_NUM>*
	first line stats summarize that since startup, hence the higher numbers
**vmstat**: virtual memory stats
	**procs**: number of active processes in the last loop or waiting for I/O devices
	**swap**: si - swap in, so - swap out. Bad if large number of blocks are swapped in and out
	**system**: provides about number of **interrupts** of **context switches**
	-s: summary
	**vmstat -s -S M**: displays summary in MiB
**pidstat**: reports stats about Linux tasks
	-w: rqeports task switching activity
	-p: writes activities for the selected task (PID)
	-d: reports I/O stats

**context switch**: occurs when a CPU core moves over to handling another task. Expensive in terms of performance, should be avoided if possible
	**cswch, voluntary cs**: when an application has no more active threads and can move control over to another app
	**nvcswch, non-voluntary cs**: when an app has used its time slice and the task scheduler moves it away to make space for another thing. Bad. Occur when there are more threads than CPU cores available to the system.

```
[user@server1 ~]$ pidstat -w -p 2610 2 5
Linux 3.10.0-327.el7.x86_64 (server1.example.com)       01/20/2024      _x86_64_        (1 CPU)

04:07:19 PM   UID       PID   cswch/s nvcswch/s  Command
04:07:21 PM  1000      2610      0.50      0.00  sshd
04:07:23 PM  1000      2610      0.50      0.00  sshd
04:07:25 PM  1000      2610      0.50      0.00  sshd
```


#### Configuring SAR

**sar** - System Activity Reporter. Collect, report or save system cumulative activity information
Can use polling similar to **iostat** and **pidstat**
If the sysstat package is installed, two processes are started **sa1** and **sa2**. Daily Data is written to **/var/log/sa**
Performance data is kept for 28 days by default. To change that, edit the HISTORY variable in the **/etc/sysconfig/sysstat** file

```
[root@server1 cron.d]# cat /etc/cron.d/sysstat
# Run system activity accounting tool every 10 minutes
*/10 * * * * root /usr/lib64/sa/sa1 1 1
# 0 * * * * root /usr/lib64/sa/sa1 600 6 &
# Generate a daily summary of process accounting at 23:53
53 23 * * * root /usr/lib64/sa/sa2 -A
```

To show network stats
**$ sar -n DEV**
```
05:34:38 PM       LINUX RESTART

03:40:01 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
03:50:01 PM      eth0      0.92      0.87      0.08      0.10      0.00      0.00      0.00
03:50:01 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
03:50:01 PM virbr0-nic      0.00      0.00      0.00      0.00      0.00      0.00      0.00
03:50:01 PM    virbr0      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

To show CPU stats
**$ sar -P 0**
```
05:34:38 PM       LINUX RESTART

03:40:01 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
03:50:01 PM       0      0.13      0.00      0.15      0.00      0.00     99.72
04:00:01 PM       0      0.04      0.00      0.13      0.00      0.00     99.83
04:10:01 PM       0      0.24      0.00      0.29      0.00      0.00     99.47
04:20:01 PM       0      0.18      0.00      0.19      0.00      0.00     99.63
Average:          0      0.15      0.00      0.19      0.00      0.00     99.66
```
To show I/O stats
**$ sar -b**
```
05:34:38 PM       LINUX RESTART

03:40:01 PM       tps      rtps      wtps   bread/s   bwrtn/s
03:50:01 PM      0.10      0.01      0.09      0.40      0.88
04:00:01 PM      0.10      0.06      0.05      3.31      0.36
04:10:01 PM      0.37      0.30      0.08     11.80      0.54
04:20:01 PM      0.18      0.05      0.13      2.71      1.22
Average:         0.19      0.11      0.09      4.55      0.75
```
To get swap stats
$ sar -S

$ uptime

```
[kimchen@rhel9 rhcsa]$ uptime
 14:41:41 up  4:08,  1 user,  load average: 0.00, 0.00, 0.00
```

Load average is a number showing the number of processes in running state (R) or in blocking state (D)
shows load average over the last 1, 5 and 15 minutes
Load average should not be higher than the numbers of CPU cores

Get info about CPU

```
lscpu
```

#### Checking I/O with dstat

Package
`pcp-system-tools`

`dstat --help`

To monitor specific disks' usage

``` bash
dstat -D 'DISK1','DISK2','DISK3'
```

