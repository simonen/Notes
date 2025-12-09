---
tags:
  - centos
  - rhel
---
[Apress - Hands-on Booting] 

Boot comes from **bootstrap**. 
Bootloader: the code that boots the OS

Booting Sequence:

System starts up
BIOS performs POST
Looks for MBR
Reads MBR
Executes the first stage 
Executes the second stage and hands over the rest of the process to the kernel

#### Bootloader with MBR

The BIOS looks for the bootloader, which is located in the first 446 bytes of the MBR. On Linux it is a file boot.img. After that, it looks for and runs core.img. core.img looks for /boot/grub and loads the modules it finds there.

#### Bootloader with UEFI

The UEFI reads the GPT partition table and executes the bootloader code in /EFI. UEFI has its own partition that the bootloader and modules can be installed into. It is a FAT32 538MB partition.

LILO: legacy Linux bootloader, not in development since 2015
NTLDR - Microsoft's bootloader
Boot Camp: Mac OS bootloader

#### GRUB 2

- `/boot/grub2/grub.cfg`: Do not edit directly!
- `/etc/grub2.cfg` : Symbolic link of the above file
- `/etc/grub.d/` : Ordered configuration files
-  `/boot/grub2/i386-pc`: Various modules loaded at boot time via the `insmod` command - `xfs`, `gzio`, etc

The GRUB2 conf file is made by the `grub2-mkconfig` command. The command runs through the config files in `/etc/grub.d/`, executes them in order and creates a `grub.cfg` file

GRUB2 uses 4 items to boot a system
1. kernel file
2. name of the drive
3. partition number where the kernel resides
4. initial RAM disk (optionally)

Two ways of booting
1. directly booting by finding and loading the kernel 
2. chain-loading by loading another bootloader first, like MS which then loads Windows

`/boot/grub2/grub.cfg`
```
menuentry 'CentOS Linux (3.10.0-1160.114.2.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-1160.el7.x86_64-advanced-1aea9025-99c1-4dd1-bf6c-767fc8017c34' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1'  8180efc2-166a-4b67-87e0-389474835739
        else
          search --no-floppy --fs-uuid --set=root 8180efc2-166a-4b67-87e0-389474835739
        fi
        linux16 /vmlinuz-3.10.0-1160.114.2.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_US.UTF-8
        initrd16 /initramfs-3.10.0-1160.114.2.el7.x86_64.img
}
```

- `set root='hd0,msdos1'`: How root device is set
- `linux16`: Load Linux in 16-bit mode.
- `root=/dev/mapper/centos-root`: This kernel argument sets the root / path
- `rd.lvm.lv=centos/root` : tells the kernel to load the lvm volume 
- `rd.lvm.lv=centos/swap` : Tells the kernel to load the swap volume
- `rhgb` : Red Hat Graphical Boot
- `initrd`: Temporary root fs to first load necessary executables then loads the true / root fs

Info about kernel boot arguments:

`man 7 bootparam`

* `kernel`: interface between the user the and hardware. The Core 
* `initramfs`: contains a mini file system mounted at boot, provides kernel modules that are needed for the rest of the boot process (LVM, SCSI modules, etc)

The default GRUB conf file. Not directly edited! 

`/boot/grub2/grub.cfg`

The GRUB config file to edit

- `/etc/default/grub`
- `/etc/grub.d/*`

To make or apply changes to GRUB2 config file

``` bash
grub2-mkconfig -o /boot/"GRUB_DIR"/"grub2.cfg"
```

- `/boot/grub2/grub.cfg`: For BIOS based systems
- `/boot/efi/EFI/redhat/grub.cfg`: EFI based systems
#### Securing GRUB2

To protect GRUB2 edits with password

``` bash
grub2-setpassword
```

The commands creates a **/boot/grub2/user.cfg** file. After that, to update the grub.cfg and set the password

``` bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

Next time the system boots, the GRUB2 bootloader will ask for the password if an attempt to pass boot options is made.

To remove the password, remove the **/boot/grub2/user.cfg** file
#### Minimal GRUB Configuration

```
GRUB_DEFAULT=0
GRUB_TIMEOUT=10
GRUB_TIMEOUT_STYLE=menu
```

##### Reinstalling GRUB

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
grub-install /dev/sda
```

#### System Initialization

[[LINUX/SERVICES/Systemd Units]]

The first process after the kernel is loaded is PID1. Loads all other processes and their dependencies.

- `SysV init`
- Upstart (Ubuntu)
- `systemd`

