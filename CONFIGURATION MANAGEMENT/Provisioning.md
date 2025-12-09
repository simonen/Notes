---
tags:
  - centos
  - dhcp
---
Automated provisioning, also called bootstrapping, kickstarting (CentOS), pre-seeding (Debian)

Provisioning: Automatically installing a distro to a host.

1. The host machine performs a network query to a DHCP server
2. The DHCP points to the PXE server, which offers the files needed to boot. 
3. Provisioning process continues by installing a prepackaged version of the distro with a series of automated scripted responses to various configuration changes prompted during installation

`PXE Preboot Execution Environment`
#### Kickstart

[Kickstart Documentation](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html)

Kickstart files work with Red Hat Linux offshoots to automate OS installations.

Red Hat derivatives create a kickstart `anaconda-ks.cfg` file in the root home directory that contains the installation configuration that was used when installing the host.

Required Kickstart directives. These directives mirror common installation questions. 

* `auth(select)`
* `bootloader`
* `part`
* `keyboard`
* `lang`
* `timezone`
* `rootpw`
* `user`
* `packages`

If any of the required directives is not specified, the user will be prompted during installation. 

A good practice is to follow the section order
1. Commands
2. %packages
3. %pre, %pre-install, %post, %onerror, and %traceback sections

Example minimal kickstart file using CD as installation source. Options starting with % must be closed with %end. 

`ks.cfg`
```
#version=RHEL9
# System authorization information
authselect select minimal

# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
autopart --type=lvm

# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8
# System timezone
timezone Europe/Sofia --utc

# Root password
rootpw --iscrypted $6$UFb1LqH34lUYJ3LR$84PbVKUzqaxsptDwCiCsI9K9E8zUA8nU2ydILmGFF9AppK7NPVmaJu5swoXWj3c/JnAAf1fUXmJR7Zqx27nGF/

# Create an additional user
user --name=kimchen --password=$6$mgdSmS6YLCYKqarV$zR99Dki2m3iYjycu4nlUjYYkLHKLfwWT8e1Yoh4v9bu38oWxGGBA22xbhtRgFiuOTmaXEbXX4s4auZbnWiLoL/ --iscrypted --gecos="Kim Chen UN"

%packages
@^minimal-environment
%end

# Eject the ISO and reboot the system
reboot --eject
```

Generate a password hash

``` bash
python -c ‘import crypt; print(crypt.crypt(“My Password”, “$6$MySalt”))’
```

[Package Selection](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html#chapter-9-package-selection) Core group is selected by default.

```
%packages
@^Fedora Server Edition
@GNOME Desktop Environment
@Graphical Internet
@Sound and Video
dhcp
%end
```

`@^`: Environment (Minimal, Standard, Infrastructure Server, etc.. as specified by the distro). Refer to `comps.xml` for supported environments
`@`: Package group
`dhcp`: individual package to install
`-dhcp`: package not to install
`maria*`: Everything that starts with `maria`

Additional kickstart options

```
### PARTITION OPTIONS
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel

# Do not configure the X Window System
skipx

# Use text mode install
text
```

Execute the newly installed Linux kernel from the current environment

```
reboot --kexec
```

[Network options](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html#id42)

```
network --device=DEVICE --bootproto=static --ip=10.0.2.15 --netmask=255.255.255.0 --gateway=10.0.2.254 --nameserver=10.0.2.1 --hostname=vm1.example.com 
--onboot yes --activate 
```

[Firewall Options](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html#firewall) and [SELinux](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html#id54)

```
firewall --enable --use-system-defaults
```

Add SSH keys

```
sshkey --username kimchen "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG/N+Pxopl97YZTX0iZuYr+nylclkcG/CMwwrldNw2T3 ed25519-key-20240904"
```

[Post-Installation Scripts](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html#id235)

```
%post
mkdir /mnt/temp
mount 10.10.0.2:/usr/new-machines /mnt/temp
open -s -w -- /mnt/temp/runme
umount /mnt/temp
%end
```

Validate the kickstart file

``` bash
ksvalidator '.ks'
```

#### Using Kickstart Files

Kickstart installations can be performed using a local CD-ROM, a local hard drive, or via NFS, FTP, or HTTP. The KS file is passed as a boot parameter.

```
kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=Rocky9-KS-ISO inst.ks=METHOD/ks.cfg
```

- `inst.stage2=hd:LABEL=Rocky9-KS-ISO`: The installation source. The installer mounts the filesystem using its LABEL, which in this case is the label of the ISO file and sources the necessary files from there.

Methods:

- `inst.ks=http://<server>/<path>`: Fetch the ks file from a web server
- `inst.ks=nfs:<server>:/<path>`
- `inst.ks=cdrom:LABEL=<cdlabel>:/ks.cfg`
- `inst.ks=hd:sda3:/mydir/ks.cfg)`: The installer will mount the filesystem (vfat or ext2) and look for the ks.cfg file

