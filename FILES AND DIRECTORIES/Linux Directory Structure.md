
#### CHROOT

man chroot

chroot - run command or interactive shell with special root directory. This provides isolation to an application, limiting it to its own root environment.

To create a new root directory and work in its isolated environment

``` bash
mkdir -p /new/root
```

``` bash
chroot /new/root
[root@prometheus root]# chroot /new/root/ /bin/bash
# chroot: failed to run command ‘/bin/bash’: No such file or directory
```

Create a /bin directory and copy the /bin/bash to it

``` bash
mkdir /bin ; cp /bin/bash /bin
chroot /new/root/
# chroot: failed to run command ‘/bin/bash’: No such file or directory
```

Bash is dynamically linked to a series of libraries. To locate dynamically linked libraries to an executable:

```
ldd /bin/bash
```

```
[root@prometheus root]# ldd /bin/bash
        linux-vdso.so.1 =>  (0x00007ffda1da9000)
        libtinfo.so.5 => /lib64/libtinfo.so.5 (0x00007f667687a000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f6676676000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f66762a8000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f6676aa4000)
```

`ldd` should be used on trusted executables only, as it can execute arbitrary code. 
On untrusted execute use:

``` bash
objdump -p /bin/bash | grep NEEDED
```

Copy the libraries to /new/root/lib64 and execute chroot /new/root

``` bash
bash-4.2#
```

When chrooting the root directory, it does so without the pseudo filesystems like `procfs`, sysfs... 

For example: as `lsblk` command reads the `sysfs` filesystem and `udev db` to gather information about block devices, the `/sys` directory must be mounted in the chroot env.
When using a SystemRescue disk which comes with pseudo filesystems, this can be achieved by binding them to the chroot env.

``` bash
mount -o bind "SR_DISK/proc" "/CHROOT_MOUNT_POINT"
```

Bind mount is a way of making a directory available in multiple places without using symlinks, which is not possible from inside of a chroot env.