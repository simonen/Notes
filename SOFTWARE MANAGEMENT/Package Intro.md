---
tags:
  - package
  - centos
  - repository
---

Packages fall into three main categories:
* application packages
* library packages
* development packages

**Library packages** contain files used by applications and operating system to provide additional functionality. Cryptography support is provided by the **libssl** package. A library package may be used by multiple different applications, hence they are distributed separately, instead of being included in every application package. Names start with **lib**

**Development packages** contain source code and header files used to compile packages from source. Names end in -dev (Ubuntu), -devel (Red Hat). 

A package contains:
* data: files to be installed
* metadata
* control files: descriptive info about the package, installation scripts

**BaseOS**: repo for core operating system packages. Package major versions remain unchanged during the distribution's lifetime.

**AppStream**: this repo includes major versions of packages

**module**: a set of **RPM** packages that belong together. Different versions and profiles can be provided. Each module can have one or more application streams. Working with modules allows the use of a specific **profile**.

**stream**: contains one specific version of a package and updates are provided for a specific stream. By using streams, different version of a package can be offered through the same repository.
Only one stream can be enabled at a time when working with modules with multiple streams.
**upstream rpms**: packages that contain newer version of the application than the one shipped with the distribution. Can be built by anyone. May not be stable, secure, maintained.

**profile**: a list of packages that are installed together for a particular use case. A profile can be **minimal**, default**,** **server**, etc.

#### Package Modules

To list modules:

``` bash
dnf module list
```

```
[kimchen@rhel9 ~]$ dnf module list
Last metadata expiration check: 13:03:28 ago on Sun 17 Dec 2023 11:06:30 PM EET.
created by dnf config-manager from file:///repo/AppStream
Name                  Stream           Profiles                                        Summary
maven                 3.8              common [d]                                      Java project management and project comprehension tool
nginx                 1.22             common [d]                                      nginx webserver
nodejs                18               common [d], development, minimal, s2i           Javascript runtime
nodejs                20               common [d], development, minimal, s2i           Javascript runtime
php                   8.1              common [d], devel, minimal                      PHP scripting language
```

To list a specific module

``` bash
dnf module list 'nginx'
```

Get module info

``` bash
dnf module info 'nginx'
```

```
[kimchen@rhel9 ~]$ dnf module info nginx
Last metadata expiration check: 13:08:24 ago on Sun 17 Dec 2023 11:06:30 PM EET.
Name             : nginx
Stream           : 1.22
Version          : 9030020231017214601
Context          : 9
Architecture     : x86_64
Profiles         : common [d]
Default profiles : common
Repo             : repo_AppStream
Summary          : nginx webserver
Description      : nginx 1.22 webserver module
Requires         : platform:[el9]
Artifacts        : nginx-1:1.22.1-5.module+el9.3.0.z+20438+032561a0.src
                 : nginx-1:1.22.1-5.module+el9.3.0.z+20438+032561a0.x86_64
```

To find info about a specific stream

``` bash
dnf module info 'nginx:1.22'
```

To enable a specific version. This disables the old streams and enables the new one.

``` bash
dnf module enable 'nginx:1.2'
```

To ensure all dependent packages that are not in the module are updated to finalize the procedure

``` bash
dnf module distro-sync
```