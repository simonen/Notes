
Packages: 
`policycoreutils, selinux, setools`

Get detailed status about SELinux

``` bash
 sestatus -v
```

All system calls are denied by default
Changing between on and off **SELinux** modes requires system reboot

- `source domain`: the object that is trying to access something. Typically a process or a user
- `target domain`: the object that is accessed. Typically - file, directory, port
- `context labels`: define what exactly is allowed
- `rule`: specific part of the policy that determines access permissions
- `label`: 

- `enforcing mode`: SELinux is fully operational and enforcing all policy rules
- `permissive mode`: SELinux-related activity is logged, nothing is blocked. Good for troubleshooting. Insecure

logs are written in:
- `var/log/audit/audit.log`

Conf file to change default SELinux mode while booting
- `/etc/sysconfig/selinux`

GRUB kernel options are be used to set `SELinux` modes
- `selinux=0`: disable `SELinux`
- `enforcing=0`: permissive mode

- `getenforce`: get the current SELinux mode
- `setenforce 0`: temporarily sets permissive mode
- `setenforce 1`: temporarily sets enforcing mode

#### Context Settings and Policy

Context: label that can be applied to different objects like files, directories, users, processes, ports

Context labels define the nature of the object. Important for stuff to be labeled correctly

#### Monitoring Current Context labels

To see the current context label settings for files and directories:

``` b
[kimchen@rhel9 ~]$ ls -Zla
total 128
drwx------.  7 kimchen kimchen unconfined_u:object_r:user_home_dir_t:s0         
-rw-r--r--.  1 kimchen kimchen unconfined_u:object_r:user_home_t:s0      
```

To see current context labels for processes:

```
[kimchen@rhel9 ~]$ ps Zaux | head
LABEL                           USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
...
system_u:system_r:kernel_t:s0   root
```

Every context label consists of three parts:
* **user**:  \_u. Not the same as Linux users
* **role**: \_r
* **type**: \_t

#### Setting Context Types

**man semanage-fcontext**

Package: 
- ` policycoreutils-python-utils`

- `semanage`: Writes the context to the policy, from which it is applied to the system
- `chcon`: Writes the context to the filesystem, not to the policy. Should be avoided

To set the context type to any directory and everything below it:

``` bash
semanage fcontext -a -t "CONTEXT_TYPE" "/DIR(/.*)?" 
```

To complete the command:

``` bash
restorecon -R -v "DIR/"
```

#### Finding Context Type

Package: 
-`selinux-policy-doc`

`man -k _selinux`

To list all context types (or only those related to a program)

``` bash
semanage fcontext -l [| grep openvpn]
```

#### Restoring Default Contexts

``` bash
restorecon
```

* New files inherit the context from the parent directory
* Copied files from other dirs are considered new files, inherit the settings from new dir
* if a file is copied or moved with -a option, the original context settings are kept

To relabel the entire file system and make sure the current context label settings are consistent with the SELinux policy, make sure the `autorelabel` file is present
`/.autorelabel`: when the server is restarted, the fs is automatically relabeled and the file is removed

Alternatively, using the `load_policy` command

```
/usr/sbin/load_policy -i
```

`restorecon -Rv /` : does the same

To restore the context type of a file:

``` bash
restorecon -v "FILE"
```
#### Managing Port Access

Adding ports to contexts does not need `restorecon`

`semanage port`

To list all labeled ports

``` bash
semanage port -l
```

To add a port to a context:

``` bash
semanage port -a -t "http_port_t" -p tcp "PORTNUMBER"
```

#### Using Boolean Settings to Modify SELinux Settings

To get a list of Booleans

``` bash
getsebool -a
```

```
[kimchen@rhel9 ~]$ getsebool -a
abrt_anon_write --> off
abrt_handle_event --> off
abrt_upload_watch_anon_write --> on
antivirus_can_scan_system --> off
antivirus_use_jit --> off
authlogin_nsswitch_use_ldap --> off
```

This shows the default and runtime boolean states with short description

``` bash
semanage boolean -l
```

```
[kimchen@rhel9 ~]$ sudo semanage boolean -l
SELinux boolean                State  Default Description

abrt_anon_write                (off  ,  off)  Allow abrt to anon write
abrt_handle_event              (off  ,  off)  Allow abrt to handle event
antivirus_can_scan_system      (off  ,  off)  Allow antivirus to can scan
```

To change a boolean. -P to make it permanent.

``` bash
setsebool [-P] "BOOLEAN" {on|off or 0|1}
```
#### Diagnosing and Addressing SELinux Policy Violations

Primary source of information
`/var/log/audit/audit.log`

SELinux messages are logged with `type=AVC` in the audit log

Sample log
```
type=AVC msg=audit(1704664062.454:385): avc:  denied  { getattr } for  pid=5114 comm="httpd" path="/web/index.html" dev="dm-0" ino=36168194 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:default_t:s0 tclass=file permissive=0
```

- `avc: denied { map }` : a map request has been denied. Some process tried to read attributes of a file and it was denied because of policy violation
- `comm="httpd"`: the command trying to issue the `getattr` request is `httpd`
- `path="/web/index.html"`: file that the process tries to access
- `scontext=system_u:system_r:httpd_t:s0`: source context. Context setting of the `httpd` command
- `tcontext=unconfined_u:object_r:default_t:s0`: target context. Context setting of the path=...

The target context needs to be relabeled for the httpd process to be able to access the file

#### Making SELinux Analyzer Easier

`sealert` provides simplified messages about SELinux events

For simplified info

``` bash
journalctl | grep sealert
```

For detailed info:

``` bash
sealert -l *6f4639d7-7cd4-42dd-8aae-24d8d9bb6bf5*
```

