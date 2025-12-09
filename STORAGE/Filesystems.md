[[gitea/LINUX 1/STORAGE/Devices]]

**Storage filesystems:**
- **xfs**: default filesystem for RHEL9
- **ext4**: default filesystem for pre-RHEL9
- **ext3** and less: not supported in RHEL
- **ntfs**: not supported in RHEL
- **vfat**: virtual FAT, kernel filesystem driver for FAT16 or FAT32

**Special filesystems:**
- `procfs`: Provides a view into the kernel and process information
	- Mounted at `/proc`. 
	- Populated **per-PID namespace** by the kernel.
- `sysfs`: Exposes **kernel objects and device information**
	- Mounted at `/sys`. 
	- Attributes can be read/written to control kernel behavior for certain devices
	- Example: `cat /sys/class/net/eth0/address` -> `52:54:00:28:08:99`
- `tmpfs`: Temporary RAM-based filesystem.
- `devtmpfs/devpts`: Kernel-managed device nodes
	- Mounted at `/dev`
	- Provides `/dev/null`, `/dev/sda`, etc
	- `devpts`: Provides Pseudo Terminal Devices
- `cgroup`: Exposes control groups for process resource management.
	- Mounted under `/sys/fs/cgroup`
- `securityfs`: Provides a kernel interface for security modules (`SELinux, `AppArmor)
	- Mounted at `/sys/kernel/security`

Refer to `/proc/filesystems` for more

The available disk space of a filesystem is divided into **blocks** of specific size. 4K by default. A block can hold a whole file or part of it. A 1K file will use up an entire 4K block. Block size should be selected according to the server's specific purpose.

[inode](https://en.wikipedia.org/wiki/Inode): where POSIX-compliant filesystems store metadata such as:
* creation time
* permissions and ownerships
* pointers to which blocks contain the actual data

>A filesystem can contain as many files and dirs as it has `inodes` for. Tiny block size and lots of files can cause the system to run out of `inodes` before it runs out of space, which is equally bad.


#### Journaled Filesystems

A journaled filesystem makes use of a circular buffer, which hold information of changes, not yet committed to the main part of the main filesystem storage. Any Information that needs to be written on disk is kept in the journal and processed only when the system is available to do so. In case of a system crash, info kept in the journal is not lost, but processed on next filesystem remount. Which means, partial changes are not applied and data is protected from corruption.

#### Listing Installed Filesystems

- `cat /proc/filesystems`
- `nodev`: pseudo filesystems managed by `systemd`

#### Listing Block Devices and Underlying Filesystems

[[LINUX/HARDWARE/Collecting Hardware Information#Listing Partitions and Hard Disks with lsblk]]

man `lsblk`

`sd`: SCSI (Small Computer System Interface) device

Get a full of list columns

``` bash
lsblk --help
```

``` bash
lsblk -o "NAME","FSTYPE"
```

```
[root@server2 block]# lsblk -o NAME,FSTYPE
NAME          FSTYPE
sda
├─sda1        xfs
└─sda2        LVM2_member
  ├─rhel-root xfs
  └─rhel-swap swap
sdb
├─sdb1        xfs
├─sdb2        xfs
├─sdb3
└─sdb4        ext4
sdd
└─sdd1        vfat
sr0           iso9660
```

Query a single disk

``` bash
lsblk -o "NAME","FSTYPE" "/dev/sdb"
```

```
[root@server2 block]# lsblk -o NAME,FSTYPE /dev/sdb
NAME   FSTYPE
sdb
├─sdb1 xfs
├─sdb2 xfs
├─sdb3
└─sdb4 ext4
```

Check the filesystem of a particular directory

``` bash
df -Th "/DIR"
```

Return the amount of space used by a directory

``` bash
du -h [-s] 'DIR' [--exclude='DIR']
```

`-s`: Returns space used just by the directory

#### Managing EXT4 Filesystems

**man tune2fs**
**man mkfs**
**man e2label**
**man mke2fs**

Packages:
- `e2fsprogs`: tune2fs, mke2fs, e2label
- `util-linux`: mkfs

Volumes should be unmounted prior to formatting
For a list of supported filesystem types:

```bash
cat /etc/mke2fs.conf
```

`tune2fs`: tool for managing `ext4` filesystems

To get a list of ext4 filesystem properties:

``` bash
tune2fs -l "/dev/PARTITION"
dumpe2fs -h "/dev/PARTITION"
```

```
[kimchen@rhel9 ~]$ sudo tune2fs -l /dev/sdd1
tune2fs 1.46.5 (30-Dec-2021)
Filesystem volume name:   <none>
Last mounted on:          <not available>
Filesystem UUID:          be4b2413-5214-4a15-9919-8b9bb8d41e79
Filesystem magic number:  0xEF53
```

Format a volume with **ext4** fs

``` bash
mkfs.ext4 [ -L "LABEL"] "DEVICE"
mkfe2fs "DEVICE" -t ext4
```

Filesystem volume name: label
To set a label:

``` bash
tune2fs -L  "LABEL" "/dev/PARTITION"
e2label "DEVICE" "LABEL"
```

Get the device label

``` bash
e2label "DEVICE"
```

##### Configuring the ext4 Journal Mode

**man mke2fs**
**man tune2fs**

- `data=ordered`: Default journal mode, journals metadata only
- `data=journal`: Safest mode. All data is forced directly into the journal prior to being written into the main filesystem. Best chances of recovering data in case of failure. Changes written twice.
- `data=writeback`: fastest, least safe

##### Improving Performance with an External Journal for ext4

Requirements:
* external journal disk and data disks must be on the same system
* similar disk speeds or the external journal can be faster
* disk block sizes must be the same

To find the filesystem block size

``` bash
tune2fs -l "DEVICE" | grep -i "BLOCK_SIZE"
```

To format a partition as a journal block device

```bash
mke2fs -O journal_dev  -b 4096 \[ -L volume-label ] 'PARTITION'
```

To format a volume with ext4 fs and attach the external journal to it

```bash
mkfs.ext4 -b 4096 \[ -L volume-label ] -J device='JOURNAL_DEVICE' 'PARTITION'
```

The external journal device fs-type is shown as **jbd**

```
$ lsblk -o NAME,FSTYPE,TYPE
NAME          FSTYPE      TYPE UUID
sda                       disk
├─sda1        xfs         part 3f6361e8-3988-4eeb-8891-43ea4a49f910
sdb                       disk
├─sdb1        jbd         part 6e6f7031-ce5c-4b47-b7d8-e5bcad2cc684
└─sdb2        ext4        part edebb976-5145-4264-9fa8-4fa427e3763c
```

To attach an external journal device to an existing filesystem, first clear its existing journal using "^" before the feature to disable it. **man tune2fs**

```bash
tune2fs -O ^has_journal** *VOLUME*
mke2fs -b SAME_BLOCKSIZE -J device='JOURNAL_DEVICE' 'VOLUME'
```

##### Finding Which Journal System is Attached to

man **dumpe2fs**
**man tune2fs**

A Linux volume with an internal journal

```
[root@localhost ~]# dumpe2fs -h /dev/sda2 | grep -i uuid
Filesystem UUID:          d1db1852-6492-4d52-8a80-ec62824ddbbc
```

The device `/dev/sdb2` shows that it has an externally-attached journal disk 6e6f70... to it

```
$ sudo dumpe2fs -h /dev/sdb2 | grep -i uuid
dumpe2fs 1.43.8 (1-Jan-2018)
Filesystem UUID: edebb976-5145-4264-9fa8-4fa427e3763c
Journal UUID: 6e6f7031-ce5c-4b47-b7d8-e5bcad2cc684
```

To check the current journal mode. Mounted fs only

``` bash
dmesg | grep "DEVICE"
```

```
[root@localhost ~]# dmesg | grep sdb1
[    5.235164]  sdb: sdb1
[   11.315726] EXT4-fs (sdb1): mounted filesystem with ordered data mode. Quota mode: none.
```

To change to **journal=data** mode

``` bash
tune2fs -o journal_data "DEV"
```

Unmount and remount the volume and check **dmesg**

```
[ 2854.928562] EXT4-fs (sdb1): mounted filesystem with journalled data mode. Quota mode: none.
```

If there are multiple lines with conflicting information, reboot

##### Freeing Space from Reserved Blocks on ext4

Most Linux ext4 systems reserve about 5% of the disk space for the root user and system services.

To find out the number of reserved blocks

``` bash
tune2fs -l | grep reserved
```

Do decrease the percentage to 1

``` bash
tune2fs -m 1 "DEVICE"
```

To specify exact number of reserved blocks

``` bash
tune2fs -r "RES_BLOCK_NUM" "DEV"
```

#### Managing XFS Filesystem 

**man xfs_admin**: tool for managing `xfs` filesystems

Package:
`xfsprogs`

XFS developed by Silicon Graphics for IRIX. Supports expanding only.

Tools: 
- `xfs_TAB_TAB` for all commands
- `xfs_repair`: helps repair damaged or corrupt fs
- `xfs_growfs`: expands the XFS 
- `xfs_freeze`: useful when making snapshots

Make an xfs filesystem

``` bash
mkfs.xfs -L "LABEL" "PARTITION"
```

To change the filesystem label

``` bash
xfs_admin -L "LABEL" "DEVICE/PART"
```

To resize an xfs filesystem, after resizing the partition first

``` bash
xfs_growfs -d "/dev/PARTITION"
```

#### Creating BTRFS Filesystems

https://wiki.gentoo.org/wiki/Btrfs

man 8 btrfs-filesystem

Package:
`btrfs-progs`

`Btrfs` uses COW (Copy-On-Write). When modifying data, it is copied, modified and then written to a new free location, rather than modifying it in-place. The metadata, the location of the file is then updated in the same way to reflect the new location of the data.

To format a device with `btrfs` 

``` bash
mkfs.btrfs "/dev/DEVICE"
```

To create a btrfs RAID partition

``` bash
mkfs.btrfs -d raid10 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

- `-d`: creates the profile for the data block groups
- `-m`: creates the profile for the metadata block groups

##### Subvolumes and Snapshots

A subvolume is a POSIX-namespace or a container, not a block device. 

``` bash
btrfs subvolume create /data/mail
```

Create new `/srv/subvol` folder and mount the mail sub volume

``` bash
 mount -t btrfs -o subvol=mail /dev/sdc /srv/subvol
```

``` bash
df -h /srv/subvol
#
Filesystem      Size  Used Avail Use% Mounted on
/dev/sde        2.2G   17M  1.9G   1% /srv/subvol
```


#### Creating exFAT Filesystem

Package: 
`exfatprogs`

**FUSE**: Filesystem in User Space

Create the exFAT filesystem

``` bash
mkfs.exfat -L "LABEL" "DEVICE"
```

To change an exFAT filesystem label

``` bash
exfatlabel "DEVICE" "LABEL"
```

#### Creating FAT16 and FAT32 Filesystems

Package:
`dosfstools`

Max file size: 4GB

Max partition size: 16TB

Create a FAT 32 filesystem

``` bash
mkfs.fat -F 32 -n "LABEL" "DEVICE"
```

#### SWAP

To see the current swap space usage:

`free -m`

```
[kimchen@rhel9 ~]$ free -m
        total        used        free      shared  buff/cache   available
Mem:    1763         678         713           7         525        1085
Swap:   2047           0        2047
```

To Increase the current swap disk

In **fdisk** or **gdisk**, create a new partition of type **swap**. Code 82, or 8200

Format the newly created partition:

``` bash
mkswap /dev/"PARTITION"
```

Activate the newly created swap space

``` bash
swapon /dev/"SWAP_PARTITION"
```

The new space is automatically allocated to the swap pool

```
[kimchen@rhel9 ~]$ free -m
               total    used      free    shared  buff/cache   available
Mem:            1763     678       713         7         525        1085
Swap:           3071       0      3071
```

To make sure the swap space is available after a reboot:

``` bash
echo "/dev/'SWAP' none swap defaults 0 0" | sudo tee -a /etc/fstab
```

##### Adding Swap Files

In case of urgency and when there is no space for a new free partition, swap files can be used

To create a swap file by adding 100 blocks of size 1MiB to the **/swapfile**:

``` bash
dd if=/dev/zero of=/swapfile bs=1M count=100
```

Format the swapfile

``` bash
mkswap /"SWAP_FILE"
```

Activate the swapfile

``` bash
swapon /"SWAP_FILE"
```

```
[kimchen@rhel9 ~]$ file /swapfile
/swapfile: Linux swap file, 4k page size, little endian, version 1, size 51199 pages, 0 bad pages, no label, UUID=297d018b-238e-43cd-bc0e-975657a658d8
```

To deactivate a swap device:

``` bash
swapoff /dev/"SWAP_DEV|FILE"
```

#### fsadm Utility

`fsadm` — utility to resize or check filesystem on a device

#### Recovering from Failure

> [!important]
> When repairing the root filesystem by running a tool from the root filesystem itself, make sure to mount the filesystem read-only first otherwise the filesystem will be modified directly by the repair tool without the kernel knowing about it

Filesystem metadata is kept in backup superblocks as well

To dump all superblocks on an ext4 filesystem

``` bash
$ sudo dumpe2fs /dev/sdg1 | grep Backup
```

```

Journal backup: inode blocks
Backup superblock at 32768, Group descriptors at 32769-32769
...
Backup superblock at 819200, Group descriptors at 819201-819201
Backup superblock at 884736, Group descriptors at 884737-884737
Backup superblock at 1605632, Group descriptors at 1605633-1605633
```

Run the e2fsck with specifying a superblock

``` bash
e2fsck -b 32768 /dev/"DEVICE"
```

#### OverlayFS

`OverlayFS` is a Linux union filesystem that allows you to combine a **read-only directory (lower layer)** with a **writable directory (upper layer)** to create a **merged filesystem** that behaves like a single writable directory.

Major Uses:
- Live systems
- Embedded devices with read-only firmware (OpenWrt)
- Immutable operating systems (Fedora Silverblue, Talos)
- Creating temporary test environments
- Forensics and safe image analysis
- Snapshotting / easy reset or rollbak
- Build systems needing isolated writable roots

Structure

```
Container sees: merged/
      ┌─────────────────────┐
      │       merged/       │  ← union of lower + upper
      └─────────────────────┘
             ▲
             │
     ┌───────┴────────┐
     │                │
  upper/            lower/
(writable)       (read-only base)
     │                │
     └───────┬────────┘
             │
          work/  ← scratch area for copy-on-write
```

- `lower/`: Read-only base image, `rootfs`
- `upper/`: Writable container changes (COW layer). 
- `work/`: Internal scratch for `OverlayFS` during writes. Whenever a file is modified from the lower layer, `OverlayFS` copies the file from `lower/` to `upper/` (copy-on-write). 
- `merged/`: Combined (Union) view of `upper` and `lower`.

Technically, the `lowerdir` allows writes, but they do not propagate to the source and `merged` mounts. To avoid confusion, it can be re-mounted as read-only.

Rule of thumb:
- Only interact with `merged/`. Use for container root.
- Only look at `upper/` to inspect changes
- !!! Do not touch `work/`!!!

> Do not touch the work directory, otherwise it becomes corrupted and changes are no longer reflected properly! Remake and remount `upper` and `work` if it is corrupted.

Load the `overlay` module

```bash
modprobe overlay
```

##### Example: Build a Simple Root Sandbox

Create the overlay structure

```bash
mkdir -p /overlay/{upper,work,merged}
```

Mount `overlayfs`. 

```bash
mount -t overlay overlay -o \ lowerdir=/,upperdir=/overlay/upper,workdir=/overlay/work /overlay/merged
```

Change the root filesystem to the merged overlay

```bash
chroot /overlay/merged /bin/bash
mount -t proc proc /proc
mount -t sysfs sysfs /sys
```

```
TARGET  SOURCE  FSTYPE  OPTIONS
/       overlay overlay rw,relatim.../upper,workdir=/overlay/work
├─/proc proc    proc    rw,relatime
└─/sys  sysfs   sysfs   rw,relatime
```

```bash
echo "192.168.99.14 test.com" >> /etc/hosts
```

```bash
ping test.com
# PING test.com (192.168.99.14) 56(84) bytes of data.
```

Open another terminal session. The modified file appears in the `upper` directory.

```bash
cat /overlay/upper/etc/hosts
# 192.168.99.14 test.com
```

The change is not present in the original `/etc/hosts` file. 

```bash
ping test.com
# PING test.com (34.224.149.186) 56(84) bytes of data
```

To rollback the changes, simply delete the files in the `upper` directory.

```bash
rm -rf /overlay/upper/etc/hosts
```

The entry is gone.

>Deleting files creates **whiteouts** in the `upper` directory, which do not actually delete the files but obscure them. To bring back the files, the `upper/` directory must be purged and the overlay filesystem must be torn down and remounted.

`whiteout`: A _character device file_ with device number 0/0. It is also stored in memory, not only on disk, that's why deleting it does not automatically bring back 'deleted' files in the merged layer, until the overlay filesystem is remounted.

For better isolation, mount the overlay filesystem in a new namespace.

> `overlayfs` mounts start out `private` even in `shared` parents.