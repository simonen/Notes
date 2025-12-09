https://wiki.manjaro.org/index.php?title=Some_basics_of_MBR_v/s_GPT_and_BIOS_v/s_UEFI
#### MBR Master Boot Record

* The first 512 bytes of the disk
* 446 bytes for the bootloader
* 64 byte partition table. A partition requires 16 bytes to be described
* 2 byes for boot signature
* 32-bit addressing
* 2TiB max partition size: 2^32 (number of blocks) x 512-byte blocks = 2.2TiB
* 4 partitions: **primary**, **logical** and **extended**.
* Total of 15 partitions when using the extended partitions to create multiple logical partitions
* Works with BIOS

One of the primary partitions can be marked as **extended**, which can hold many logical partitions. If the extended part goes down, so do logical partitions.
The fourth partition is the last. Makes sense to fill up all remaining space.
#### GPT GUID Partition Table

* Max partition size: 8 ZiB
* 64-bit addressing
* Max partitions: 128
* No need to distinguish between primary, logical and extended partitions
* Backup copy of the partition table is created at the end of the disk
* Works with **BIOS** and **UEFI**

#### Blocks and Sectors

**Blocks**: the smallest logical unit on a disk that a filesystem can use. Can span multiple sectors. Store additional information like timestamps, permissions, filenames, ownership, block ID, inode, order in relation to other blocks, metadata. Blocks = 2^32 for MBR and 2^64 for GPT partitions

**Sector**: the smallest physical unit on a disk. Standard sector sizes are 512 and 4MB for larger disks. 

#### Storage Measurement Units

- `MB` - Megabyte - multiple of 1000
* `MiB` - Mebibyte - multiple of 1024

Linux works with MiBs
#### Managing Partitions and Filesystems

* `fdisk`: Manage MBR and GPT partitions
- `gdisk`: Manage GPT partitions
- `parted`: Utility for basic partition management

Device naming:
- `/dev/sda`: Uses the **iSCSI** driver. Used for **SCSI** and **SATA** devices
- `/dev/nvmen0n1`:  The first nvme drive on a nvme interface
- `/dev/hda`: Legacy disks on the IDE interface
- `/dev/vda`: A disk in a KVM virtual machine
`/dev/xvda`: A disk in a Xen virtual machines, using the Xen virtual disk driver

#### Creating MBR Partitioning

Find the disk device that needs to be partitioned

```bash
lsblk -fs
```

Create the partition with **fdisk**

```bash
fdisk /dev/sdb
```

If `fdisk` prints a message stating that it could not update the partition table:

To manually update the partition

```bash
partprobe
```

#### Creating GPT Partitioning

# Never use **gdisk** on a **fdisk** partitioned device!!!!
WHY?



