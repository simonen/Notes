---
tags:
  - samba
  - kerberos
  - centos
---
Packages:
* `samba`: Samba daemons and conf files
* `cifs-utils`: Utilities for mounting and managing CIFS mounts
* `samba-client`: Utilities required to set up SMB shares

V3: emulating NT4. Supports LDAP
V4: emulating Active Directory with forest level up to Windows Server 2008. 4.19 support for higher forest level is experimental. LDAP experimental.

### Samba Configuration

Configuration file: 
`/etc/samba/smb.conf`

##### Samba Services

Services are described in \[sections]. Some services in are considered 'special' like: 
`[global]`; `[printers]`'; `[homes]` and user-defined services: `[Shared Dir]`

``` bash
testparm -v
```

`/etc/samba/smb.conf`:
* `[global]`: basic samba parameters:
	* `workgroup`: Windows workgroup the samba server is a member of or a smb domain
	* `security`: Indicates how security is handled. `security=user` requires a valid samba `managed` username that is mapped to a Linux user account name
	* `host allow`: comma or tab spaced list of allowed hosts. **man 5 hosts_access**
	* `load printers`: if printers from the CUPS system are shared through SAMBA
	* `cups options`: specifies that print driver processing is handled by CUPS
	* `log files`: name of the log file
* `[homes]`: contains default values for accessing shared home directories
* `[printers]`: used to provide access to printers using the CUPS system
* `[user-defined services (shares)]`: go to the bottom of the file

#### Samba Users

`security = user` :  the default mode security modes that controls how authentication is handled for accessing shares.

When working as a stand-alone server, Samba uses its own internal users for authentication, stored in its own database, but still needs to interact with the Linux filesystem through regular Unix users/groups.

How `security = user` works:

1. `Authentication`: The Samba client authenticates against the Samba server, which checks credentials in its configured backed password store.
2. `Authorization`: After the user authenticates successfully, Samba checks the share-specific configuration to determine if the user is authorized to access that share.
3. `Access`: Permissions (read-only or write access) are determined based on the Samba configuration and the Unix file permissions of the shared directory.

Unlike Unix users, who are mapped by default from 1000-60000, Samba users are mapped 300000-8000000. Incoming client requests come with the Samba UID/GID, therefore the Unix directory owner must match them.

Default Unix user
`kimchen:x:1000:1000`
Default Samba user
`kimchen:x:3000000:100`

Windows clients do not need the corresponding Unix users for logins, therefore they could be without passwords, shells and home directories.

##### Samba User and Group Management

> smbpasswd is obsolete

Tools: 
`samba-tool`, `wbinfo`, `smbpasswd`, `pdbedit`

Create a Samba user, optionally with a custom UID

``` bash
samba-tool user create 'USERNAME' [-uid='UID']
```

Create a Samba group, optionally with a custom GID

``` bash
samba-tool group create 'GROUP' [-gid='GID']
```

Add a Samba user to a Samba group

``` bash
samba-tool group addmembers 'GROUP' 'USERNAME'
```

Samba user UID/GID can be manually changed with the samba-tool also.

**The `wbinfo` tool** is very handy for retrieving information about users, groups, domains

Get a group GID

``` bash
wbinfo --group-info 'GROUP'
# SAMBA\sambagroup:x:3000033:
```

List all domain users

``` bash
wbinfo -u
```

Test Samba user authentication

``` bash
wbinfo -a 'SAMBA_USER'
```

#### Samba Shares

##### Firewall and SELinux 

Allow samba through the firewall

```bash
firewall-cmd --add-service=samba --permanent ; firewall-cmd --reload
```

Change the SELinux context of the shared directory

```bash
semanage fcontext -a -t samba_share_t "/SMB_SHARE(/.*)?"
```

or using `chcon` if `semanage` is not available. `Semanage` - better

```bash
chcon -R -t samba_share_t /shares -v
```

[Setting up a Share Using POSIX ACLs](https://wiki.samba.org/index.php/Setting_up_a_Share_Using_POSIX_ACLs)

Minimal samba share configuration

`/etc/samba/smb.conf`
```
[global]
...
[shared]
	path = /DIR
```

For more info [[gitea/LINUX 1/NETWORK STORAGE/SAMBA/Service Sections and Directives#Share Directives]]

Common directory share options:
* `path`: path of the shared directory on the Linux filesystem
* `writable`: if permitted by Linux fs permissions, authenticated users have write access to the share. If set to no, a comma separated list of Linux users only are allowed write acc
* `read only`: `read only no` == `writable yes `
* `write list`: comma separated list of users who have write access even if writable is set to no. `@<group>` or `+<group>` - specifies groups
* `valid users`: limit access to the share to a list of users. All users by default are allowed
* `comment`: 
* `guest ok`: bypasses all security settings
* `browsable`: users can navigate through the structure of the share and see items available. Should be set to `no` on the `[homes]` share
* `create mask = `: the default `rwx` mask for new files
* `directory mask =` : the default `rwx` mask for new directories

To enable write access on a Samba share:
* users with no write permissions on the Linux fs will not have write permissions on the share
* if `writable` is set to yes, all Linux users with write permissions on the Linux fs will have write access to the share
* `read only = no`  == `writable = yes`
* if `read only = yes` , users in the `write list` will have write access

- `force user = <unix directory owner>`
- `force group = <unix group directory owner>`

Samba users will have their effective UID/GID overridden by the Unix directory owner UID/GID. In that case the Samba users do not need to have corresponding Unix UID/GIDs.

To test the `smb.conf` file for syntax errors or show current configuration:

``` bash
testparm
```

To list active samba sessions 

```
smbstatus
```

```
ps aux | grep smbd
```

Package: 
- `samba-client`

To list samba shares

``` bash
smbclient -L 'HOST' [-U 'SAMBA USER']
```

#### Mounting Samba Shares

Package: 
- `cifs-utils`

man mount.cifs for all mount options

``` bash
mount -t cifs -o username='SMB_USER' //'SMB_SERVER'/'[SMB_SHARE]' /'MNT_POINT'
```

User credentials can be stored in a file (`root:root, 400`). Good for single desktop access, not shared environments

`/home/user/.creds`
```
username=USERNAME
password=PASSWORD
```

```
mount ... -o credentials=/home/user/.creds
```

`-o uid='UID'` will automatically set the file owner UID on the mounted filesystem. Defaults to 0 (root)

Automatic mount

`/etc/fstab`
``` bash
//'SMB_SERVER'/'SHARE' /'MOUNTPOINT' cifs credentials=/'PATH_TO_CRED_FILE', _netdev 0 0
```

##### Multiuser Samba Mount

By default, CIFS mounts only use a single set of user credentials (the mount credentials) when  accessing a share, locking the session to that user and what he is allowed to do. `multiuser` opens a new session on top of the established one whenever a new user accesses the mount.

The mount could be done as an unprivileged samba user, then new users accessing it, by opening a new session, are now able to escalate the original privilege by authenticating as another privileged samba user. This is done by caching new credentials with `cifscreds` tool.

This allows multiple users to access a single mount point made by the administrator, authenticate with their own credentials and receive the correct privileges.

On the server, an unprivileged samba user `sambo` is used only for accessing the share. The privileged `sambagroup` members are lisa, bill and melissa.

`drwxrwsr-x. 2 root sambagroup 121 Oct  2 12:53 /data/sambashare/`

```
[sambashare]
        comment = My Samba Share
        path = /data/sambashare
        write list = @sambagroup
```

Mount the share on the client with the multiuser option. It requires that credentials are stored in a file, as the Linux Kernel does not ask for passwords

``` bash
mount -t cifs //'SAMBA_SERVER/SHARE' /'MOUNT_POINT' -o credentials='/.CREDS',multiuser
```

`/etc/fstab`
``` bash
//SMB_SERVER/SHARE /'MOUNTPOINT' cifs credentials='CRED_FILE',multiuser,sec=ntlmssp  0 0
```

Show current client sessions.

```
sambastatus
```

```
Samba version 4.18.5
PID     Username     Group        Machine                    Protocol Version   
------------------------------------------------------------------------------
3353    SAMBA\sambo  users        ipv4:192.168.137.10:35786) SMB3_11           

Service      pid     Machine       Connected at                     
------------------------------------------------------------------------------
IPC$         3353    192.168.137.10 Wed Oct  2 05:32:29 PM 2024 EEST -
sambashare   3353    192.168.137.10 Wed Oct  2 05:32:29 PM 2024 EEST -
```

Access the mount from another user on the client machine

``` bash
[kimchen@dc1 shared]$ ll
ls: cannot open directory '.': Permission denied
```

Open a new session as another samba user

``` bash
cifscreds -u 'PRIVILEGED_SAMBA_USER' add 'SMB_SERVER'
```

Now the current user on the client can access the share on behalf of the privileged samba user. Created files will be owned by the privileged samba user.

```
[kimchen@dc1 shared]$ ll
total 32
-rwxr-xr-x. 1 kimchen kimchen 0 Oct  2 16:49 alabala
```

Here we see 4 sessions from 4 different users for the same share

```
Samba version 4.18.5
PID     Username     Group        Machine                      Protocol Version
-------------------------------------------------------------------------------
3353    SAMBA\melissa users        (ipv4:192.168.137.10:35786) SMB3_11
3353    SAMBA\lisa    users         (ipv4:192.168.137.10:35786) SMB3_11
3353    SAMBA\sambo   users         (ipv4:192.168.137.10:35786) SMB3_11
3353    bill          users         (ipv4:192.168.137.10:35786) SMB3_11

Service      pid     Machine       Connected at                     
-----------------------------------------------------------------------
IPC$         3353    192.168.137.10 Wed Oct  2 05:35:12 PM 2024 EEST - 
sambashare   3353    192.168.137.10 Wed Oct  2 05:33:34 PM 2024 EEST - 
IPC$         3353    192.168.137.10 Wed Oct  2 05:47:34 PM 2024 EEST -
IPC$         3353    192.168.137.10 Wed Oct  2 05:32:29 PM 2024 EEST -
sambashare   3353    192.168.137.10 Wed Oct  2 05:32:29 PM 2024 EEST -
IPC$         3353    192.168.137.10 Wed Oct  2 05:33:34 PM 2024 EEST -
sambashare   3353    192.168.137.10 Wed Oct  2 05:35:12 PM 2024 EEST -
sambashare   3353    192.168.137.10 Wed Oct  2 05:47:34 PM 2024 EEST -
```

###### Automounting Samba Shares with autofs

Install the `autofs` package on the client

On the client in 

`/etc/auto.master` -> client
```
/LOCAL_ROOT_MOUNTPOINT /etc/auto.samba
```

`/etc/auto.samba`
```
LOCAL_MOUNTPOINT_DIR_NAME -fstype=cifs,username=USERNAME,password=PASSWORD ://SAMBASERVER/SHARE
```

Protect the `/etc/auto.samba` file because of the crendentials

```bash
chmod 400 /etc/auto.samba
```
####  Samba Security

##### SELinux

* `samba_share_t`
* `public_content_t`
* `public_content_rw_t`
* `smbd_anon_write`: allows write access for anonymous samba users. Works with `public_content_t` context type
* `samba_enable_home_dirs`: allows Llinux home dirs to be shared with samba
* `use_samba_home_dirs`: allows remote SMB file shares to be mounted and shared as Linux home dirs

For authenticated access

``` bash
semanage -a -t samba samba_share_t "/*DIR*(/.\*)?" ; restorecon -Rv /'DIR'
```

To allow anonymous access:

``` bash
semanage -a -t public_content_rw_t "/*DIR*(/.\*)?" ; restorecon -Rv /'DIR'
setsebool smbd_anon_write on
```

##### SMB Firewalling

The samba service **samba.xml** allows the following through the firewall:
* `udp 137`: NetBIOS name services
* `udp 138`: netbios datagram
* `tcp 139`: netbios ssn
* `tcp 445`: Microsoft Directory Services
* `nf_conntrack_netbios_ns` module. NetBIOS name service connection tracking

``` bash
firewall-cmd --info-service=samba
firewall-cmd --add-service=samba --permanent ; firewall-cmd --reload
```
#### Kerberized Samba Server

1. Join the samba server to the domain. 
2. Create the service:

```bash
ipa service-add cifs/'SMBSERVER'
```

```bash
kinit -k
```

```bash
ipa-getkeytab -s 'IPA_SERVER' -k /etc/krb5.keytab -p cifs/SMB_SERVER_FQDN
```

To verify the list of active shares with Kerberos:

```bash
smbclient -k -L //'SMB_SERVER_FQDN'
```

##### Access Control Lists

Windows clients use ACLs to manage permissions. When files are created on a Samba share, Samba may map these Windows ACLs to the underlying file system.

```
-rwxrwx---+ 1 3000000 users  7 Sep 30 14:39 gina.rtf
```

#### Troubleshooting SAMBA

`protocol negotiation failed: NT_STATUS_IO_TIMEOUT`

This could indicate a problem with the DNS server.

##### Problem: Inconsistent UID and GIDs 

> Samba clients use the `winbind` mappings

**`wbinfo`**: This command is part of the `Winbind` service, which is responsible for resolving names (users/groups) from Windows domains or Active Directory environments and mapping them to Unix UIDs and GIDs. Winbind maintains its own mappings of user/group SIDs to UIDs/GIDs, which might differ from local Unix accounts.

**`pdbedit`**: This command manipulates the Samba user database (`passdb`). It will list users stored in the Samba database, including any Unix-to-Samba user mappings (from `/etc/passwd` or `Winbind`).

Creating Samba users with `smbpasswd` will use the UID of an existing Unix user with the same login name, while `wbinfo` will use its own mapping, which are used for authentication. 

force user = 1004 in smb.conf

```
[2024/09/30 23:08:08.118777,  0] ../../source3/smbd/smb2_service.c:120(chdir_current_service)
Sep 30 23:08:08 dc1.samba.ohio.net smbd[18945]:   chdir_current_service: vfs_ChDir(/srv/samba/demo) failed: Permission denied. Current token: uid=1004, gid=1004, 6 groups: 1004 3000013 3000014 3000003 3000009 3000016
```

Indicates that the Samba process (`smbd`) is unable to change directories to `/srv/samba/demo` due to **permission issues**. The user with `uid=1004` and `gid=1004` doesn't have sufficient permissions to access the directory.

Windows 

List open SMB server sessions (CMD)

``` bash
net use
```

If windows is erroring out about being unable to open multiple sessions, delete the current ones

```cmd
net use * /delete
```

You can also disconnect Samba sessions or mapped drives using PowerShell.

2. **List Active Network Sessions**: 
    `Get-WmiObject -Class Win32_NetworkConnection`
3. **Disconnect All Network Connections**: 
    `Get-WmiObject -Class Win32_NetworkConnection | Remove-WmiObject`