
Users a listed in: `/etc/passwd`. Groups are listed in: `/etc/group`
Hashed passwords are stored in `/etc/shadow`

`/etc/passwd` entry breakdown
```
Username:x:UID:GID:GECOS:Home Directory:Default login shell
```

`x`: Marker for password, stored in the `/etc/shadow` as a hash

`/etc/shadow` entry breakdown
```
babuna:$6$0VllUFP4$UjlNOukc8...riDJO4oaKk2uRP990T2eLQ9N38JXb/:19842:0:99999:7:::
```
`username:(password prefix)pass_hash:date pass was last changed:minimum days between pass changes:pass expiration in days:pass exp warning in days:number of days after pass expiration account will disabled:date since account has been disabled`

Password prefixes:
- `$`: Active account
- `* or !`: Password is disabled, account locked.
- `!!` : Pass never set


Use `getent` to list the contents of the respective directories

```
getent passwd; getent group
```

* human users: root / superuser or regular / unprivileged 
* system users: represent services or processes. Do not have logins or home dirs

Each user has at least one UID and GUID, is a member of a primary and a supplementary group

Commands for managing users and groups. Part of the Shadow Password Suite
**/etc/login.defs** - main conf file
* `useradd`
* `groupadd`
* `userdel`
* `groupdel`
* `usermod`

#### User UID and GID

**man 1 id**

* **Real UID / GUID**: the UID and primary GID assigned to user at creation.
* **Effective UID / GUID**: The UID used to run a process that requires different privileges than the user who launched the process
* **Saved UID / GUID**: used when a program running with elevated privileges needs to do unprivileged work temporarily; changing euid from privileged (typically 0) to unprivileged value causes the privileged value to be stored in suid

Executable files with the set uid special permission are executed always as the owner of the file, regardless of the user who is actually executing it.

EXAMPLE: 

1. Execute the `passwd` command as a regular user
2. Check out the `passwd` process information

```bash
ps -eo pid,euser,ruser,rgroup** *PASSWD_PID*
```

`2243 root     kimchen  kimchen`

The `passwd` command is run effectively by the root user (`euser`), who is the owner of the exe file, by real user kimchen (`ruser`), primary group kimchen (rgroup)

To get the real **UID** of a user:

``` bash
id "USER"
```
#### Creating Users

**man useradd**
**man passwd**

- `/etc/login.defs`
- `/etc/default/useradd` : `useradd` default settings
- `/etc/skel`

`/etc/passwd` structure
```
USERNAME:x:UID:GID:COMMENT:HOME_DIR:/SHELL
klientela:x:1003:1003:Account manager:/home/klientela:/bin/bash
```

To list `useradd` default settings. Displays the contents of `/etc/default/useradd`

``` bash
useradd -D
```

To list available shells

``` bash
cat /etc/shells
chsh -l # CentOS
```

Create a human user

``` bash
useradd "USER"; 
```

If the Linux system does not create home dir and primary group for the user automatically:

```bash
useradd -mU 'USER'
```

Assign or change passwords

``` bash
passwd "USER"
```

To read the password from STDIN

```bash
echo 'PASSWORD' | passwd --stdin 'USER'
```

To force the user to change his password at first login (expire the password)
``` bash
passwd -e 'USER'
```

Create and add the user to existing group(s)

``` bash
useradd -G "GROUP1","GROUP2","GROUP3" -c "FULL NAME,,,," "USER"
```

##### Create a System User with useradd

``` bash
useradd -rs /sbin/nologin "SERVICE_USER"
```

- `-r`: Create system user with real ID in the correct numerical range (as defined in `/etc/login/defs`) for system users
- `-s`: login shell
- `USERGROUPS_ENAB yes`: if set to 'no' default user group is users(gid=100)

#### Password Aging

man chage

`/etc/login.defs`

Set the password to expire after 30 days

``` bash
chage -M 30 "USER"
```

#### Customizing the Documents, Music, Downloads Directories

These directories are set up from `/etc/xdg/user-dirs.defaults`
Include KEY=VALUE pairs. Only the VALUE can be changed, or dirs can be excluded overall

```
DESKTOP=Desktop
```

For per-user customization the conf file is `~/.config/user-dirs.dirs`

```
XDG_DESKTOP_DIR="$HOME/Desktop"
```

To apply changes. Custom dirs must exist before applying the changes. Apply automatically on login

```bash
xdg-user-dirs-update --set DESKTOP $HOME/'NEW_NAME'
```

Icons should change if successful

To revert back to defaults

```bash
xdg-user-dirs-update --force
```

For dirs outside of $HOME, use symbolic links

```bash
ln -s /'OUTSIDE_DIR' ~/OUTSIDE_DIR
```

`~/.config/user-dirs.dirs`
```
XDG_MUSIC_DIR="$HOME/OUTSIDE_DIR"
```

Log out and log in to apply changes automatically

#### Checking Password File Integrity

- `pwck` : Check the integrity of the **shadow** file
- `grpck` : checks the integrity of the **gshadow** file

#### Disabling User Accounts

**man passwd**
**man chage**
**man usermod**

Disable the password only.  This adds a !! to the hash in **/etc/shadow** thus invalidating it.
The user can still log in using PKI. To lock a user's account

``` bash
passwd -l "USER"
```

or 

Replace the 'x' with an '\*' in the password field in `/etc/passwd`

To completely disable a user account

``` bash
usermod -e 0 "USER"
chage -E 0 "USER"
```

or set the login to /bin/false or /sbin/nologin. /sbin/nologin also logs login attempts to syslog.

``` bash
usermod -s /bin/false
usermod -s /sbin/nologin
```

Enable (unlock) a user account

``` bash
passwd -u "USER"
usermod -e -1 "USER"
chage -E -1 "USER"
```

To list an account's expiration info

``` bash
chage -l "USER"
```

#### Deleting User Accounts

**man userdel**
**man deluser** 

The **userdel** command will not delete a currently logged in user. Files, owned by a deleted user will now be owned by its former UID. A new user with the same UID will taken ownership of those files.

Deletes the user entry in the **/etc/passwd**, user's primary group in **/etc/group** and group memberships in the shadow files

```bash
userdel "USER"
```

Deletes the home directory with its contents and the mail spool as well

``` bash
userdel -r "USER"
```

Do not use the -r switch for service accounts as deleting their home dir can destroy the system.
#### Finding and Managing All Files for a User

man find

To locate all files belonging to a UID, username or group name

``` bash
find / -uid "UID"
find / -user "USER/UID"
find / -group "GROUP/GID"
find / -user "UID" -o -group "GID"
```

Find all files belonging to non-existent user

``` bash
find / -nouser
```

Perform an action on files returned by **find**. In this case move them over to another loc

```bash
find / -user 'USER' -exec mv {} 'SOME_DIR' \; 
```
