
**man lshw**

- `lspci`
- `lsusb`
- `lscpu`
#### Filtering `lshw` Output

`lshw` options

`lshw -short` produces

```
H/W path              Device      Class       Description
=========================================================
                                  system      VirtualBox
/0                                bus         VirtualBox
/0/0                              memory      128KiB BIOS
/0/1                              memory      8GiB System memory
/0/2                              processor   Intel(R) Xeon(R) CPU E5-2680 v4 @ 2.40GHz
/0/100                            bridge      440FX - 82441FX PMC [Natoma]
/0/100/1                          bridge      82371SB PIIX3 ISA [Natoma/Triton II]
/0/100/1/0                        input       PnP device PNP0303
/0/100/1/1                        input       PnP device PNP0f03
/0/100/1.1            scsi0       storage     82371AB/EB/MB PIIX4 IDE
/0/100/1.1/0.0.0      /dev/cdrom  disk        CD-ROM
```

To filter out specific items from the Class column

```bash
lshw [-short] -class *CLASS_ITEM*
```

- `-short`: Short version
- `-sanitize`: Removes sensitive info such as IP addresses, serial numbers

Format the report:
-json
-html
-xml 

#### Detecting Hardware with `hwinfo`

**man hwinfo**

`hwinfo --help`: For a list of hardware items

List RAID devices, if any

```bash
hwinfo --listmd
```

Get detailed info about selected components

```bash
hwinfo --'COMPONENT1' --'COMPONENT2' --'COMPONENTN'
```

#### Detecting PCI Hardware with `lspci`

Read information from the PCI bus, includes onboard components, items plugged in the PCI slots. Reads info from its own databases `pci.ids`

To update the database

```bash
update-pciids
```

Locate the database

```
locate pci.ids
```

```
/usr/share/misc/pci.ids : Debian
/usr/share/hwdata/pci.ids : RHEL, CentOS
```

```bash
lspci
lspci -v, vv, vvv
```

#### Understanding lspci Output

```
[root@server2 ~]# lspci
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
00:01.1 IDE interface: Intel Corporation 82371AB/EB/MB PIIX4 IDE (rev 01)
00:02.0 VGA compatible controller: VMware SVGA II Adapter
00:0d.0 SATA controller: Intel Corporation 82801HM/HEM (ICH8M/ICH8M-E)
```

00:0d.0: **BDF** number. bus:device:function. 

Display tree view to see relationship between the PCI bus and devices

```bash
lspci -tvv
```

```
[root@server2 ~]# lspci -tvv
-[0000:00]-+-00.0  Intel Corporation 440FX - 82441FX PMC [Natoma]
           +-01.0  Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
           +-01.1  Intel Corporation 82371AB/EB/MB PIIX4 IDE
           +-02.0  VMware SVGA II Adapter
           +-03.0  Intel Corporation 82540EM Gigabit Ethernet Controller
```

PC's almost always have a single PCI bus, so the number is 00

\[0000:00]: domain(host bridge):bus. Domain is a Linux term. Aka **segment group**

**host bridge**: connects the PCI controller to the CPU
Servers with multiple CPUs have multiple host bridges, and sometimes - multiple buses on a single domain

Display the domain

```bash
lspci -D
```

#### Filtering lspci Output

Using **awk**

To filter out entries containing a class keyword \[ Audio, CPU, USB, Ethernet... as per the `lspci` standard output ]

```bash
lspci -v | awk '/KEYWORD/,/^$/'
```

\/^$/: looks for the line breaks. Great for extracting text blocks from the -v output
```
text block 1
	text text text

text block 2
	text2 text2 text2
```

```
lspci -nn
```

```
[root@server2 ~]# lspci -nn
00:00.0 Host bridge [0600]: Intel Corporation 440FX - 82441FX PMC [Natoma] [8086:1237] (rev 02)
00:01.0 ISA bridge [0601]: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II] [8086:7000]
00:01.1 IDE interface [0101]: Intel Corporation 82371AB/EB/MB PIIX4 IDE [8086:7111] (rev 01)
00:02.0 VGA compatible controller [0300]: VMware SVGA II Adapter [15ad:0405]
```

- \[0600]: class number
- \[8086:1237]: vendor_number : device_number

To filter out by class number, vendor, device number

``` bash
lspci -d ::"NUMBER"
```

#### Using lspci to Identify Kernel Modules

To find out which kernel module a device is using

```bash
lspci -kd ::'CLASS_NUMBER'
```

To filter out the Ethernet only in machine-readable output -mm

```bash
lspci -vmmk | awk '/Ethernet/,/^$/'
```

```
Class:  Ethernet controller
Vendor: Intel Corporation
Device: 82540EM Gigabit Ethernet Controller
SVendor:        Intel Corporation
SDevice:        PRO/1000 MT Desktop Adapter
Rev:    02
Driver: e1000
Module: e1000
```

#### Using lsusb to List USB Devices

```bash
lsusb -tv
```

```
[root@server2 ~]# lsusb -tv
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/6p, 5000M
    ID 1d6b:0003 Linux Foundation 3.0 root hub
    |__ Port 1: Dev 2, If 0, Class=Mass Storage, Driver=usb-storage, 5000M
        ID 0781:5580 SanDisk Corp. SDCZ80 Flash Drive
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/8p, 480M
    ID 1d6b:0002 Linux Foundation 2.0 root hub
```

The dev number changes every time a device is plugged

ID 1d6b:0003: vendor_code:device_code

#### Listing Partitions and Hard Disks with lsblk

[[LINUX/STORAGE/Devices]] [[LINUX/STORAGE/Filesystems]]

On Linux, mass storage devices like SATA and flash media use the SCSI driver

```bash
lsblk
```

```
[root@server2 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda             8:0    0   20G  0 disk
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0   19G  0 part
  ├─rhel-root 253:0    0   17G  0 lvm  /
  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0   20G  0 disk
├─sdb1          8:17   0  1.9G  0 part
├─sdb2          8:18   0  2.8G  0 part
sdc             8:32   1 29.2G  0 disk
└─sdc1          8:33   1 29.2G  0 part
sr0            11:0    1  9.8G  0 rom  /repo
```

**RO**: real-only
**RM**: removable
**MAJ:MIN**: major:minor numbers. Major identifies category, 8 is for sd devices, minor labels each device in sequence **/sys/dev/block**
Full list of major numbers for block and character devices
https://www.kernel.org/doc/Documentation/admin-guide/devices.txt

To determine the corresponding device in `/dev`

``` bash
ls -l /dev/block/"MAJ:MIN"
```

`lrwxrwxrwx. 1 root root 7 May  9 23:20 /dev/block/253:2 -> ../dm-2`

To get info about the device

``` bash
lsblk /dev/dm-2
```

```
NAME             MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
raid--volume-www 253:2    0   2G  0 lvm
```

Get extended info with udevadm

``` bash
udevadm info --query=all --name=/dev/dm-2
```

To display SCSI devices

``` bash
lsblk -S
```

```
NAME HCTL       TYPE VENDOR   MODEL          REV SERIAL               TRAN
sda  2:0:0:0    disk ATA      VBOX HARDDISK 1.0  VB0f2b4859-1c3cccab  sata
sdb  3:0:0:0    disk ATA      VBOX HARDDISK 1.0  VB8b8d3bc6-90291afb  sata
sdc  4:0:0:0    disk SanDisk  Extreme       0001 AA010409151407392732 usb
sr0  0:0:0:0    rom  VBOX     VBOX CD-ROM   1.0  VB0-01f003f6         ata
```

#### Identifying Hardware Architecture

```bash
uname -m
arch
```
