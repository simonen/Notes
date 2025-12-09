**NFS** is used mainly:
* to provide access to home directories for **LDAP** users
* to easily access shared filesystems on other Linux hosts

**NFSv4** default version on RHEL7. Offers:
* **pseudo root mount**: mounting the root directory on the client mounts all exported dirs on the server instead of mounting them individually
* **Kerberos security**: NFS was designed with host-based security only. **kerberos** allows for user-based security by authentication
* **simplified firewalling**: old NFS versions used random ports for processes. Newer versions have fixed **TCP ports**:
	* `2049`: NFS processes
	* `111`: rpc-bind (`showmount` command)
	* `20048`: mountd

**NIS** Network Information Service: provides a network-based authentication server. Deprecated

- `/etc/exports`: in this file exports are defined
- `/etc/exports.d`/: files, defining exports, with `.export` extension go here

```
# Export structure
/<EXPORTED_DIR> <CLIENT_HOSTS>(OPTIONS)

Example:
/nfsexports *(rw, no_root_squash)
/srv/nfsexport vm[1-10].example.com(rw,no_root_squash)
```

Note there is not space between the client hosts and the options. A space would give anyone in the world full access while restricting the specified hosts to ro.

Export options:
* `ro`: read-only
* `rw`: read / write
* `no_root_squash`: by default NFS maps the root user to **nfsnobody** which has minimal permissions. no_root_squash maps root user on the client to uid=0 (root) on the server, which is very insecure
* `all_squash`: Map all uids and gids to the anonymous user.
* `anonuid=UID,anongid=GID` : These  options explicitly set the uid and gid of the anonymous account
* `fsid=`: `root` | `0` : designates a directory as the root of the NFS export tree for NFSv4.
* `hide`: Prevents clients from accessing or viewing files that exist below the mount point of a subdirectory.
#### Setting up NFS

Packages: 
`nfs-utils.x86_64`

Install and start NFS Server

``` bash
dnf install -y nfs-utils
systemctl enable --now nfs-server.service
```

##### Allow NFS through the firewall

``` bash
for i in nfs rpc-bind mountd; do firewall-cmd --add-service=$i --permanent; done
firewall-cmd --reload
```

##### SELinux for NFSv4

Important **SELinux** context labels:
* `nfs_t`: allows the NFS server to access the share
* `public_content_t`: read only access to the contents
* `public_content_rw_t`: allows read/write access to NFS and other services

**Booleans** (on by default):
* `nfs_export_all_ro`
* `nfs_export_all_rw`

#### Export shares:

man `exportfs`
man 5 exports (export options)

``` bash
exportfs -o 'OPTIONS' 'CLIENT_HOSTS':/'DIR'
```
or
add the shares in the `/etc/exports` | `/etc/exports.d/.export`

Some `exportfs` options:
- `-a`: To export all un-exported shares in `/etc/exports`
- `-r`: To re-export and clean non-existing exports:
- `-s`: To display exports and their options on the server

Security methods:
* `none` : Anonymous access based on the `nobody` permissions. Used with `nfsd_anon_write `boolean. Insecure
* `sys`: Default security method. Access is based on `UID` and `GUID`. Incoming user UID is mapped to the same UID on the server, even if names do not match.

To discover NFS shares:

``` bash
showmount -e 'NFS_SERVER'
```

To list NFS (network) mounts only:

```
mount -t nfs4
findmnt -t nfs4
```
#### Mounting NFS Shares

Root mount on the client mounts only the discoverable NFS shares

``` bash
mount 'NFS_SERVER':/ /'MOUNTPOINT`
```
##### Automatic Mounting

`/etc/fstab`
``` bash
'NFS_SERVER':/'SHARE' /'MOUNTPOINT' nfs _netdev 0 0
```

`sync`: ensures the modified data is committed immediately, and not written to buffers first.
* `_netdev` : tells the mount command to skip filesystems until the network service is completely started. Obsolete and replaced by `remote-fs.target`
* `remote-fs.target`: should be running

#### Mounting as Systemd Unit

Create a unit file in **/etc/systemd/system/** named *MOUNT*.**target** on the client machine

```
[Unit]
Description=NFS Share info
After=nfs-client.target
Requires=nfs-client.target

[Mount]
What=<NFS_SERVER>:/<SHARE>
Where=<MOUNT_POINT>
Type=nfs
Options=

[Install]
WantedBy=multi-user.target
```
##### **Automount**

Package: 
`autofs`

`autofs`: mounts a share on demand when access is requested, not permanently. Works in user space, no root permissions are required

Install and start `autofs`

```bash
dnf install -y autofs
```

Discover NFS shares on a server

```bash
shoumount -e 'SERVER'
```

```
[kimchen@rhel81 files]$ showmount -e server1
Export list for server1:
/users   *
/nfsdata *
```

 `/etc/auto.master`
```
 /LOCAL MOUNT_POINT ROOT_DIR /etc/auto.'EXPORT'
```

`/etc/auto.'EXPORT'`
```
LOCAL_MOUNTPOINT_DIR -rw(options) 'SERVER':/'EXPORT'
```

EXAMPLE:

Directory `nfsserver:/data` is exported

`/etc/auto.master`:
```
/exported /etc/auto.exported
```

`/etc/auto.exported`
```
nfs -rw nfsserver:/data
```

On the client:

`/exported/nfs` will be the mountpoint  of `nfsserver:/data`

The export is mounted automatically upon accessing, not mounted explicitly.
##### Wildcards in `autofs`

It is a good practice to have the user home directories stored in a central NFS server, not locally, and have them automounted when the user logs in.

`/etc/auto.master`
```
/LOCAL_MOUNT_POINT /etc/auto.EXPORT
```

`/etc/auto.'EXPORT'`
```
* -rw server:/'EXPORT'/&
```

- `*` : arbitrary local mount point
- `&`: matching item on the remote server

Restart the `autofs` daemon after configuring the files

```bash
systemctl restart autofs
```

#### NFSv4 SELinux Transparency

By default the NFS clients mount a share as `nfs_t` context type, regardless of the type set on the server.

For NFS v4.1 and prior the context type can be define as a mount option:

``` bash
mount -o context="system_u,object_r:public_content_rw_t:s0" 'NFSERVER':/'EXPORT' /'MOUNTPOINT'
```

To make the SELinux context of the share on the server transparent to the client for NFSv4.2

`/etc/sysconfig/nfs` on the server
```
RPCNFSDARGS="-V 4.2"
```

On the client:

```bash
mount -o v4.2 'NFSSERVER':/'EXPORT' /'MOUNTPOINT'
```
