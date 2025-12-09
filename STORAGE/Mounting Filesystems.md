
Mounting a partition makes its contents accessible through a specific directory
#### Device Names, UUIDs or Disk Labels

UUIDs: universally unique ID. Used by default for device mounting in RHEL9

#### Persistent Mounts with `fstab`

man fstab
man blkid

Anatomy of the `/etc/fstab` file:

`<DEVICE> <MOUNTPOINT> <TYPE> <OPTIONS> <DUMP> <PASS>`

* `DEVICE`: device file. Can be `LABEL='FILESYSTEM_LABEL'` or `UUID="UUID_NUMBER"`
* `MOUNTPOINT`: directory or kernel interface the device needs to be mounted on. Use **`none`** if `swap`
* `FILESYSTEM` type to try: `xfs`, `etx4`, `swap`. Comma separated values can be used. udf,iso9660 will try fs for DVD and CD-ROM respectively
* MOUNT OPTIONS: mount options. defaults. Comma-separated options can be provided. 
* DUMP SUPPORT: legacy feature. 0 in most cases
* AUTOCHECK: the order in which filesystems should be checked
	* 1 if filesystem that should be auto-checked at boot is root fs 
	* 2 for other filesystems. 
	* Always 0 for network filesystems. Commonly set to 0. No check

Entries in the `/etc/fstab` file are processed from top to bottom

SWAP is mounted on a kernel interface. KIs do not start with /

Mount options:
* `defaults`:  `rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, and `async`
* `noauto`: do not mount when mount `-a` is given
* `acl`: access-control list
* `user_xattr`: user extended attributes
* `ro`: read-only mode
* `user`: allow user to mount
* `owner`: allow device owner to mount
* **`atime` / `noatime`**: allows, not allows file access time to be modified
* **`noexec` / `exec`**: allows, not allows programs to be executed
* `error=remount-ro`: remount the fs in read-only mode if a fs error occurs.
* `nodev`: Ignore device files 

If the kernel has not registered an UUID change, the following error might occur

```
sudo mount /mnt/datax
mount: special device /dev/disk/by-uuid/ccd60fc3-bbaf-40e5-a93e-43743f9176d9
does not exist
```

Reload the `udev` device to redetect the UUID and correct the symbolic link `/dev/disk/by-uuid`

```
udevadm control --reload
```

#### Mounting Filesystems

The `mount` command reads the `/proc/self/mounts` file to list mounts. The `-a` options reads the `/etc/fstab` to apply mounts. Entries with the `noauto` flag are skipped.

Existing mounts with their mount options can be listed with the `mount` or `findmnt` utilities. 

```
TARGET                     SOURCE     FSTYPE     OPTIONS
/                          /dev/sda2  ext4       rw,relatime,seclabel
├─/dev                     devtmpfs   devtmpfs   rw,nosuid
│ └─/dev/pts               devpts     devpts     rw,nosuid,noexec
├─/sys                     sysfs      sysfs      rw,...,nodev,noexec,seclabel
│ ├─/sys/fs/selinux        selinuxfs  selinuxfs  rw,nosuid,noexec
```

`mount` basic usage

```bash
mount [-t 'FS_TYPE'] 'SOURCE' 'TARGET'
```

When mounting pseudo-filesystem `SOURCE` can be any arbitrary label, like 'cigar'. 

```bash
mount -t tmpfs cigar /mnt/cigarbox
```

- `-t tmpfs`:  create a `tmpfs` instance
- `/mnt/cigarbox`: attach it here
- `cigar`:  simply to satisfy the `mount` command syntax.

Unmounting 

```bash
umount "TARGET"
```

To show the **mountpoint** and filesystem a path lives under:

```bash
df "PATH" 
```

`PATH=/home/vagrant/.ssh`
```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda2       10215700 3435816   6239368  36% /
```

Output can be limited to filesystem type and mount point

```
df /mnt/foo/file1.txt --output=fstype,target
```

```
Type  Mounted on
tmpfs /mnt
```

##### Nested Mounts

Mounts can be nested under other mounts. `/mnt/foo` is nested under `/mnt`. The nested `/mnt/foo` mount will obscure files and directories that live in the outer `/mnt` mount while it is mounted. 

Create a new `tmpfs` mount in `/mnt` and a new directory `foo`

```bash
mount -t tmpfs tmpfs /mnt
mkdir /mnt/foo
echo "mnt!" > /mnt/foo/file1.txt
cat /mnt/foo/file1.txt
# mnt!
```

Mount another `tmpfs` at `/mnt/foo`. The outer mount point is obscured. Create another `file1.txt` in the same place.

```bash
mount -t tmpfs tmpfs /mnt/foo
echo "foo!" > /mnt/foo/file1.txt
# foo!
```

```
TARGET                     SOURCE     FSTYPE     OPTIONS
└─/mnt                     tmpfs      tmpfs      rw,relatime,seclabel,inode64
  └─/mnt/foo               tmpfs      tmpfs      rw,relatime,seclabel,inode64
```

`df /mnt/foo/file1.txt`
```
Filesystem     1K-blocks  Used Available Use% Mounted on 
tmpfs            1004264     4   1004260   1% /mnt/foo  <---
```

Unmount `/mnt/foo` and check the `file1.txt` again. It is no longer obscured.

```bash
umount /mnt/foo
df /mnt/foo/file1.txt
cat mnt/foo/file1.txt
# mnt!
```

```
Filesystem     1K-blocks  Used Available Use% Mounted on 
tmpfs            1004264     0   1004264   0% /mnt   <---
```

##### Bind Mounts

**Bind mount**: A _mount-level alias_ exposing an existing subtree of a filesystem.

 The `--bind` options creates a **second mount point** for the _same_ filesystem. Only the top-level mount is replicated. Nested mounts are skipped.

```
/mnt
├── backup
│   └── file.txt # back
└── data
    └── file.txt # data
```

```bash
mount --bind /mnt/data /mnt/backup
```

```
TARGET                     SOURCE               FSTYPE  OPTIONS
/                          /dev/sda2            ext4    rw,relatime,seclabel
└─/mnt/backup              /dev/sda2[/mnt/data] ext4    rw,relatime,seclabel
```

- `/dev/sda2`: Real device
- `/mnt/data`: Sub path (inside the device)
- `ext4`: Underlying filesystem type
- `rw..`: Options carried over from source (`/dev/sda2`)

- `/mnt/data` is the **real directory** on `/dev/sda2`.
- `/mnt/backup` is just **another entry point** into the _same inodes_.

Notice how `findmnt` shows **`/dev/sda2[/mnt/data]`** instead of just `/mnt/data`.
This is because bind mounts don’t point to “a directory path” directly — they reference **a location inside another mount**.

Both files share the same `inode` and device. Changes are reflected both ways.

```
File: /mnt/backup/file.txt
Device: 8,2     Inode: 262582
File: /mnt/data/file.txt
Device: 8,2     Inode: 262582
```

`/proc/self/mountinfo`
```
69   1 8:2 / / OPTS shared:1 - ext4 /dev/sda2 OPTS # Normal mount 
172 69 8:2 /mnt/data /mnt/backup rw, shared:1 - ext4 /dev/sda2 # Bind mount
```

- `172 69`: mount ID parent mount ID: `/mnt/backup` is rooted at `/`
- `8:2`: major/minor device number for `/dev/sda2`
- `/mnt/data`: the root of the bind inside the filesystem (mount rooted at `/mnt/data`)

Options 

Changes made to `/mnt/backup` are reflected in `/mnt/data`. To make the `/mnt/backup` read-only, it has to be remounted with the option:

```bash
mount -o remount,ro,bind /mnt/backup/
```

`bind`: Is mandatory 

```
[root@centos10 ~]# echo "write me" >> /mnt/backup/file.txt
-bash: /mnt/backup/file.txt: Read-only file system
```

Files can also be used as mounts

```bash
echo "fake hosts" > /etc/hosts.fake
mount --bind /etc/hosts.fake /etc/hosts
```

```
TARGET               SOURCE               FSTYPE  OPTIONS
/                    /dev/sda2            ext4    rw,relat
└─/etc/hosts         /dev/sda2[/etc/hosts.fake] # /etc/hosts now refers to .f
```

The `/etc/hosts` file is now overridden. Classic trick to override `resolv.conf` in `chroot` or a container.

```bash
cat /etc/hosts
fake hosts
```

###### Recursive Binding

Some directories like `/dev`, `/proc` and `/sys` have nested sub-mounts like `/dev/pts`, etc.. 
Recursive binding ensures that those get replicated as well. 

```bash
mount --rbind / /mnt/chroot
```

Mount the cd in `/mnt/data/cdrom`. Now, bind mount `/mnt/data` into `/mnt/backup`

```
TARGET               SOURCE               FSTYPE
├─/mnt/data/cdrom    /dev/sr0             iso9660
└─/mnt/backup        /dev/sda1[/mnt/data] ext4
```

The directory mountpoint is there but the actual `cdrom` sub-mount has not propagated. 

```bash
ls -l backup/cdrom/
total 0
```

Unmount and remount recursively

```bash
umount backup/
mount --rbind data/ backup/
```

The nested `cdrom/` sub-mount has now propagated.

```
TARGET                  SOURCE               FSTYPE
├─/mnt/data/cdrom       /dev/sr0             iso9660
└─/mnt/backup           /dev/sda1[/mnt/data] ext4
	└─/mnt/backup/cdrom /dev/sr0             iso9660
```

##### Mount Propagation Flags

Linux mount propagation flags control how **mount and unmount events** spread between mount points and across **mount namespaces**. 

```bash
# See current flags
findmnt -o TARGET,SOURCE,PROPAGATION

#TARGET      SOURCE               PROPAGATION
#/mnt/backup /dev/sda2[/mnt/data] shared

# Change them
mount --make-private  /mnt/point        # stop all propagation
mount --make-shared   /mnt/point        # enable full propagation
mount --make-slave    /mnt/point        # receive-only
mount --make-rprivate /                 # isolate an entire namespace
```

- `--make-shared`: **Bi-directional propagation**. every `mount/umount` under this point is automatically mirrored to all peers that share the same “peer-group”.
- `--make-private`: No propagation. Total mount isolation
- `--make-slave`: **One-way propagation**. this mount receives events **from** its master peer-group, but any change it makes does **not** propagate outward.
- `--make-unbindable`: Like **private**, plus **cannot be bind-mounted**; useful for strong isolation.

`/proc/self/mountinfo`
```
shared:123
master:456
propagate_from:456
```

- `shared:1`: The mount is **shared** (propagates events)
- `master:456`: The mount is a slave of.
- (no flag): private

---

PRIVATE PROPAGATION EXAMPLE

---

```bash
watch findmnt -R -o TARGET,SOURCE,PROPAGATION /mnt
```

Start with mounting a `tmpfs` on `/mnt`

```bash
mount -t tmpfs pmount /mnt
```

```
TARGET             SOURCE   PROPAGATION
/mnt               mountain shared
```

Create the mountpoints

```bash
mkdir /mnt/{alpha,beta}
mount -t tmpfs alpha /mnt/alpha
mount --bind /mnt/alpha /mnt/beta
```

```
TARGET             SOURCE    PROPAGATION
/mnt               mountain  shared
├─/mnt/alpha       alpha     shared
└─/mnt/beta        alpha     shared
```

Make the two mountpoints private

```bash
mount --make-private /mnt/alpha
mount --make-private /mnt/beta
```

Now create new mountpoints

```bash
mkdir /mnt/alpha/foo
mkdir /mnt/beta/bar
```

```
/mnt/
├── alpha
│   ├── bar
│   └── foo
└── beta
    ├── bar
    └── foo
```

Create  new mounts in the now private `/alpha` and `/beta` mounts

```bash
mount -t tmpfs secret /mnt/alpha/foo
mount -t tmpfs secret /mnt/beta/bar
```

The new mounts on `foo` and `bar` are now isolated and do not propagate.

```
TARGET             SOURCE    PROPAGATION 
/mnt               mountain  shared
├─/mnt/alpha       alpha     private
│ └─/mnt/alpha/foo secret    private
└─/mnt/beta        alpha     private
  └─/mnt/beta/bar  secret    private
```

Files and directories now can be created in `/foo` and `/bar` independently from each other.

```
/mnt
├── alpha
│   ├── bar
│   └── foo
│       └── secret.key
└── beta
    ├── bar
    └── foo
```

---

SLAVE PROPAGATION

---

Set up 

```bash
mkdir -p /mnt/alpha /mnt/beta
mount -t tmpfs tmpfs /mnt/alpha
mount --bind /mnt/alpha /mnt/beta
```

```
TARGET       SOURCE   PROPAGATION FSTYPE
/mnt         mountain shared      tmpfs
├─/mnt/alpha alphafs  shared      tmpfs
└─/mnt/beta  alphafs  shared      tmpfs
```

Make `/mnt/beta` a `rslave`. Recursive. Every sub-mount below it inherits the propagation flag.

```bash
mount --make-rslave /mnt/beta
mount -t tmpfs master /mnt/alpha/foo
```

Mounts in `alpha` propagate to the slave beta. But not the other way around. 

```
TARGET             SOURCE   PROPAGATION   FSTYPE
/mnt               mountain shared        tmpfs
├─/mnt/alpha       alphafs  shared        tmpfs
│ └─/mnt/alpha/foo master   shared        tmpfs
└─/mnt/beta        alphafs  private,slave tmpfs
  └─/mnt/beta/foo  master   private,slave tmpfs
  └─/mnt/beta/bar  rslave   private       tmpfs
```

```
239 56 0:49 / /mnt/beta rw,relatime master:234 - tmpfs alphafs 
341 239 0:50 / /mnt/beta/foo rw,relatime master:357 - tmpfs master 
406 239 0:51 / /mnt/beta/bar rw,relatime - tmpfs rslave rw,seclabel,inode64
```

If just `slave` was used, bind mounts in `/beta` default back to `shared`

```bash
mount -t tmpfs tmpfs /mnt/alpha
mount --bind /mnt/alpha /mnt/beta
mount --make-slave /mnt/beta
mount --bind /mnt/alpha/foo /mnt/beta/foo
```

```
TARGET            SOURCE      PROPAGATION   FSTYPE
/mnt              mountain    shared        tmpfs
├─/mnt/alpha      tmpfs       shared        tmpfs
└─/mnt/beta       tmpfs       private,slave tmpfs
  └─/mnt/beta/foo tmpfs[/foo] shared        tmpfs
```

> A mount can only be `slave` if it has a `shared` parent.
> If a mount has a `private` parent, `--make-slave` collapses to `private.`

```bash
mount --move 'old_mount' 'new_mount'
```


#### Mounting a Filesystem with `systemd` mount units

[[LINUX/SERVICES/Systemd Units#Systemd Mount Units]]

`systemd` ultimately does the mounts. It generates files off of the fstab file in

`/run/systemd/generator`

Manual mount files are made in `/etc/systemd/system/*.mount`
The name of the .mount file corresponds to the directory the filesystem is mounted on
