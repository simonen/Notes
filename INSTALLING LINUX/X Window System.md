
CentOS 7

Packages
"X Window System" group
"Gnome Desktop" group

To install a graphical interface on top of a minimal Linux installation, mount the installation media in a directory as configured in the  /etc/yum.repos.d/CentOS-Media.repo

 /etc/yum.repos.d/CentOS-Media.repo
 
```

```

Install the required package groups

``` bash
yum --disablerepo=* --enablerepo=c7-media group install 'X Window System'
yum --disablerepo=* --enablerepo=c7-media group install 'Gnome Desktop'
```

Start the graphical interface

``` bash
startx
```
