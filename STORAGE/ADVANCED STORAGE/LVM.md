
* logical volume resizing: if the filesystem supports it. XFS only supports increasing.
* **snapshots**: grows when the original volume has changed. Should be temporary. Remove when their purpose has been served.
* **failing hardware**: failing disks can be removed and replaced, dynamically, without downtime

#### Creating LVM Logical Volumes

Physical Volumes( From LVM partitions ) -> Volume Groups -> Logical Volumes

In fdisk or gdisk, createa GPT partition table on the device, create an LVM partition.

To create a physical volume:

``` bash
pvcrate /dev/"DEVICE"
```

To create the volume group from PV

``` bash
vgcreate "VGROUP" "PV"
```

To get info about a vg. Defaults to all groups:

``` bash
vgdisplay ["VGROUP"]
```

```
--- Volume group ---
  VG Name               vgdata
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1020.00 MiB
  PE Size               4.00 MiB
  Total PE              255
  Alloc PE / Size       0 / 0
  Free  PE / Size       255 / 1020.00 MiB
  VG UUID               XdhSt7-UikG-QIkX-I03W-FMqL-ZOph-YQ8Js3
```

**PE**: physical extent. 4MiB (up to 128) by default. A multiple of 2. PE is the size of the building block the LVM volume uses. A logical volume has a size that is a multiple of the PE size. For large volumes use big PE size.

When working with ext4, **logical extents** are used. Not related to **PE** of LVM

To get a short summary of volume groups:

``` bash
vgs
```

To create a volume group from an entire unpartitioned disk:

``` bash
vgcreate "VGROUP" "DISK"
```

To get info about a disk partition label

``` bash
partprobe -s
```

#### Creating Logical Volumes and Filesystems

**Volume Name** and **Size** must be specified

To create a lvm volume using absolute size:

``` bash
lvcreate -L "SIZE" "VGROUP"
```

To create a logical volume with a relative size (half of available space):

``` bash
lvcreate -l 50%FREE "VGROUP"
```

To create a logical volume using 25 **extents** x 4MiB = 100MiB .

``` bash
lv create -l 25 "VGROUP"
```

It is highly recommended to name the logical volume using -n option:

``` bash
lvcreate -n "LV" -L 100MiB "VGROUP"
```

#### LVM Device Naming

`dm`: device mapper - a generic interface to address storage devices. Used by LVM, RAID, multipath devices. DMs are created at boot time. `/dev/dm-0, /dev/dm-1`, etc
The device mapper uses symbolic links in the `/dev/mapper` directory to `/dev/dm-#` devices

Ways of addressing an LVM device:
  - `/dev/vgname/lvname`
  - `/dev/mapper/VGROUP-LVNAME`

#### Resizing Volume Groups

- `vgextend`: Add physical volumes to a volume group
- `vgreduce`: Remove physical volume(s) from a volume group

Multiple Linux LVM partitions can be added to a volume group

``` bash
vgextend "VGROUP" "/dev/sdx{1..n}"
```

To see which PVs are in a volume group:

``` bash
vgdisplay -v "VGROUP"
```

#### Resizing Logical Volumes

- `lvextend` - Add space to a logical volume
- `lvreduce` - Reduce the size of a logical volume

> [!NOTE]
> Resizing a filesystem on an LVM is limited by the capabilities of the filesystem itself as it uses the underlying filesystem tools. XFS can still only be extended. Also, LVM should not be resized smaller than the filesystems they hold.

Increase a LVM by half of VG free space. Similar to resizing volume groups

``` bash
lvresize -l -r +50%FREE "/dev/vg_group/lvm"
```

`-r, --resizefs`: Resize underlying filesystem together with the LV using fsadm. Recommended

To manually resize an XFS filesystem

``` bash
xfs_growfs "LVM"
```
#### Reducing Volume Groups

To remove a PV from a Volume Group, its used PE must first be moved to another PV that can accommodate them.

``` bash
pvmove -v /dev/"PV_TO_REMOVE" /dev/"PV_WITH_AVAILABLE_SPACE"
```

When the PV-to-be-removed has 100% of its PE available, it can be removed from the VG:

``` bash
vgreduce "VOLUME_GROUP" /dev/"PV_TO_REMOVE"
```