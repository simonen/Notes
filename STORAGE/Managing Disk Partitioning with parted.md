
`parted` only manages partitioning. **Writes changes immediately to disk!**!!
GPT GUID Partition Table: Globally Unique Identifier Partition Table

>Partitions attached to active root systems like boot, home, var, should not be unmounted. Use `SystemRescue` or make changes from another Linux system

>Non-root partitions, however, should be unmounted before modification in order to be properly reread by the kernel, otherwise the `partprobe` command is used to redetect partitions.

Enter the `parted` shell

``` bash
parted
```

```
parted --help
```

To run parted commands in a regular shell

``` bash
parted /dev/"DEVICE" print devices
```

#### Viewing Existing Disks and Partitions

```
parted > print devices
parted > select *DEVICE*
parted > print
```

List unpartitioned space

```
(parted) print free
```

Partition flags

* `boot`: Indicates a MBR boot partition
* `esp`: EFI System Partition. GPT boot partition
* `diag`: Windows recovery partition
* `msftres`: MS reserved partition
* `msftdata`: GPT partition containing Microsoft filesystems like **NTFS**, **FAT**

#### Creating GPT Partitions

Creating a new partition table wipes out the entire disk!

On an unpartitioned disk

```
(parted) print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table:
Disk Flags:
```

To list available filesystem types. These are NOT filesystems on disk:

```bash
(parted) help mkpart
(parted) help mklabel
```

Create the GPT table

``` bash
parted /dev/"DEVICE" mklabel gpt
```

Create a 2GB partition, labeled "images", ext4 file system

```bash
parted /dev/"DEVICE" mkpart "images" ext4 1MB 2004MB
```

Create a second partition, named "audio" right after the previous partition, xfs file system, use up the remaining space

``` bash
parted /dev/"DEVICE" mkpart "images" xfs 2005MB 100%
```

List the newly created partitions

``` bash
(parted) > p
```

```
Number  Start   End     Size    File system  Name    Flags
 1      1049kB  2004MB  2003MB  ext4         images
 2      2005MB  21.5GB  19.5GB  xfs          audio
```

- START (begging of partition) and END points must not overlap. 
- START cannot be 0 because the first 33 sectors are reserved for EFI label
- END can be an int value or percentage of the remaining space

#### Removing Partitions

List the disks to get partition numbers

```
parted /dev/DISK p
```

Remove the partitions

``` bash
parted /dev/DISK rm 'PART_NUMBER'
```

#### Recovering Deleted Partitions

Recovery must be performed asap or temporarily shutdown the affected disk
The exact start and end sectors are needed 

```bash
parted rescue /dev/DISK *START END*
parted /dev/sdb1 rescue** 1048kB 2004MB
```

#### Increasing Partition Size

There has to be free space at the end of the partition you want to increase. The filesystem must also be resized to fit the new partition size.

To get partition free space

```bash
parted /dev/DISK print free
```

```
Number  Start   End     Size    File system  Name       Flags
        17.4kB  1049kB  1031kB  Free Space
 1      1049kB  2004MB  2003MB  xfs
        2004MB  2005MB  1049kB  Free Space
 2      2005MB  4008MB  2003MB  xfs          aodio
        4008MB  5016MB  1009MB  Free Space
 3      5016MB  7032MB  2015MB               dolumenti
        7032MB  21.5GB  14.4GB  Free Space
```

There is 1009MB of unallocated space after partition 2, so it can be increased within that range. Only the endpoint of a partition can be changed. In this case partition 2 can be increased by 1009MB and partition 3 by 14.4GB

Expand a partition to a new point

```bash
parted /dev/DISK resizepart 'PART_NUMBER' 'END_POINT'
```

EXAMPLE:

```bash
parted /dev/sdb resizepart 2 5015MB
```

Different filesystems have their own tools.
`ext4`, `XFS`, `btrfs` can be resized online.
FAT must be offline.

Filesystems resize commands:

`ext4`
```bash
resize2fs /dev/PARTITION
```

`xfs`
```
xfs_growfs -d /dev/PARTITION
```

`btrfs`
```bash
btrfs filesystem resize max /dev/PARTITION
```

#### Shrinking a Partition

Partitions should be offline

1. Run a filesystem health check
2. Resize the filesystem
3. Resize the partition

Filesystems check tools:

`ext4`
```
e2fsck -f /dev/PARTITION
```

`btrfs`
```
btrfs check /dev/PARTITION
```

`fat`
```
fsck.vfat -v /dev/PARTITION
```

Resize the filesystem to the desired and possible size

Shrink the partition to match the filesystem

```bash
parted /dev/DISK print -> get the partition number
parted /dev/DISK resizepart 'PART_NUMBER' 'END_POINT'
```

