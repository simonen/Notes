To list permissions of a file with stat

```
$ stat --format=%a:%A:%U:%G /tmp
1777:drwxrwxrwt:root:root
```

#### SUID Set User ID 

**SUID** is applicable to executables. Elevates any user running the command to the level of the owner. Placed in the "x" position of user permissions.

```-rwsr-xr-x. 1 root root 27856 Aug 12  2019 /usr/bin/passwd```

lowecase **s**: **SUID** and **execute** permissions are set
uppercase **S**: only **SUID** is set

#### SGID Set Group ID

The SGID is useful where shared spaces are used because by default users create files and directories with the group owner set to their own primary groups, therefore other users cannot automatically use them. The **SGID** appears as an **s** in the place of the **execute** permission of the **group owner**. The group owner of files and subfolders in the directory marked with SGID inherit the group owner of the parent directory.

When applied to files, it changes the effective group of the user to the group of the file owner

Placed in "x" position of group permissions

`drwxrwsr-x. 2 kimchen kimchen        6 Apr 29 18:03 documents`

lowercase s: **SGID** + **execute**
uppercase S: **SGID** only

#### STICKY BIT

Protects files against accidental deletion where multiple users have write permissions in the same directory. 

With sticky bit applied, a user can delete a file or a directory only if:

* the user has root access
* the user is the owner of the file
* the user is the owner of the directory.

The sticky bit appears as a T at the position where the execute permission of **others** is located

```drwxrwxrwt. 18 root root 4096 Dec 16 19:32 /tmp```

- `t`: Sticky bit + execute 
- `T`: Sticky bit only

#### Applying Advanced Permissions

SUID, SGID and Sticky bits are applied with the `chmod` command

- `SUID`: numeric value of 4
- `SGID`: numeric value of 2
- `Sticky bit`: numeric value of 1

The special permission goes before the regular permissions:

``` bash
chmod 1770 "FILE/DIR" or chmod +t "FILE/DIR"
chmod 2640 "FILE/DIR" or chmod g+s "FILE/DIR"
chmod 4700 "FILE" or chmod u+s "FILE"
```

#### Setting Default Permissions with umask

**man umask**

`umask` is a shell built-in

Default permissions are determined by the `umask` settings.
Files are created with 0666 permissions without umask.
Directories are created with 0777 permissions without umask.

The `umask` setting is subtracted from the default max setting to give the default permissions of files and dirs.

- Effective files permissions = default max - umask -> 0666 - 0022 = 0644 = rw- r-- r--
- Effective directory permissions = default max - umask -> 0777 - 0022 = 0755 = rwx-r-xr-x

The default `umask` setting can be set
* `/etc/login.defs`
* `~/.bashrc` - Per user by inserting `umask` *CUSTOM_UMASK*
* By using the `pam_mask` module

See the default `umask`

``` bash
umask
```

In symbolic notation

``` bash
umask -S
```

```
[kimchen@prometheus ~]$ umask -S
u=rwx,g=rwx,o=rx
```

To change it temporarily for the duration of the current session

``` bash
umask "NEW_UMASK"
```

#### User-extended Attributes

Display attributes

``` bash
lsattr
```

`----i----------- ./myfile`

The i attribute means file is immutable, i.e cannot be deleted even by root

To remove an attribute:

``` bash
chattr -i FILE
```

``` bash
chattr [+-]"attribute" "FILE"
```
