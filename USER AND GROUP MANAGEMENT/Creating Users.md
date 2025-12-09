Add safely a user with the `useradd` utility

``` bash
useradd "USER"
```

Delete a user
``` bash
userdel "USER"
```

Delete a user with along with its environment

``` bash
userdel -r "USER"
```

> [!NOTE]
> Adding users by editing the `passwd` and `shadow` files is **Not recommended**. Could mess up login to everyone.

Edit the `/etc/passwd` file

```bash
vipw 
```

Edit the `/etc/shadow` file

```bash
vipw -s
```

Edit the `/etc/groups` file

```bash
vigr
```

Better to use `usermod` and `groupmod` for modifications

Create a user and add it to the specified supplementary groups

``` bash
useradd -m -u UID -G "GROUP1","GROUPN" "USER"
```

- `-m`: Create user home directory
- `-u`: Specify user ID
- `-G`: Add user to secondary groups

Configuration files for accounts
`/etc/passwd`

Configuration file for groups
`/etc/groups`

Configuration file for account passwords
`/etc/shadow`

Technically, a user can be created by editing the previous three files.
#### Home Directories

`/etc/skel`

When a user is created, its own **home directory** is copied from `/etc/skel`. The skeleton directory can be modified for customized user home environments.

#### Default Shell

- `/bin/bash` - The default shell for regular users
- `/usr/sbin/nologin` - For system accounts that do not require interactive shell

Create a system account that cannot login

``` bash
useradd -s /sbin/nologin "USER"
```

Change the default shell of a user to regular **/bin/bash**

``` bash
usermod -s /bin/bash "USER"
```

#### Managing User Properties

man `usermod`

Modifying user properties
`usermod`

#### Configuration Files for User Management

When using `useradd` command
`/etc/defaults/useradd`

Important variables in the `/etc/login.defs` file

`ENV_PATH`: Defines the $PATH variable

#### Password Properties

Setting or changing user passwords
`passwd`

Control the password expiration period
`chage`

Set user's password to expire on December 31st 2025

```bash
chage -E 2025-12-31 'USER'
```

#### Creating a User Environment

Files used to construct the user env and read in the following order upon login:

- `/etc/profile`: Used for default settings for all users when starting a login shell
- `/etc/bashrc`: Used to define defaults for all users when starting a subshell
- `~/.profile`: Specific settings for one user applied when starting a login shell
- `~/.bashrc`: Specific settings for one user applied when starting a subshell

If the same variable occurs more than once, the last one read wins.

