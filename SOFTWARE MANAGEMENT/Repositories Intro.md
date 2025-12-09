
Every Linux distribution has 3 types of software management commands:
* single package manager \[ **dpkg, rpm** ]
* dependency-resolving package manager \[ **apt, dnf, yum** ]
* commands to manage groups of related packages \[ **tasksel, group** ]

Package managers by OS:
- `dnf, rpm` - Red Hat, Fedora, CentOS
- `apt, dpkg` - Debian, Ubuntu

Repositories are specific to an operating system. RHEL should only use RHEL repositories, etc
#### Red Hat, CentOS Repositories

man yum.conf
man dnf.conf

https://vault.centos.org provides an archive of old repos.
https://mirror.centos.org/ : current versions repos

Repositories are specified in a file ending in **.repo** in the `/etc/yum.repos.d`
Commonly used parameters in the **.repo** file:

* `[label]` - this identifies the different repositories that might be included in the **.repo** file
* `name=` - Mandatory option that specifies the name of the repository
* `baseurl=` the url of the used repository - **most important!** Format is: **protocol://url**. Protocols could be **http://**, **ftp://** or **file://**. file:// needs additional / because system locations start with /
* `mirrorlist=` - typically used for big online repositories
* `gpgcheck=0 | 1` specifies whether GNU Privacy Guard key validity check should be used to verify files for compromises.
* `gpgkey=`  Specifies the location of the GPG key
* `enabled=0 | 1`

Minimal .repo file

```
[repotonaboga.com_BaseOS]
name=created by dnf config-manager from http://repotonaboga.com/BaseOS
baseurl=http://repotonaboga.com/BaseOS
enabled=1
gpgcheck=0
```

```
[source]
name=CentOS-releasever - Sources
baseurl=http://vault.centos.org/centos/$releasever/os/Source/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

Yum variables:
* `$releasever`: /etc/centos-release; /etc/redhat-release
* `$basearch`: x86_64, uname -p
* `$arch`

> **Time must be synced or else the repo certificates might not work!**

The centos-release file is provided by 

```
yum provides centos-release
centos-release-7-9.2009.0.el7.centos.x86_64 : CentOS Linux release file
Repo        : base
```

```
[root@prometheus teemip]# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
[root@prometheus teemip]# cat /etc/centos-release
CentOS Linux release 7.9.2009 (Core)
```

A list of URLs can also be retrieved from a webserver via mirrorlist= option
`mirrorlist=https://mysite.org/mirrorlist.txt`
The `mirrorlist.txt` file would contain

```
http://repo1
http://repo2
```

To add a web repo from http://mirror.centos.org

```
[reponame]
name=Name...
baseurl=http://mirror.centos.org/centos/$releasever/fasttrack/$basearch/
```

To clean downloaded packages and metadata

``` bash
yum clean all
```
#### Manually Create an rpm Repository

Package: 
`createrepo`

1. All packages must be present
2. `createrepo` command generates the metadata that enables a directory to be used as a repo

This example uses the RHEL9 installation disk

```bash
sudo mkdir /repo
mount /dev/sr0 /repo
```

Add the mount to the fstab

```bash
echo '/dev/sr0 /repo iso9660 defaults 0 0' | sudo tee -a /etc/fstab
```

```bash
yum install -y createrepo
createrepo /DIR
```

Then manually create the new **.repo** file in `/etc/yum.repos.d` and specify label, name and baseurl

If using `dnf config-manager`, `dnf` will create the .repo file.

``` bash
yum-config-manager --add-repo file:///"REPO_DIR"
dnf config-manager --add-repo file:///"REPO_DIR"
```
#### Repository Security - WIP

When using online repositories, it is a good idea to check the files for integrity against compromise.
Secure software usually is signed and a GPG key is provided along with it.
When a repository that uses GPG signed packages is contacted, **dnf** offers to download the GPG key. It is stored in **/etc/pki/rpm-gpg** by default. If the GPG key signature does not match, **dnf** will complain about it.

Internal repositories are less risky than online ones, therefore signing packages is not much required.

Some legacy packages work with outdated, less secure hashing algorithms. For testing purposes only consult the [[LINUX/PKI/System Crypto Policies]]

#### Debian Repositories

Oreilly - Debian 2005 - Good reference for Debian package management

https://wiki.debian.org/SourcesList
https://wiki.debian.org/DebianRepository/Format
[Official List of Debian Mirrors](https://www.debian.org/mirror/list)
https://wiki.debian.org/DebianRepository

Default repo configuration information is stored in
- `/etc/apt/sources.list`

Drop-in sources lists for custom repo configurations go here
- ` /etc/apt/sources.list.d/`

Debian repository anatomy

```
ARCHIVE_TYPE ARCHIVE_ROOT_URL/dist RELEASE_CODE_NAME COMPONENT1 COMPONENT2
```

```
deb http://deb.debian.org/debian/ bookworm main non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main non-free-firmware
```

* eb: type of archive; repository contains binary files
* eb-src: contains source code for applications
* uri: root of the archive
* distribution: 
	* could be the release code name - Named after Toy Story characters  -`bookworm, bullseye, stretch, buster`
	* or release class (suites): `stable, unstable, oldstable`. Avoid using `stable` in sources.list

components (define licensing terms):
* `main` - official free packages
* `contrib` - free packages which might contain non-free dependencies
* `non-free`
* `non-free-firmware` - device firmware could be included in the official repos


To download packages from a repository APT will download InRelase or Release from the $ARCHIVE_ROOT/dists/\$DISTRIBUTION directory

```
http://deb.debian.org/debian/dists/bookworm/

ChangeLog
InRelease
Release
Release.gpg
contrib/
main/
non-free-firmware/
non-free
```

To download the index of the **main** component, apt will scan the Release file for hashes of files in the main 

```
Release
---
Origin: Debian
Label: Debian
Suite: stable
Version: 12.5
Codename: bookworm
Changelogs: https://metadata.ftp-master.debian.org/changelogs/@CHANGEPATH@_changelog
Date: Sat, 10 Feb 2024 11:07:25 UTC
Acquire-By-Hash: yes
No-Support-for-Architecture-all: Packages
Architectures: all amd64 arm64 armel armhf i386 mips64el mipsel ppc64el s390x
Components: main contrib non-free-firmware non-free
Description: Debian 12.5 Released 10 February 2024
MD5Sum:
 0ed6d4c8891eb86358b94bb35d9e4da4  1484322 contrib/Contents-all
 d0a0325a97c42fd5f66a8c3e29bcea64    98581 contrib/Contents-all.gz
 58f32d515c66daafcdac2595fc984814   840179 contrib/Contents-amd64
 834052be97d03cb35ab4d679db464648    61153 contrib/Contents-amd64.gz
 033c926c3a2656ff86318461a0e9c635  4208092 main/binary-all/Packages.xz
 4609852c57e8b51f2cf54a99480b5d67     5628 main/i18n/Translation-tr.bz2
``` 

As seen from the file above, an index may be compressed in the following formats
* no compression (no extension)
* XZ
* gzip, GZIP
* bzip2
* LZMA

The appropriate archiver utilities should be installed on the system

