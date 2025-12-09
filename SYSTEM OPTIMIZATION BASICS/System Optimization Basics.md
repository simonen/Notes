---
tags:
  - centos
---

> Do **NOT** try out settings on a production server!
> Change settings one at a time. Test 3 times
> Make a good plan

#### The /proc Filesystem

The `/proc` filesystem is a pseudo (virtual) filesystem that provides a mechanism for the kernel to expose information about the system and running processes. It does not store real files on disk but acts as a window into the kernel data structures.

Tools like `lscpu`, `uname`, `top`, `ps` get info from the `/proc` directory

##### Using /proc/sys to Optimize Performance

The `/proc/sys` contains virtual files that can be modified (tunables)

* **abi**: provides an interface to applications, particularly non-open source. Rarely used
* **crypto**: provides cryptographic interface to specific services like IPsec. Rarely used for pt
* debug
* **dev**: tunables for devices
* **fs**: interface to the virtual filesystem. Contains **file-max**, controlling max num of open files
* **kernel**: the kernel interface. Contains useful tunables
* **net**: network interface. Contains useful tunables
* **sunrpc**: tunables related to **NFS** file sharing
* **vm**: virtual memory interface. Contains useful tunables

`vm.swappiness [0-100]:` Willingness of system to swap data

Methods of changing a parameter:
* **echo** to write the parameter to the kernel tunable
* `sysctl -w "PARAM=value"`

When optimizing:
1. Write the parameter in the `/proc/sys` file system in a non-persistent way
2. Test and confirm it is working
3. Write permanently

To temporarily change a parameter (machine's hostname):

``` bash
echo "pipa.server.lsaa.lab" > /proc/sys/kernel/hostname
```

Changes are not persistent after reboot

#### Using sysctl

`sysctl` - configure kernel parameters at runtime

The parameters put in `sysctl` are directly related to the `/proc/sys` files. 
`/proc/sys/swappiness` = `vm.swappiness` in `sysctl`

 While the system is booting `sysctl` reads from the following `.conf` files:
`/etc/sysctl.conf`: Default conf file. Should be used no longer
`/usr/lib/sysctl.d`: Default settings. Managed by the system, not manually
`/etc/sysctl.d`: Modification overrides go here. Persistent after reboot.

To read a particular value

``` bash
sysctl "KEY"
```

Some `sysctl` options:
* `-a`: Displays all tunables
* `--system`: Loads settings from all conf filesq
* `-p <filename>`: Reads values from a specified file. If no filename specified,   `/etc/sysctl.conf` is used
* `-w`: Writes a new value to a tunable

To temporarily block ping requests using `sysctl`:

``` bash
sysctl -w "net.ipv4.icmp_echo_ignore_all=1" 
```

To make changes persistent after reboot, make a drop-in file in the `/etc/sysctl.d/`

``` bash
echo "net.ipv4.icmp_echo_ignore_all=1" > /etc/sysctl.d/ping.conf
```

#### Storage

RAID rebuild speed limits
`dev.raid.speed_limit_min`
`dev.raid.speed_limit_max`

`relatime`: This mount option tells the filesystem not to update file access time stamps

`dir_index`: `ext2,3,4` filesystem feature. Speeds up access to directories with lots of files and subs

``` bash
tune2fs -l "DEV" # list fs features
tune2fs -O dir_index "DEV" # turn it on
tune2fs -O ^dir_index "DEV" # turns it off
```

#### I/O Schedulers

I/O schedulers (elevators) are algorithms the kernels uses to order I/O to disk subsystems. 

* `Cfq`: (Completely Fair Queueing) Deprecated. 
* `[mq]Deadline`: Designed to ensure that read and write requests are processed within a certain time frame, thus avoiding starvation of any request. It's often used for spinning disks (HDDs) and may perform well on SSDs depending on the workload.
* `Noop (none)`: Essentially means there is no active scheduling of I/O requests. It is often used with SSDs because these devices generally don't benefit from complex scheduling, given their near-instant access times.
* `bfq`: (Budget Fair Queueing). I/O scheduler that provides fair bandwidth distribution between different processes. It is designed to improve the performance of desktop systems by ensuring that processes have equitable access to the disk, reducing latency in interactive applications.
* `kyber`: Optimized for **fast storage devices** like SSDs. It aims to provide a balance between low latency and high throughput, using multiple queues for different types of I/O operations (e.g., read/write) to achieve better performance on modern storage systems.

To find the scheduler of a device

``` bash
cat /sys/dev/sdb/queue/scheduler
# none [mq-deadline] kyber bfq (active scheduler in [])
```

To change a scheduler

``` bash
echo bfq > /sys/block/sdb/queue/scheduler
```

To make changes persistent, modify the boot-loader

`/etc/default/grub`
``` bash
GRUB_CMDLINE_LINUX="... elevator=SCHEDULER"
```

Update grub

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg # EFI
```
#### Diagnosing Slow Startups

Get a list of processes and how long it takes for them to start

```
systemd-analyze blame
```

#### Resource Limits

man `ulimit` 

The `ulimit` command sets resource limits to the current shell.

To report current limits

``` bash
ulimit -a
```

To limit the number of allowed processes for a shell

``` bash
ulimit -u 'NUMBER'
```

```
open files                          (-n) 1024
cpu time                   (seconds, -t) unlimited
max memory size             (kbytes, -m) unlimited
```

`-m "MEMORY"`: Sets a limit on memory
`-n "FILES"`: Sets a limit on open files

Setting limits on users can be done via `/etc/profile` or `/etc/bashrc`

or using the `limits` module for PAM. The module is invoked when a user creates a new process. The file defines both hard and soft limits for users and groups.

`/etc/secure/limits.conf`
```
#<domain>      <type>  <item>         <value>
#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4
```

A `hard` limit can be set only by root. The system will prevent breaking the limit
A `soft` limit can be adjusted by a user with `ulimit` but is limited to the value of the hard limit

```
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
```

The faculty group is allowed to run 20 concurrent processes (nproc) according to their soft limit. Any member of the group is allowed to change the limit to any value up to 50.