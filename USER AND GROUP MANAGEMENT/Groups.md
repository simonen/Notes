
`/etc/groups`

`GROUP:X:GID:MEMBER1,MEMBER2,MEMBERN`
`cigani:x:1004:kimchen,gligana`

- primary groups
- supplementary groups

Users can be added directly in the groups file after the desired group, comma-delimited 

Users must be members of a primary group. When a file is created the user's primary group becomes the group owner of that file.

Users' primary group is defined in the **/etc/passwd** file
The group itself is in the **/etc/group** file

A user has access to files its primary or secondary groups it belongs to are group owners of that file. Particularly useful with file share servers

> To Do: Add a group pass and observe the gshadow file
#### Creating Groups

man `groupadd`

Groups can be created by editing the `/etc/group` config file using the `vigr` command
or the `groupadd` command

Create a group

``` bash
groupadd "GROUP"
```

- `-g`: specify group ID. Optional
- `-r`: create a system group with the correct GID range
#### Adding Users to Groups with `usermod`

> Logged-in users must log out and log in for changes to take effect

Add user to a group, overriding his existing memberships!!!

``` bash
usermod -G "GROUP" "USER"
```

To append a user to a group or comma-delimited list of groups

```bash
usermod -aG "GROUP1","GROUP2" "USER"
```

To change the name or group ID of a group:

``` bash
groupmod [options] "GROUP"
```

See which groups a user is a member of

``` bash
groups "USER"
```

#### Listing Group Members

``` bash
lid -g "GROUP"
groupmems -g "GROUP" -l
getent group "GROUP"
cat /etc/group | grep "GROUP"
```

#### Group Administration with `gpasswd`

man `gpasswd`

The `gpasswd` command is used to administer `/etc/group`, and `/etc/gshadow`. Every group can have administrators, members and a password.