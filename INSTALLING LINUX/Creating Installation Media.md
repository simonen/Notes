---
tags:
  - centos
---
#### Create a USB Bootable Media

`dd`: Disk duplicator. Part of the `coreutils` package

``` bash
dd status=progress if='INSTALLATION.ISO' of=/dev/'USB_DRIVE'
```

Verify the file checksum.

``` bash
echo "CHECKSUM FILE" | sha256sum -c
```

```
Win10_22H2_English_x64v1.iso: OK
```

#### Red Hat Family Bootable ISO Structure

Package
`genisoimage`

The contents of a bootable Rocky9 Linux

```
[root@ldap roki]# tree /mnt/rocky/ -L 1 -a
/mnt/rocky/
├── BaseOS
├── .discinfo
├── EFI
├── images
├── isolinux
├── LICENSE
├── media.repo
├── minimal
└── .treeinfo
```

BIOS: `isolinux.bin` and the boot catalog are essentials components of a bootable image.

**ISOLINUX.BIN**

`isolinux.bin`:  A binary file that comes from the ISOLINUX bootloader, part of the SYSLINUX project, which specializes in creating bootloaders for Linux-based systems.
* Purpose: The `isolinux.bin` is the file that directly interfaces with the system's firmware. It provides the instructions and bootable code to start the OS or the installation program
* During the boot process, the BIOS reads `isolinux.bin`, which then reads a configuration file (e.g., `isolinux.cfg` ) to present a boot menu, load the Linux kernel, and initiate the boot process. It can also handle boot options, kernel parameters, and multiboot setups.

**ISOLINUX.cfg**

The `isolinux.cfg` defines the boot menus and bootparams. 

Example of a minimal `isolinux.cfg` file defining just the boot parameters

`/isolinux/isolinux.cfg`
```
default linux
label linux
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=Rocky9-KS-ISO inst.ks=cdrom:LABEL=Rocky9-KS-ISO:/isolinux/ks.cfg
```

**Boot Catalog**

The boot catalog is required by the **El Torito** bootable CD/DVD specification which describes how optical disks can be made bootable. Contains metadata about the bootable sectors and servers as an index for the booting system. The boot catalog is referenced by the `isolinux.bin` to ensure the correct boot information and files are loaded.

**.treeinfo**

The `.treeinfo` file provides information about the layout and structure of the installation or repository tree, which can be crucial for tools like Anaconda (the Red Hat installation program) to find and use the correct resources during installation. It is typically placed in the root directory of the installation medium or repository.

#### Create a Bootable ISO image

Copy the contents of the original iso file, including the hidden files, to a new location. CD into the new location.

Generate the iso file using `genisoimage` or `mkisofs`. Paths must be relative! 

``` bash
genisoimage -o 'FILE.iso' \
-b isolinux/isolinux.bin \
-c isolinux/'BOOT CATALOG FILE' \
-no-emul-boot \
-boot-load-size 4 \
-boot-info-table \
-J -R -V "CD LABEL" "WORKING DIR"
```

`-b`: Specifies the boot image
`-c`: Creates a boot catalog file (if none)
`-J`: Joilet extensions
`-R`: Rock Ridge extensions
`-V`: Volume label

To check if a .iso file is bootable

``` bash
isoinfo -d -i ".ISO"
```

```
[root@ldap roki]# isoinfo -d -i /home/kimchen/Rocky9-text.iso
CD-ROM is in ISO 9660 format
System id: LINUX
Volume id: Rocky9-KS-ISO
...
El Torito VD version 1 found, boot catalog is in sector 136
...
Eltorito validation header:
    Hid 1
	...
    Eltorito defaultboot header:
        Bootid 88 (bootable)
        Boot media 0 (No Emulation Boot)
        Load segment 0
        Sys type 0
        Nsect 4
        Bootoff 89 137
```

##### Partitioning

It is a good idea to put the following in separate partitions

**/boot**: on a separate partition is a good option for multiboot systems. 500MB is enough
**/**:  putting the /, root, in a separate partition makes it easier to restore or nuke, replace with new distro. 60GB needed if BTRFS is used to store snapshots. Else, 30G is ok
**/home**: can be saved in case the distro is changed
**/var**: 20G 
**/tmp**: 20G
**swap**: equal to RAM

##### Multiple Installations

Make sure to have the /boot mount on separate partitions

##### Dual-Booting With Windows

Install Windows first, Linux afterwards so that the Linux bootloader takes control

#### Recover an OEM Windows Key

In Linux

```
cat /sys/firmware/acpi/tables/MSDM
```

In Windows 

`PS: Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SoftwareProtectionPlatform\' -Name BackupProductKeyDefault`

##### Mounting ISO images

**/dev/loop**: pseudodevice to make image files accessible like any other filesystem
\# **mount -o loop** *IMAGE*.iso /*DIR*

