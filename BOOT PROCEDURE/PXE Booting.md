---
tags:
  - centos
  - dhcp
  - apache
---
[CentOS Network Boot](https://docs.centos.org/en-US/centos/install-guide/pxe-server/)
PXE: Pre-Executed Environment

PXE boots rely on a DHCP server to assign IP address to the client machine and point to the necessary boot files, and a TFTP server to make those files available. The installation source is served by HTTP or NFS server.

> The PXE client must have sufficient memory >= 4G

The PXE boot server is made of these stages:
- **stage0:** the PXE-booting client sent an extended DHCPDISCOVER
- **stage1:** the DHCP server send an IP address to the client and supplementing information such as the TFTP server
- **stage2:** the PXE-booting client configures the network and requests the first boot files from the TFTP server (boot loader, kernel image and sysresccd.img)
- **stage3:** the PXE-booted client requests the `squashfs` root file system image from the HTTP or NFS or NBD server

Package
`syslinux`

SYSLINUX is a suite of bootloaders
#### Configure TFTP

The TFTP protocol is extensively used to support remote booting of diskless devices.  The server is normally started by `inetd`, but can also run standalone.

Package
`tftp-server`

Default TFTP root directory
`/var/lib/tftpboot`

Directory tree

`/var/lib/tftpboot`
```
├── grub # EFI
│   ├── grub.cfg
│   ├── grubx64.efi
│   └── shimx64.efi
├── images
│   ├── initrd.img
│   └── vmlinuz
├── ldlinux.c32
├── libcom32.c32
├── libutil.c32
├── pxelinux.0 # BIOS
├── pxelinux.cfg
│   └── default
└── vesamenu.c32
```

`/var/www/OS_INSTALLATION_SOURCE`
```
└── Rocky
    ├── BaseOS
    ├── .discinfo
    ├── EFI
    ├── images
    ├── isolinux
    ├── ks.cfg
    ├── LICENSE
    ├── media.repo
    ├── minimal
    └── .treeinfo
```

Boot menu files. Found in the ISO

BIOS:
- `pxelinux.0`: BIOS bootloader. From the `syslinux` package
- `pxelinux.cfg/default`: BIOS menu configuration file. From `isolinux/isolinux.cfg`
- `.32`: BIOS menu libraries. From `isolinux/`
- `vmlinuz and initrd`: From `images/pxeboot` or `isolinux/`
EFI:
- `shimx64.efi`: EFI bootloader. From the `Packages/shim-x64_xxx.rpm`
- `grub.cfg`: EFI boot menu configuration file. From `/EFI/BOOT`
- `grubx64.efi`: From `/EFI/BOOT` 

`.treeinfo`: This file defines the installation source directory structure. Pulled during stage2

Extract the `shimx64.efi` file from the shim-x64 archive

``` bash
rpm2cpio minimal/Packages/shim-x64.xxx.rpm | cpio -dimv
```

Enable and start the TFTP

```bash
systemctl enable --now tftp.socket
```

Allow TFTP through the firewall

```
firewall-cmd --add-service=tftp --permanent ; firewall-cmd --reload
```
#### Troubleshooting

Listening on the TFTP can give hints on what files and from where the PXE client is trying to pull from the server

``` bash
tcpdump port tftp
```

PXE client is requesting file grub/shimx64.efi

```
IP 192.168.137.225.r > pxeclient.tftp: TFTP, length 59, RRQ "grub/shimx64.efi"
```

The TFTP server says the PXE client successfully downloaded the PXE boot files.

```
ldap.ohio.cc in.tftpd: Client :192.168.137.121 finished pxelinux.0
ldap.ohio.cc in.tftpd: Client :192.168.137.121 finished pxelinux.cfg/default
ldap.ohio.cc in.tftpd: Client :192.168.137.121 finished menu.c32
ldap.ohio.cc in.tftpd: Client :192.168.137.121 finished /images/pxeboot/vmlinuz
ldap.ohio.cc in.tftpd: Client :192.168.137.121 finished /images/pxeboot/initrd.img
```
#### Configure DHCP

[[LINUX/NETWORK SERVICES/DHCP]] 
[DHCP and PXE Boot](https://datatracker.ietf.org/doc/html/rfc5071)

`PXE (Pre-boot execution Environment)` is a first-stage network bootstrap agent. PXE is loaded out of firmware on the client host, and performs DHCP queries to obtain an IP address

`PXELINUX` is a second-stage bootstrap agent. `PXELINUX` seeks its configuration from a cache of DHCP options supplied to the PXE first-stage agent, and then takes action based upon those options.`

`/etc/dhcp/dhcpd.conf`
```
allow booting; # Tell the DHCP server to respond to queries from booting clients
allow bootp; # Tell the DHCP server to respond to queries from booting clients
option architecture-type code 93 = unsigned integer 16;

subnet 192.168.137.0 netmask 255.255.255.0 {
    option routers             192.168.137.1;
    option subnet-mask         255.255.255.0;
    range dynamic-bootp        192.168.137.220 192.168.137.254;
    default-lease-time         21600;
    max-lease-time             43200;
    next-server                192.168.137.103;
    class "pxeclients" {
        match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
		if option architecture-type = 00:07 {
	        filename "grub/shimx64.efi";
	    } 
	    else {
	        filename "pxelinux.0";
	    }
    }
}
```

- `filename "pxelinux.0";`: (BIOS) The name boot file that PXE-booted hosts should look for to start the boot process. 
- `filename "grub/shimx64.efi"`: (UEFI)
- `next-server IP`: The address of the file server (TFTP) that makes the files for PXE boot available.
- `option vendor-class-identifier, 0, 9`: The DHCP server will look in the first 9 characters for the vendor class... 

Architecture types:
- `00:01`: x86
- `00:07`: x84_64
- `00:0E`: ARM

The Client System Architecture type can be pulled from the DHCP discover packet
`Option: (93): Client System Architecture ` in Wireshark or by listening to DHCP traffic with `tcpdump`.

``` bash
tcpdump -i enp0s18 -n udp port 67 -v | grep Vendor-Class
```

```
Vendor-Class (60), length 32: "PXEClient:Arch:00007:UNDI:002001"
```

##### Paths 

The TFTP server's default root directory is `/var/lib/tftpboot`, and paths in the DHCP server configuration are relative to that root directory.

The `filename` option sets the path prefix for the configuration, kernel and initrd files.
Example: `filename "pxelinux.0"` will make the bootloader look for the config file `pxelinux.cfg/default` in the root tftp directory. 

#### HTTP 

[[LINUX/APACHE HTTP SERVICES/Basic Apache Web Server]]

Package
`httpd`

`/etc/httpd/conf.d/os.conf`
```
<VirtualHost 192.168.137.11:80>
    ServerName 192.168.137.11
    DocumentRoot /var/www/cobbler

    # Alias the directory to /os
    Alias /os /var/www/os

    # Allow all access to /var/www/os and all subdirectories
    <Directory /var/www/os/>
        Require all granted
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>

    # Optional logging
    ErrorLog /var/log/httpd/os_error.log
    CustomLog /var/log/httpd/os_access.log combined
</VirtualHost>
```

#### Booting From the Network

The kernel boot parameters must be edited in the respective configuration files to point to the correct location of the kernel image and the installation directory tree (stage2)

`pxelinux.cfg/default`
```
kernel /images/vmlinuz
append initrd=/images/initrd.img inst.stage2=http://server/Rocky9 [inst.ks=...]
```

`grub/grub.cfg`
```
linuxefi /images/vmlinuz inst.repo=http://server/Rocky9
        initrdefi /images/initrd.img [inst.ks=...]
```

The installer will read the `.treeinfo` file from the installation root directory tree which defines its structure.