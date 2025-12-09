1. POST
2. Selecting Bootable Device
3. Loading the Boot Loader
4. Loading the kernel: the kernel is loaded together with **initramfs**
5. starting **/sbin/init** process
6. **initrd.target**: minimal OS environment is prepared. Mounts the root fs on /sysroot
7. switching to the root file system: systemd processes begin loading from disk
8. running the default target

#### Manually Booting from grub

Identify the root partition
**grub> ls**
```
grub> ls
(hd0) (hd0,msdos5) (hd0,msdos1)
grub> ls (hd0,msdos1)/ls 
	lost+found/ etc/ media/ bin usr/ root/ run/ sys/ mnt/ var/ tmp/

```

The root partition is `(hd0,msdos1)`

Mount the root partition

```
grub> set root=(hd0,msdos1)
```

Load the kernel
`grub> linux /boot/vmzlinux-linux-... root=/dev/sda1 ( the correct device must be selected)`

`grub> initrd /boot/initramfs-linux.... `

Boot

```
grub> boot
```

#### Finding Useful Information in Logfiles

**man dmesg**

**/var/log**: legacy location for log files

`dmesg`: reads the kernel ring buffer. Displays everything that happens at startup, hardware activity after startup, i.e attaching devices, network activity

Display kernel info in human readable timestamps

``` bash
dmesg -T 
```

To filter out device-specific information

``` bash
dmesg -T | grep -w sd*
```

```
[Sun Mar 24 18:09:47 2024] sd 9:0:0:0: [sdc] Attached SCSI removable disk
[Sun Mar 24 18:20:03 2024] sd 9:0:0:0: [sdc] Synchronizing SCSI cache
```

`dmesg -h` for all log levels and facilities

To display certain logging levels. 

``` bash
dmesg -l "LOG_LEVEL{1-9}"
```

To traverse all logs in **/var/log/** dir

``` bash
grep -ir "KEYWORD" /var/log/*
```

#### Passing Kernel Boot Arguments

In the GRUB menu:

- `e`: To pass kernel options 
- `c`: To enter full GRUB command prompt

The line that tells GRUB how to start the **kernel**

```
linux ($root)/vmlinuz-{versionnumber].el9.x86_64 root=/dev/mapper/
rhel-root ro crash kernel=[options] resume=/dev/mapper/rhel-swap
rd.lvm.lv=rhel/ root rd.lvm.lv=rhel/swap rhgb quiet
```

#### Starting a Troubleshooting Target

To enter in the following TS mode, in GRUB, press e and add one of following options at the end of the 

`rd.break`: Breaks the boot process while in `initramfs` procedure 
`init=/bin/bash`:  Root fs is mounted in `ro` mode. A shell is immediately started after kernel is loaded
`systemd.unit=emergency.target`: The bare minimum of unit files are loaded
`systemd.unit=rescue.target`: Starts more units
`single`: Tells the bootloader to boot the OS in single user mode.

#### Rescue Disks

The default rescue image is on the Linux installation disk

`/mnt/sysimage`: detected installations are mounted here when using rescue disks

To make the contents of the directory actual working environment:

```bash
chroot /mnt/sysimage
```

**chroot**: ensures path references to all configuration files are correct

#### Recreating initramfs from a Rescue Disk

If during boot the root filesystem is not mounted on the /, or no **systemd** units are started
it could be a broken **initramfs**

$ **dracut**: without any arguments, the command creates a new **initramfs** for the current 
kernel

`dracut --force`: Override the current `initramfs`

- `/usr/lib/dracut/dracut.conf.d/*.conf`: system default conf files
- `/etc/dracut.conf.d`: Custom dracut conf files
- `/etc/dracut.conf`: Deprecated

#### Recovering From File system issues

Non-existent devices or wrong UID in fstab can lead to file system errors. Use labels instead
#### Resetting Root Password

Boot into **minimal mode**, where root pass is not needed

From the GRUB menu, add the following option at the end of the line starting with linux
`init=/bin/bash` 

The root file system is in ro mode. Remount it in rw.

``` bash
mount -o remount,rw /
```

Change the root password 

`passwd`

The SELinux policy may not be loaded. In this case `passwd` will successfully change the password, but you will have the problem described in the original question above: no one will be able to log in.

Password hashes are stored in the `/etc/shadow` file. This file normally has the SELinux type `shadow_t`. However, changing the file while no SELinux policy is loaded causes the SELinux type to be removed from the file, leaving it as `unlabeled_t`. Thus, services which try to read the file to authenticate logins are no longer able to read it.

To load the SELinux policy

``` bash
/usr/sbin/load_policy -i
```

or

Make sure `/.autorelabel` file is present

Currently, the PID 1 process is `/bin/bash`, not `systemd`, and the system cannot use the `reboot` command. 

To change to `systemd`:

``` bash
exec /usr/lib/systemd/systemd
```

or

Reboot the system by

``` bash
exec /sbin/init 6
```

- `exec`: replaces the current process with the given command
- `init 6`: reboots the machine using the `kexec.target`

**man init** for all signals info
