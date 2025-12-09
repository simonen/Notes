
The Kernel is the layer between the user and the hardware of the system
Takes care of essential system tasks such as the task scheduler. Hardware initialization

#### Kernel Threads and Drivers

Kernel threads are placed in square brackets in the **ps aux** command output
Drivers are loaded as kernel modules.

**tainted kernel**: kernel that contains closed source drivers. Helps in identifying driver issues

#### Kernel Ring Buffer

The kernel writes messages about its activities to a ring buffer. Old messages are dropped out when the buffer is full. Time indicator is relative to the start of the kernel

The Kernel Ring Buffer holds info on:
* how the kernel was loaded
* the hardware the kernel detected and related actions

Tools for analysis
* `dmesg`: shows the contents of the kernel ring buffer with most recent messages. Time indicator is relative to the start of the kernel
* `journalctl -k`, `journalctl --dmesg`: Same as `dmesg`, give events in absolute clock time
* The `/proc` file system: an interface to the kernel. Gives detailed status info about the system
* `uname`: info about the OS. -r: kernel version, -a: overview
* `hostnamectl`: OS info

Useful `dmesg` options
* `-C`: clears the kernel ring buffer
* `-H`: human readable format
* `-T`: human-readable timestamp
* `-w`: wait for new messages
* `-l <1-9>`: show messages of certain level: info, crit, err, emerg...

#### Kernel Modules

kernels are modular since 1996

Functionality, loaded as modules:
* **drivers**
* **file system support**
* **other features**

#### Hardware Initialization

* `systemd-udevd`: the process loads a driver upon detection, making the hardware device available. Continuously monitors hardware changes
* `/usr/lib/udev/rules.d`: rules on how devices are initialized. Should not be modified.
* `/etc/udev/rules.d`: Custom rules. Read after rules in `/usr/lib/udev/rules.d`
* `/sys`: pseudo `sysfs` file system to track hardware-related settings and where status about kernel modules is written. Each phase of hardware probing is concluded in a file in `/sys`

To list all events that are processed when loading or unloading a hardware:

``` bash
udevadm monitor
```

#### Managing Kernel Modules

Common commands used for manual kernel management:
* `lsmod`: Lists currently loaded kernel modules
* `modinfo MODULE_NAME`: Displays info about kernel modules
* `modprobe`: Loads kernel modules, including their dependencies
* `modprobe -r`: Unloads kernel modules, considering dependencies. Can unload only modules that are NOT currently being used.

Alternative method of loading kernel modules 
+ `/etc/modules-load.d`: Manually create files here for modules that are not automatically loaded by `systemd-udevd`
- `/usr/lib/modules-load.d`: For default modules that should always be loaded

`insmod` and `rmmod` are legacy utilities that do not address dependencies and should **NOT** be used

`/boot/config-kernel-version`: Kernel config file. To search for a module within the file

```bash
grep -i 'MODULE' *config-5.8.0-45-generic* 
```

- MODULE=m
- MODULE=y 
- `m`: Loadable kernel module. Can be added to `/etc/modules`
- `y`: module is built-in to the kernel, should not be added to `/etc/modules`

To see if it is currently loaded

```bash
lsmod | grep 'MODULE'
```

#### Checking Driver Availability for Hardware Devices

- `lspci`: Shows all devices detected by the PCI bus
- `lspci -k`: Shows all modules that are used by the detected PCI devices

If a detected PCI device doesn't list any modules, probably means it is not supported. Closed sourced drivers might endanger stability. 

**Always check if the hardware device supports Linux prior to purchase!!!**

#### Managing Kernel Module Params

`/etc/modprobe.d`: Drop-in files with module parameters go here

```
echo MODULE_NAME PARAM=0 or 1 > /etc/modprobe.d/MODULE_NAME.conf
```

#### Upgrading the Linux Kernel

Upgrading the kernel installs and uses the new version but keeps the old version.
To upgrade the Linux Kernel:

``` bash
dnf upgrade kernel
dnf install kernel
```

`/boot`: the files of the 4 last kernels are kept here. The GRUB 2 bootloader picks all  versions that are located in the dir.

#### The proc Filesystem

http://en.wikipedia.org/wiki/Procfs

This is a special directory that contains a virtual, or pseudo, filesystem that provides a way of interacting with the kernel. It exists only in memory.

Internal kernel variables are accessible by the `/proc/sys` directory