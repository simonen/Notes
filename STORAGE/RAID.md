---
tags:
  - centos
---
[Apress – Pro Linux System Administration 2nd 2017.pdf](file:///E:/_DEVOPS/APRESS/_LINUX/Apress%20%E2%80%93%20Pro%20Linux%20System%20Administration%202nd%202017.pdf)

Redundant Array of Inexpensive Disks

Types
* hardware: stand-alone raid controller
* [fake](https://raid.wiki.kernel.org/index.php/DDF_Fake_RAID): raid chip embedded in the motherboard.
* software: md (multiple disks). md arrays are transferrable between hosts

RAID Levels

| RAID Level |          | Functionality                  | Storage Capacity |
| ---------- | -------- | ------------------------------ | ---------------- |
| RAID 0     | striping | Speed                          | N * size         |
| RAID 1     | mirror   | Redundancy                     | N * size / 2     |
| RAID 5     | parity   | Redundancy, speed              | N - 1 * size     |
| RAID 6     |          | Redundancy, reliability, speed | N - 1 * size     |
| RAID 10    |          | Redundancy, reliability, speed | N / 2 * size     |
| RAID 50    |          | Redundancy, speed              | N – 1 * size     |

spare disk

striping: data is split between disks arbitrarily in parts. Very fast. One disk fails, info is lost
mirror: data is cloned among disks
parity: data is striped across multiple disks and a checksum of that stripe is written to a different disk (block parity). Slow. Requires processing power to calculate parity

#### Creating Software RAID 

> Oreilly - Managing RAID on Linux
> http://linux-raid.osdl.org/index.php/Linux_Raid
> Apress – Pro Linux System Administration 2nd 2017.pdf

Package 
`mdadm`

man mdadm.conf
man mdadm

Configuration file:
- `/usr/lib/tmpfiles.d/mdadm.conf`

`mdadm`: Multiple Device Administration Tool

Create a RAID 1 with 2 disks

``` bash
mdadm --create /dev/md0 --level=raid1 --raid-devices=2 'DEV1' 'DEV2'
```

Create a RAID 5 array with 3 (minimum) active disks and 1 spare

``` bash
mdadm --create /dev/md0 --level=raid5 --raid-devices=3 --spare-devices=1 /dev/sdc /dev/sdd /dev/sde /dev/sdf
```

To disassemble an array

``` bash
mdadm manage 'RAID_ARRAY' --stop
```

##### Querying and Inspecting RAID Arrays

To list RAID arrays

```bash
mdadm --examine --scan
```

Check on the status of the RAID

``` bash
mdadm --query --detail /dev/md0
```

> [!NOTE]-
> ```
> mdadm: added /dev/sdb
> /dev/md0:
>            Version : 1.2
>      Creation Time : Thu May  9 11:06:53 2024
>         Raid Level : raid1
>         Array Size : 1098944 (1073.19 MiB 1125.32 MB)
>      Used Dev Size : 1098944 (1073.19 MiB 1125.32 MB)
>       Raid Devices : 2
>      Total Devices : 4
>        Persistence : Superblock is persistent
> 
> Update Time : Thu May  9 21:00:31 2024
> State : clean
> Active Devices : 2
> Working Devices : 4
> Failed Devices : 0
> Spare Devices : 2
> 
> Consistency Policy : resync
> 
> Name : prometheus.olympus.local:0  (local to host prometheus.olympus.local)
> UUID : 90e44cb3:8a9c60ac:ee86f3c4:ce24681e
> Events : 78
> 
> Number   Major   Minor   RaidDevice State
> 4       8       64        0      active sync   /dev/sde
> 1       8       32        1      active sync   /dev/sdc
> 2       8       16        -      spare   /dev/sdb
> 3       8       48        -      spare   /dev/sdd
> ```
> 

``` bash
cat /proc/md0
```

```
Personalities : [raid1]
md0 : active raid1 sdb[2](S) sde[4] sdd[3](S) sdc[1]
      1098944 blocks super 1.2 [2/2] [UU]

unused devices: <none>
```

- `[UU]`: both devices are UP. 
- `[_U]`: one device is degraded
- `(S)`: spare

##### Removing RAID Arrays

Remove configuration in `mdadm.conf` associated with the array

Stop the array

``` bash
mdadm --manage /dev/"ARRAY" --stop
```

Remove RAID metadata from the disks that were part of the array. 

``` bash
mdadm --zero-superblock /dev/sd"[XYZ]"
```

##### Expanding Arrays

1. Add appropriate devices to the array
2. Grow the array

To add spares

``` bash
mdadm --manage /dev/"ARRAY" --add "DEV1" "DEVX"
```

The process of rebuilding a RAID 5 array is very slow and destructive. If a failure occurs during hash recalculation, data will be lost. Make a backup first. 

To grow the array. Mdadm will pick available devices from the spares.

``` bash
mdadm --grow /dev/md0 --raid-disks="NEW_TOTAL_NUMBER" --backup-file="RAID_BACKUP_FILE"
```

####  Creating LVM on RAID

[[gitea/LINUX 1/STORAGE/ADVANCED STORAGE/LVM]] [[gitea/LINUX 1/STORAGE/Filesystems]]

Create the physical volume out of the RAID array

``` bash
pvcreate /dev/"ARRAY"
```

Create the volume group

``` bash
vgcreate raid-volume 'PV'
```

Create the logical volumes

```
lvcreate --name www --size 2G raid-volume
```

#### Disk Failures

Current config

```
Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       4       8       48        2      active sync   /dev/sdd
       5       8       80        3      active sync   /dev/sdf

       3       8       64        -      spare   /dev/sde
```

To simulate the event of a disk failure 

``` bash
mdadm --manage /dev/md0 --fail /dev/sdd
```

```
[34116.596391] md/raid:md0: Disk failure on sdd, disabling device.
md/raid:md0: Operation continuing on 3 devices.
[34116.641127] md: recovery of RAID array md0
[34140.131754] md: md0: recovery done.
```

The system picked up on the failure and replaced the failed disk with an available spare

```
Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       3       8       64        2      active sync   /dev/sde
       5       8       80        3      active sync   /dev/sdf
       4       8       48        -      faulty   /dev/sdd
```

Remove the failed disk from the array

``` bash
mdadm --manage /dev/"ARRAY" --remove "/dev/FAILED_DISK"
```

And add a new spare with the `--add` command

If the array rebuilding does not happen automatically for some reason, it can be forced manually with the `assembe` mode. To recreate a RAID array from existing components

``` bash
mdadm --assemble /dev/"ARRAY" "DEV1" "DEVX"
```
