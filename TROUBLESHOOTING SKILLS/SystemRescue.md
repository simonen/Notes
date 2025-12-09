
#### Getting Started with SystemRescue

Download from https://system-rescue.org/
[[LINUX/FILES AND DIRECTORIES/Linux Directory Structure#CHROOT]]

**SecureBoot** must be disabled for **SystemRescue** to work
Automatically logged in as root, no password
Removable media can be set up as default booting device

UEFI and BIOS boot screens

To launch a Xfce Desktop

``` bash
startx
```

`SquashFS`: the basis of many LiveCD distributions

A `SystemRescue` supports only limited commands because it does not mount pseudo filesystems like `/proc` or `/sysfs` which exist only in memory.
#### Resetting Root Password

For more info check **SystemRescue pam** configuration
**man 7 pam**

Identify the root filesystem

```
lsblk -f
```

Mount the root filesystem in a new dir

``` bash
mkdir "/mnt/linux" && mount /dev/"ROOT_DEV" "/mnt/linux"
```

Change from the SystemRescue filesystem to the mounted root fs

``` bash
system rescue> chroot /"NEW_ROOT_DIR" /bin/bash
```

Change the root pass

``` bash
passwd root
```

If the system is SELinux enabled

```
touch /.autorelabel
```

Exit and the chroot env and reboot

#### Enable SSH in SystemRescue

SSH is enabled by default, along with firewall. Disable the firewall

``` bash
systemctl stop iptables
```

By default, the root has no password on a **SystemRescue**. Make sure it has prior to SSH-ing.
Root password can be set as a boot param. From the **SystemRescue** menu press TAB or 'e' to open the boot options window and add `rootpass=PASSWORD nofirewall` at the end of the line starting with **linux** .CTRL + x to boot.

`scp` and `sshfs` can be used

#### Repairing GRUB from SystemRescue

If the system boots to grub rescue> -> grub might be missing. Should be re-installed

Boot up **SystemRescue**, create a chroot environment and install grub

Create a mountpoint and mount the root filesystem on it

``` bash
mkdir /mnt/linux && mount /dev/"ROOT_DEV" /mnt/linux
```

The root filesystem is now mounted but the `/dev` `/sys` and `/proc` directories are empty. That's because pseudo filesystems like `procfs`, `sysfs`, etc.,  are initialized in memory during boot that never happened. `SystemRescue` has its own pseudo filesystems which can be borrowed by `binding` them to the chrooted env.

``` bash
mount -o bind /proc /mnt/linux/proc
mount -o bind /sys /mnt/linux/sys
mount -o bind /dev /mnt/linux/dev
```

```
TARGET  SOURCE    FSTYPE   OPTIONS
/       /dev/sda1 ext4     rw,relatime
├─/proc proc      proc     rw,nosuid,nodev,noexec,relatime
├─/dev  dev       devtmpfs rw,nosuid,relatime,size=1962568k,nr_inodes=490642,mode=755,inode64
└─/sys  sys       sysfs    rw,nosuid,nodev,noexec,relatime
```

Change to the new environment

```
chroot /mnt/linux /bin/bash
```

Reinstall GRUB

``` BASH
grub-install "BOOT_DEV"
```

Repair GRUB

``` bash
update-grub2
```

Exit the chroot environment, unmount all chroot systems and reboot

#### Resetting Windows Password

Mount the windows partition 

``` bash
mount /dev/"WINDOWS_DEV" /mnt/windows
cd /mnt/windows/Windows/System32/config
```

Get help

``` bash
chntpw -h
```

List all Windows users

``` bash
chntpw -l SAM
```

```
[root@sysrescue /mnt/win10/Windows/System32/config]# chntpw -l SAM
chntpw version 1.00 140201, (c) Petter N Hagen
Hive <SAM> name (from header): <\SystemRoot\System32\Config\SAM>
ROOT KEY at offset: 0x001020 * Subkey indexing type is: 686c <lh>
File size 65536 [10000] bytes, containing 7 pages (+ 1 headerpage)
Used for data: 318/32368 blocks/bytes, unused: 20/12464 blocks/bytes.

| RID -|---------- Username ------------| Admin? |- Lock? --|
| 01f4 | Administrator                  | ADMIN  | dis/lock |
| 01f7 | DefaultAccount                 |        | dis/lock |
| 01f5 | Guest                          |        | dis/lock |
| 03e8 | kimchen                        | ADMIN  |          |
| 01f8 | WDAGUtilityAccount             |        | dis/lock |
```

Enter user options prompt

``` bash
chntpw -u "USERNAME" SAM
```

Follow instructions, reset pass, quit, write changes, unmount, reboot. Change pass

#### Rescuing a Failing HDD with GNU ddresecue

GNU ddrescue, by Antonio Diaz

**ddrescue** tries to rescue good data, skips bad data. Does multiple passes. Works at block level, i.e filesystem doesnt matter

The backup drive should be at least 1.5 times bigger. Both filesystems should be unmounted

``` bash
ddrescue -f -n /"SOURCE" /"DEST" ddlogfile
```

Run three more passes to try saving more data

``` bash
ddrescue -d -f -r3 /"SOURCE" /"DEST" ddlogfile
```

- `-rN`: retry passes
- `-f`: force
- `-n`: no scrape

Run a filesystem check on the unmounted recovery disk

``` bash
e2fsck -vfp /"RECOVERY_DISK"
```

#### Creating a Data Partition on SystemRescue USB Drive

Package:
`syslinux`

Utilities: 

```
isohybrid
mbr.bin
```

Make the USB SystemRescue bootable and usable as a recovery drive at the same time. To do this, the SystemRescue ISO must be made bootable from a partition.
A writable partition on USB allows to make persistent configuration changes to SystemRescue

Make the SystemRescue bootable from a partition

``` bash
isohybrid --partok "SR_IMAGE".iso
```

Create an msdos partition table on the USB

```bash
parted /dev/*USB* mklabel msdos
```

Create the SR partition FAT32 formatted, 2G size, and set the **boot flag**

```bash
parted /dev/'USB' mkpart 'sysrec' fat32 1MB 2000MB
parted /dev/'USB' set 1 boot
```

Create the data partition. Type xfs

```bash
parted /dev/'USB' mkpart "PARTITION LABEL" xfs 2001MB 'WHATEVERMB'
```

Create the FAT32 on Partition 1 on the USB

```bash
mkfs.fat -F 32 -n SYSRESCUE /dev/'USB_PART_*1'
```

Create the XFS on Partition 2 

``` bash
mkfs.xfs -L data /dev/USB_PART_2
```

Install **SystemRescue** to Partition 1

``` bash
dd status=progress if="SR_IMAGE".iso of=/dev/"USB_PART1"
```

Install the MBR to the USB 

``` bash
dd if=/usr/share/syslinux/mbr.bin of=/dev/"USB"
```

On Debian

``` bash
install-mbr /dev/"USB"
```

Verify the USB structure

```
[root@server33 tmp]# fdisk -l /dev/sdc

Disk /dev/sdc: 31.5 GB, 31457280000 bytes, 61440000 sectors
Disk label type: dos
Disk identifier: 0x000a13a7

   Device Boot      Start         End      Blocks   Id  System
/dev/sdc1   *        2048     3905535     1951744    c  W95 FAT32 (LBA)
/dev/sdc2         3907584     7813119     1952768   83  Linux
```

#### Preserving Changes in SystemRescue

To customize a SystemRescue, changes need to be made persistent

Add the boot parameter `cow_label=data` in the selected boot option

The configuration is stored in `/run/archiso/cowspace/persistent_RESCUE`

