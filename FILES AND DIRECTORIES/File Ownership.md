
Ownerships of files and directories are assigned at their creation. The file or directory **user owner** gets the name of the user creating it, the user's **primary group** becomes the file or dir **group owner**.

When a file or dir is accessed the shell checks for permissions in the following order

1. Checks if the user owner is the user
2. Checks if the user is a member of the file or dir group owner
3. Checks if the user is a member of Others

These checks work on the principle of ACLs. If a condition from the list above is met, the shell looks no further. 
The shell then grants the assigned permissions to the user request access to the resource

Display permissions. r-x permissions are required.

``` bash
ls -l
stat "FILE/DIR"
```

Look for files that are owned by a specific users

```bash
find / -user 'USER'
```

Look for files that are owned by a specific group

```bash
find / -group 'GROUP'
```

#### Default Ownership

By default, the user's primary group is set at user's creation. This primary group becomes the file or dir group owner by default for each file or dir created by the user.

To change the primary group of a user:

``` bash
newgrp "GROUP"
```

This opens a subshell and the effect is temporary until the subshell is closed.
New files and older are now **group owned** by the new **primary group**. The user must be either a member of that group or the group must have a group password.

#### Changing User Ownership

Changing file or dir user ownership requires elevated privileges

Change user ownership

``` bash
chown "USER" "FILE/DIR"
```

Change ownership recursively

``` bash
chown -R "USER" "FILE/DIR"
```

#### Changing Group Ownership

Changing group ownership requires the user changing it to be member of that group and owner of the file.

Changing with the `chown` command. Include . or : in front of the group name

Set the user and group ownership to a file or dir

``` bash
chown "USER":"GROUP" "FILE/DIR"
```

Change just the group owner without setting a user owner

``` bash
chown :"GROUP" "FILE/DIR"
```

Transfer ownership of files from one user to another. Username or UID

```bash
chown -Rv --from 'ORIGINAL_OWNER' 'NEW_OWNER'
```

```bash
find / -user 'ORIGINAL_OWNER' -exec chown -v 'NEW_OWNER' {} \;
```

Changing group ownership with the `chgrp` command

``` bash
chgrp [OPTION] "GROUP" "FILE/DIR"
```
