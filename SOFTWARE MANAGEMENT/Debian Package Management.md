---
tags:
  - debian
  - ubuntu
  - "#repository"
  - package
---

Debian based distributions use the following software management commands

**aptitude**: terminal-based package manager
`apt`: Advanced Packaging Tool: dependency resolver
`dpkg`: Single package manager
`tasksel`: Managing package groups

> The `apt` command is intended for interactive use. Behavior can change in the future
> The `apt-get` command is for scripting and automation
#### Apt Cache

Installed Debian packages are first downloaded and stored in `/var/cache/apt/archives`. The cache can be shared among multiple clients to install already downloaded software. Utilities for this are: `apt-cacher`, `apt-cacher-ng`, and `apt-proxy`.

#### Installing and Removing Debian Repositories

`/etc/apt/sources.list.d`

Learn the codename of the Debian release

```
lsb_release -sc
```

```
root@server15:~# lsb_release -sc
No LSB modules are available.
bookworm
```

Add a Debian repository

```bash
add-apt-repository "deb URL" 'CODENAME'/'SUITE'  'COMPONENTS'
```

Remove a repository

```bash
apt-get repository -r 'REPO'
```

When installing or removing repositories, update the package cache

```bash
apt update
```

To download repository updates and install the upgrades

```bash
apt upgrade
```

List package dependencies

```bash
apt depends 'PACKAGE'
```

#### Using apt to Search, Inspect, Install and Remove Packages

Search for a package

``` bash
apt search "PACKAGENAME"
```

Limit the search to package names that include the search term

``` bash
apt search "PACKAGE" --names-only
```

Get detailed info on a package

``` bash
apt show "PACKAGE"
```

Install / remove package. Remove does not remove conf files

``` bash
apt install | remove "PACKAGENAME"
```

To remove a package along with conf files

``` bash
apt remove purge "PACKAGENAME"
```

#### Using dpkg to Install, Remove Packages

man dpkg

To install a `dpkg` package

```bash
dpkg -i 'PACKAGENAME'
```

Remove a package

``` bash
dpkg -r "PACKAGENAME"
```

Remove a package and its conf files

``` bash
dpkg -P | --purge "PACKAGENAME"
```

##### Using dpkg to Query and Inspect Packages

`dpkg` can query installed packages or .deb files. When querying installed packages only the name of the package is used as in "fdisk". When querying a .deb file, the full file path of the package is provided ./fdisk_2.38.1-5+deb12u1_amd64.deb

``` bash
dpkg-query [OPTION] [PACKAGE.deb]
```

List all `dpkg-query` commands

```
dpkg-query --help
```

List all installed packages

``` bash
dpkg -l | --list
```

```
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name             Version    Architecture Description
+++-================-==========-=======-==================================>
ii  accountsservice  22.08.8-6  amd64   query and manipulate user account 
ii  acl              2.3.1-3    amd64   access control list - utilities
```

Statuses
* Desired
* Current
* Error

Status codes:
`ii` - Package is installed and will install updates if available
`hi` - package is on hold and installed but will not install updates
`un` - package is not installed
`rc` - package is not installed, but there are leftover conf files. (Usually after uninstalling)

List the contents of an installed package

``` bash
dpkg -L | --listfiles "PACKAGENAME"
```

List the contents of a .deb package file

``` bash
dpkg -c | --contents "PACKAGE.deb"
```

Get info about a deb package file

``` bash
dpkg -I "PACKAGE.deb"
```

Get info about an installed package

``` bash
dpkg -p "PACKAGENAME"
```

Find which package an installed file belongs to (what provides)

``` bash
dpkg -S | --search "PATH_TO_FILE"
```
#### Using tasksel

**tasksel** manages tasks (debian package groups)

To list available tasks (groups)

``` bash
tasksel --list-tasks
```

Install / remove a task

``` bash
tasksel install | remove  "TASK"
```

#### Compiling from source

1. Configure the application
2. Compile or make the application
3. Install the application

Download the app source. In this case nginx

``` bash
wget -c http://nginx.org/download/nginx-1.10.1.tar.gz
```

-c: resume the download if network interrupted from where it left off

Extract the compressed archive 

``` bash
tar -zxvf nginx-1.10.1.tar.gz
```
#### Configure the Application

[Info about the make command]( http://www.tutorialspoint.com/makefile/why_makefile.htm f)

Install the required dependencies:

``` bash
apt intall -y zlib1g-dev libpcre3-dev
```

``` bash
cd nginx-1.10.1
./configure --help
```

Create a /opt/nginx dir and a system user nginx

Configure the installer

``` bash
./configure --prefix=/opt/nginx --user=nginx --group=nginx
```

The configuration command writes a configuration header and a special file **Makefile** which contains commands needed to build the software by the **make** command

Execute the make command to compile the app then the make install command

``` bash
make [-j"CPU_CORES"] # for parallel computing
make install
```

