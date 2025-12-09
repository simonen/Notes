---
tags:
  - package
  - repository
  - centos
  - fedora
  - rhel
---
[[gitea/LINUX 1/SOFTWARE MANAGEMENT/Repositories Intro]] 
[[gitea/LINUX 1/SOFTWARE MANAGEMENT/Package Intro]]

Fedora represents a family of distributions based on Red Hat Linux. 
Red Hat, CentOS, Scientific Linux, Oracle Linux use the same package management
**RPM** and **DNF**

`YUM`: Yellow Dog Modified. Old

`DNF`: Dandified YUM.


When installing packages with **dnf**, the **dnf** database is updated then the **rpm** database. 
When installing packages with **rpm**, the **rpm** base is updated only.

Only packages, installed through managed repositories, are updated. Manually installed packages have to be patched manually.

#### Repositories

`/etc/yum.repos.d/`: Repositories are defined here

##### YUM YellowDog Updater Modified

Install the repositories needed for **PHP**

``` bash
yum install epel-release yum-utils
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

Enable the PHP8.1 module

``` bash
yum-config-manager --enable remi-php8
```

YUM repository cache consists of many small db files that contain package data.

To rebuild the cache without checking for available updates

``` bash
yum makecache
```

To check for updates. This downloads the latest repository cache data and finds out outdated packages.

``` bash
yum [--security] check-updates
yum list updates
```

To install the (--security) updates

``` bash
yum [--security] update
```

To update Individual packages

``` bash
yum update "PACKAGE"
```

Major kernel updates should not be done. Only minor version updates should be done

#### Using dnf to Manage Repositories

- **man dnf**
- **man dnf-config-manager**

List all installed repositories, enabled and disabled

``` bash
dnf repolist --all
```

List enabled repos only

```bash
dnf repolist --enabled
```

Dump the dnf main config

``` bash
dnf config-manager --dump
```

##### Searching, Listing and Inspecting Packages with dnf

To list all available packages in the repos:

``` bash
dnf list all
```

To list installed packages

``` bash
dnf list installed
```

lists a specific package. Useful to check the installed version also and if a new version is available in an online repo.

``` bash
dnf list "PACKAGE"
```

Search for a package by name

``` bash
dnf search "PACKAGENAME"
```

Search for a package by keyword in title and description

``` bash
dnf search all "NAME"
```

Get info about a package

``` bash
dnf info "PACKAGE"
```

Get a list of group sub-commands

``` bash
dnf help groups
```

Get a list of package groups

``` bash
dnf group list "PACKAGE_GROUP"
```

Get an extended list of package groups

``` bash
dnf group list --hidden
```

Search for a package that contains a filename

``` bash
dnf provides "FILENAME"
dnf whatprovides | wp "FILENAME"
```

List package dependencies

``` bash
dnf deplist "PACKAGE"
```

To download a package from the repo

```sh
dnf download "PACKAGE"
```
##### Installing, Removing and Updating Packages

Install | remove a package. DNF removes dependencies automatically. Never use with **-y**

``` bash
dnf install | remove "PACKAGENAME"
```

Install | remove a group

``` bash
dnf group install | remove "PACKAGE_GROUP"
```

Package groups often come with optional packages which are installed if specified.

```
Group: Virtualization Client
 Group-Id: virtualization-client
 Description: Clients for installing and managing virtualization instances.
 Mandatory Packages:
   +gnome-boxes
	...
 Default Packages:
   +virt-top
 Optional Packages:
	...
   python-ipaddr
```

To install a package group plus optional packages

``` bash
yum --setopt=group_package_types=optional groupinstall "Virtualization Client"
```

Display dnf history

``` bash
dnf history
```

```
[kimchen@rhel9 yum.repos.d]$ dnf history
ID     | Command line          | Date and time    | Action(s)      |Altered
--------------------------------------------------------------------------
     4 | remove tigervnc-*     | 2023-12-17 23:36 | Removed        |    4
     3 | install tigervnc      | 2023-12-17 23:35 | Install        |    4
     2 | install nmap          | 2023-12-17 22:52 | Install        |    1
     1 |                       | 2023-12-03 16:27 | Install        | 1200 
```

To undo a an action from history

``` bash
dnf history undo 4
```

The **dnf update** command checks the package version of installed packages and compares it with the version in the online repository then replaces the old version with the newer one.

!!! The **kernel** package is not updated but installed separately alongside the older version. Allows to switch kernel versions in case of some kind of incompatibilities or problems.

##### Managing Repositories with dnf

Download additional.repo and store it in repodir.

``` bash
dnf config-manager --add-repo "http://example.com/some/additional.repo"
```

Create new repo file with `http://example.com/different/repo` as baseurl and enable it.

``` bash
dnf config-manager --add-repo http://"example.com/different/repo"
```

To add a local repo. The **dnf config manager** generates the **.repo** file BaseOS.repo auto:

``` bash
dnf config-manager --add-repo file:///"repo/BaseOS"
```

Enable | disable repository identified by \[REPO_ID] and make the change permanent.

``` bash
dnf config-manager --set-enabled | --set-disabled "REPO_ID"
```

To search only a specific repo, disable everything else

``` bash
dnf --disablerepo=* --enablerepo="REPO" "COMMAND"
```

Using `repoquery`

```bash
dnf repoquery --repo="REPO"
```
#### Using rpm to Manage Software

The rpm Name Structure

```
Source       : autofs-5.1.7-55.el9.src.rpm

- autofs: name of the actual package
- 5.1.7: version of the package
- -55: sub-version
- el9: Red Hat version the package was created for
- x86_64: 32-bit or 64-bit platform the package was created for
- no_arch: no architecture. Can be used on any platform
```

> RPM does not handle dependencies and using it to install and remove packages should be avoided. It is good for querying.

RPM packages are compressed with the **cpio** archiver. The rpm2cpio tool is used to access files from the archive. To extract files from a cpio archive

``` bash
rpm2cpio "PACKAGE.rpm" | cpio -idvm 
```

- `-i`: extract
- `-d`: creates leading directories
- `-v`: verbose
- `-m`: preserves modification times of files
##### Installing, Removing and Updating rpm Packages

Install a rpm package

``` bash
rpm -ivh "PACKAGE"
# i: install
# v: verbose
```

Remove a package

``` bash
rpm -e "PACKAGE"
```

Upgrade a package

``` bash
rpm -U "PACKAGE"
```

To install a local RPM package

``` bash
rpm -Uhv "PACKAGE.RPM"
```

- `-U`: Upgrade if package already installed or install if not
- `-v`: verbose
- `-h`: shows progress with the "#" symbol
##### Querying the RPM database

RPM works with present or already installed packages

``` bash
rpm -q[ a i l d c f p R ] "PACKAGE"
```

- `-qa`: Lists all installed packages. Used with grep to find a specific package:
- `-qi`:  To find info about a specific package:
- `-ql`:  To list all files in a package:
- `-qd`: To list documentation files of a package only
- `-qc`: To list configuration files only:
- `-p`: package is not installed but provided as a file path
- `-q --scripts`  : To query a rpm package for containing scripts
- `-qR`: Show dependencies of a RPM package
- `-q --changes`: See the changelog for a package
- `-q`: Shows the version of the installed package

To find which package a file belongs to:

``` bash
rpm -qf "/FILE_PATH"
```

Check which files have been changed since installation

``` bash
rpm -Va "PACKAGE"
```

To list files in .rpm with paths

``` bash
rpm -qlp "PACKAGE"
```
#### Cleanup Ops

**man package-cleanup**

Package: 
`yum-utils`

To clean up old kernels, leaving only the last N kernels

``` bash
package-cleanup --oldkernels --count="N"
```

To list orphaned packages

``` bash
yum list extras
package-cleanup --orphans
```

#### Building an RPM Package From Source

Packages: 
`rpm-build`
`rpmdevtools`

The following steps will show how to build a package that installs a simple shell script in `/usr/local/bin`

The basic directory structure needs to be created

``` bash
rpmdev-setuptree
ls -lC1 rpmbuild/
```

The following dirs have been created

```
BUILD
RPMS
SOURCES
SPECS
SRPMS
```

Put the shell script in SOURCES

Create the .spec file and move it to the SPECS folder

``` bash
rpmdev-newspec simple_echo && mv simple_echo.spec SPECS/
```

> [!NOTE]- /root/rpmbuild/SPECS/package.spec
> ```
> # Define metadata for the package
> %define name simple_echo
> %define version 1.0
> %define release 1
> %define buildroot %{_tmppath}/%{name}-%{version}-%{release}-root
> 
> Summary: A simple echo script
> Name: %{name}
> Version: %{version}
> Release: %{release}
> License: MIT
> URL: http://www.example.com/simple_echo
> Source0: %{name}-%{version}.tar.gz
> 
> # Define dependencies
> Requires: bash
> 
> # Description of the package
> %description
> This package provides a simple echo script.
> 
> # Preparation steps (unpacking source code)
> %prep
> %setup -q
> 
> # Installation steps
> %install
> rm -rf $RPM_BUILD_ROOT
> mkdir -p $RPM_BUILD_ROOT/usr/local/bin
> install -m 0755 simple_echo-%{version}.sh $RPM_BUILD_ROOT/usr/local/bin/simple_echo-%{version}.sh
> 
> # Files included in the package
> %files
> %defattr(-,root,root,-)
> /usr/local/bin/simple_echo-%{version}.sh
> 
> # Changelog
> %changelog
> * Mon May 06 2024 John Doe <john.doe@example.com> 1.0-1
> - Initial RPM release
> ```

Build the package 

``` bash
rpmbuild -bb simple_echo.spec
```

The RPM is created in `/root/rpmbuild/RPMS/x86_64/simple_echo-1.0-1.x86_64.rpm`

Install the rpm package

``` bash
rpm -ivh /root/rpmbuild/RPMS/x86_64/simple_echo-1.0-1.x86_64.rpm
```

Verify the installation

``` bash
rpm -qi "simple_echo"
```

```
Name        : simple_echo
Version     : 1.0
Release     : 1.el9
Architecture: x86_64
Install Date: Mon 06 May 2024 10:05:13 AM EEST
Group       : Unspecified
Size        : 35
License     : GPLv3+
Signature   : (none)
Source RPM  : simple_echo-1.0-1.el9.src.rpm
Build Date  : Mon 06 May 2024 10:04:04 AM EEST
Build Host  : localhost
URL         : http://www.ebetese.org/simple_echo
Summary     : Echoes From The BAST
Description :
This program e4os the first parametere uZED!
```