
Packages `CentOS):
- `epel-release`
- `lxc`: client
- `lxc-templates`
- `bridge-utils`
- `debootstrap`: Tool to bootstrap minimal Debian root filesystem into a directory

The `lxc` tools do creation and manipulation of namespaces 

List available templates

```bash
ls -ls /usr/share/lxc/templates/
```

```
16 -rwxr-xr-x 1 root root 13630 Jun 14  2024 lxc-alpine
12 -rwxr-xr-x 1 root root 11234 Jun 14  2024 lxc-archlinux
12 -rwxr-xr-x 1 root root  8601 Jan  3  2025 lxc-busybox
32 -rwxr-xr-x 1 root root 30633 Jun 14  2024 lxc-centos
32 -rwxr-xr-x 1 root root 30408 Jun 14  2024 lxc-debian
16 -rwxr-xr-x 1 root root 14497 Jan  3  2025 lxc-download
44 -rwxr-xr-x 1 root root 41919 Jun 14  2024 lxc-fedora
20 -rwxr-xr-x 1 root root 16877 Jun 14  2024 lxc-opensuse
 8 -rwxr-xr-x 1 root root  6845 Jun 14  2024 lxc-sshd
28 -rwxr-xr-x 1 root root 26276 Jun 14  2024 lxc-ubuntu
12 -rwxr-xr-x 1 root root 11747 Jun 14  2024 lxc-ubuntu-cloud
```



Download CentOS images from

https://cloud.centos.org/centos/9-stream/x86_64/images/

#### Debian Containers

```bash
sudo mkdir -p /var/lib/lxc/debian1/rootfs
```

```bash
sudo debootstrap bookworm /var/lib/lxc/debian1/rootfs/ http://deb.debian.org/debian/
```

