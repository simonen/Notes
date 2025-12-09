#### Using `su` to Be Root

**man 8 sudo**
**man 5 sudoers**

- `/etc/sudoers`
- `/etc/sudo.conf`
- `/etc/sudoers.d/`

`sudo` (super user do) - allows regular users to use administrative privileges for a specific task. The idea of **sudo** is to protect the **root** password. **sudo** users use their own passwords

Using `sudo` to get temporary privileges to execute a command can be avoided, while protecting the root password at the same time.

`sudo` command failures go to `/var/log/secure` on CentOS and `/var/log/auth.log` on Ubuntu

In order to get root privileges without switching to root using `su -` ,thus using the root password, a new bash instance with root privilege can be launched by a non-root user. 

```bash
echo "'USER' ALL=/usr/bin/bash" >> /etc/sudoers.d/'USER'
```

To switch to another user without entering their password, as if doing it as root

``` bash
sudo su "USER"
```

To execute a command on behalf of another user (Run As)

``` bash
sudo -u "USER" "COMMAND"
```

To run a command with substitute user and group ID

```
runuser [options] -u user [[--] command [argument...]]
```

Test if a user has read permission on a file

``` bash
[root@localhost] runuser -u kimchen -- sudo -l
Sorry, user kimchen may not run sudo on
```

`sudo` command-line options

| OPTION | DESCRIPTION                              |
| ------ | ---------------------------------------- |
| -l     | prints allowed and disallowed commands   |
| -b     | runs given command in the background     |
| -u     | runs command effectively as another user |

In RHEL9, during installation, if **Make User Administrator** is checked, root has no password and **sudo** can used by the privileged user

When logging as root in a graphical interface, all processes are run as root - bad practice

If not configured during installation, **sudo** can be used by a user if it is a member of the group **wheel**

- `su`: opens a subshell as another user but environment settings for that account are not set
- `su -` : opens a login shell as another user with its own environment settings and scripts 
The root can `su -` into whatever user without having to enter their password.

To safely edit the `/etc/sudoers` file

```
visudo
```

Structure of the `sudoers` file permission 

```
'USER' | %'GROUP' 'LOCAL_HOSTNAME' = ( RUN_AS_USER1 ) 'comma delimited list of commands'
```

EXAMPLE:

```
dgb     boulder = (operator) /bin/ls, (root) /bin/kill, /usr/bin/lprm
```

User `dgb` on host boulder will **Run As** user `operator` the command `/bin/ls` and `/bin/kill` and `/usr/bin/lrpm` as `root`.

- `ALL=(ALL) ALL`: Root privileges
- `%wheel ALL=(ALL) ALL`: Must be present to allow users in the `wheel` group to use the `sudo` command and ALL administrative privileges 
- `%`: Means that the group is present in the `/etc/group` file

Users can also be added in the file

`<user> ALL=(ALL) ALL`

For more fine-grained user access privileges, individual accounts can be granted access to specific commands only

Or, users can be granted sudo permissions on commands in a specific directory only

`<user> <host> = /bin/*`  

##### sudoers Aliases

The sudoers file can include command, host and user aliases ( uppercase )

- `Cmnd_Alias` `CMD_ALIAS` = `COMMAND1`, `COMMAND2`, `COMMANDN`
- `Host_Alias` `HOST_ALIAS` = `HOST1`, `HOST2`, `HOSTN`

To create user groups, not related to system groups in `/etc/group` 
- `User_Alias` `USER_ALIAS` = `USER1`, `USER2`, `USERN`

To put it all together

`USER_ALIAS HOSTALIAS` = `CMD_ALIAS1`, `CMD_ALIAS2`, `CMD_ALIASN`

Host aliases allow you to share a single `/etc/sudoers.conf` file on multiple machines. Non-existent items are ignored.

##### sudoer mailto

`sudo` can be configured to send a mail on specific occasions

`mailto admin@host on | off `  

mail options
* `mail_always`: Sends a mail every time `sudo` is used, off
* `mail_badpass`: Send mail if user running `sudo` entered incorrect pass, off
* `mail_no_user`: Sends mail if user running `sudo` does not exist in `sudoers` file, on
* `mail_no_host`: Sends mail if user exists in `sudoers` but not authorized on this host, off
* `mail_no_perms`: Sends mail if user exists in `sudoers` but not authorized to run command, off

**!!! SHELL ESCAPE! PRIVILEGE ESCALATION !!!**

`sudoers` with limited privileges can do privilege escalation by exploiting program shell escapes. Use the `NOEXEC`: tag to deny shell execution by programs

To prohibit the specific command, prepend a "!" before the command :

`sudoers`
```
USER ALL= path_to_command_1, path_to_command_2, ! command
```

To not allow a user with access to `passwd` to change **root**'s password:

`sudoers`
```
'USER' ALL=/bin/passwd, ! /bin/passwd root
```

EXAMPLE:

A file *user_useradd* file can contain just  to allow `user` to do `sudo useradd`, etc

`sudoers.d/user_useradd`
```
'USER' ALL=/sbin/useradd
```

Use `sudo` with pipes:

```bash
sudo sh -c 'COMMAND | COMMAND'
```
#### sudo Password Timeout

When `sudo` is used, the user is prompted for its password. When password is given, a token is generated that expires in 5 minutes by default. 
Can be changed to max 240 minutes.

To change the `sudo` timeout, add the following to the `sudoers` file
- `Default timestamp_timeout MINUTES`
- `0`: No password caching; will be prompted for pass every time
- `-1`: password never expires during the session

#### Individual sudoers Configurations

A drop-in file with just the modifications that would have been made to the `sudoers` file, using visudo, can be used for the same purpose in the `/etc/sudoers.d/` directory
#### Policy Kit

Most administration programs use `PolicyKit` to authenticate root users
`pkexec` can be used as `sudo` or to restore a `sudo` configuration
In case `sudo` is broken:

```bash
pkexec visudo
```
