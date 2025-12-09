[Apress – Pro Linux System Administration 2nd 2017.pdf](file:///E:/_DEVOPS/APRESS/_LINUX/Apress%20%E2%80%93%20Pro%20Linux%20System%20Administration%202nd%202017.pdf)

The /dev/ directory is populated by the **udev** service. Device files are the way the kernel provides access to devices for apps and services

Device files types:
* block devices: hard disks, USB drives, CD/DVD, tape drives
* dm-X: device mapper

```
[root@prometheus ~]# ls -l /dev/ | grep ^b
brw-rw----. 1 root disk    253,   0 May  8 10:29 dm-0
brw-rw----. 1 root disk    253,   1 May  8 10:29 dm-1
brw-rw----. 1 root disk      7,   0 May  8 10:29 loop0
brw-rw----. 1 root disk      8,   0 May  8 10:29 sda
brw-rw----. 1 root disk      8,   1 May  8 10:29 sda1
brw-rw----. 1 root disk      8,   2 May  8 10:29 sda2
brw-rw----. 1 root cdrom    11,   0 May  8 10:29 sr0
```

sd: SCSI (Small Computer System Interface) device
a, b, c, etc: first detected device letter
1, 2, 3, etc: partition number of the device node

The printed block devices are readable and writable by the root and the disk group only.

b: block device
disk 8, 0:  device major and minor numbers
* major number: identifies device type
* minor number: identifies device specific device

Use the dmesg command to see if a disk (or other devices) was detected by the system

``` bash
dmesg | grep sdX
```

```
[ 1.839834] sd 2:0:0:0: Attached scsi generic sg1 type 0
[ 1.840183] sd 2:0:0:0: [sda] Write cache: enabled, read cache: enabled
[ 1.842304] sda: sda1 sda2 < sda5 >
[ 1.842784] sd 2:0:0:0: [sda] Attached SCSI disk
```

\<sda5> indicates logical partition

To list SCSI devices with lsblk

``` bash
lsblk -S
```

``` bash
NAME HCTL       TYPE VENDOR   MODEL             REV TRAN
sda  0:0:0:0    disk ATA      VBOX HARDDISK    1.0  sata
sdb  1:0:0:0    disk ATA      VBOX HARDDISK    1.0  sata
sdc  2:0:0:0    disk ATA      VBOX HARDDISK    1.0  sata
sdd  3:0:0:0    disk ATA      VBOX HARDDISK    1.0  sata
sde  4:0:0:0    disk ATA      VBOX HARDDISK    1.0  sata
sr0  6:0:0:0    rom  VBOX     CD-ROM           1.0  ata
```

` chatgpt`

In the context of SCSI (Small Computer System Interface) devices, "HCTL" stands for "Host, Channel, Target, and LUN" and represents the address of the SCSI device within the system. Each part of the address corresponds to a specific aspect of the SCSI bus:

* **Host**: Refers to the SCSI host adapter or controller through which the device is accessed. Multiple host adapters can exist in a system.
    
- **Channel**: Represents the SCSI bus number, which allows multiple SCSI buses to be present in a system.
    
- **Target**: Identifies the SCSI device connected to the channel. Each target typically represents a different device on the SCSI chain.
    
- **LUN (Logical Unit Number)**: Further refines the target specification and allows multiple logical units (such as partitions or logical drives) to exist under a single target.
    

`0:0:0:0` is the HCTL address for the SCSI device `sda`, where:

- Host: 0
- Channel: 0
- Target: 0
- LUN: 0

This address uniquely identifies the SCSI device within the system and is often used for management and configuration purposes.

In modern Linux systems, device management is typically handled by utilities such as `udev` or `sysfs`. 

 To disable a SCSI device. Make sure the right HCTL is selected with `lsblk` first. Not permanent

 ``` bash
 echo 1 > /sys/class/scsi_device/"HCTL"/device/delete
 ```

``` #dmesg
[ 4192.015119] ata7.00: disabled
```

If the physical device is not removed, it will reappear in the next boot.

`chatgpt`

To re-enable a disabled device, we run a scan on the appropriate host adapter

``` bash
echo "- - -" > /sys/class/scsi_host/host1/scan
```

```
[ 9252.983608] scsi 1:0:0:0: Direct-Access     ATA      VBOX HARDDISK    1
[ 9252.993197] sd 1:0:0:0: [sdb] ... blocks: (1.13 GB/1.05 GiB)
[ 9252.993352] sd 1:0:0:0: [sdb] Write Protect is off
[ 9252.993365] sd 1:0:0:0: [sdb] Mode Sense: 00 3a 00 00
[ 9252.993437] sd 1:0:0:0: [sdb] Write cache: enabled, read cache
[ 9252.998663] sd 1:0:0:0: Attached scsi generic sg1 type 0
[ 9253.007161] sd 1:0:0:0: [sdb] Attached SCSI disk
```

#### udevadm 

