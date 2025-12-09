
#### Files to Backup

- `/mnt`: mounpoints
- `/etc`: conf files
- `/var`: logs
- `/boot/grub`: if contains grub customizations
- `/root`: root's home dir
- `/opt`: for proprietary apps 
- `/srv`: data for servers 
- `/home`

##### Things not to Backup 

- `/dev`, `/proc/`, `/sys`: Pseudo-filesystems, existing in memory only
`/media`: For mounting removable storage

Databases have their own tools of backing up data which should be used

##### Things to Restore from Backup

- `/etc/fstab` should not be restored on new systems because devices have unique **UUID**s
- `~/.config`, `/etc`, `~/.local` configs should be restored with caution

#### Automating Simple Backups

Simple backups is using the `cp` command
If file attributes such as ownership, permissions need to be preserved, the backup should be done on Linux filesystems: `xfs`, `ext4`, `btrfs`, etc. FAT not Linux fs.

`cp` command options:
- `-a`, archive: Preserves file attributes - file ownership, permissions, timestamps, extended attributes
- `-u`, update: Overwrites existing files with newer versions only
`--bwlimit`=KILOBYTES
- `-v`: verbose
- `--parents`:  Creates missing parent directories on the destination
- `-x`, `--one-file-system`: Prevents copying files from other partitions and filesystems 

Schedule `cp` commands with `cron`

```bash
crontab -e
```

```
30 22 * * * /bin/cp -au DIR_TO_BACKUP /TARGET
```

#### rsync for Local Backups

Works similarly to the **cp** command

```bash
rsync [OPTIONS] 'SOURCE' 'DESTINATION'
```

> [!NOTE]
> SOURCE DIR TRAILING SLASH
> 
> `rsync /DIR/`: copies the contents of the dir only
> `rsync /DIR`: NO trailing slash copies the directory as well. Applies to the source dir only

Options:
- `-a`, archive
- `-z`: compress
- `-r`: recursive
- `-u`: update
- `--delete`: Files on the destination are purged if missing on the source
- `--dry-run`
- `-x`: Copy from local filesystems only. Good if network filesystems are mounted
- `-q`: Suppresses non-error messages
- `-A`: Preserves ACLs
- `-X`: Preserves `xattrs` extended attributes

#### Remote File Transfers with `rsync` Over SSH

```bash
rsync -av 'SOURCE' 'USER@REMOTE_HOST':/'DESTINATION'
```

If the destination dir does not exist, **rsync** will create it

Useful options:
- `-partial`: Preservers partially downloaded files in case of network interruption and resumes them when network is available again
- `-h`: Human readable size
- `--log-file='LOG_FILE'*`: keeps a log of transfers

**rsync** can run as a service, not requiring user authentication but does not support encryption

#### Automatic rsync Transfers with cron and SSH

For files that need root permissions. Edit the `/etc/crontab` file

```
30 22 * * * root /usr/bin/rsync -av 'SOURCE' 'USER@REMOTE':/'DESTINATION'
```

#### Excluding Files from Backup

To exclude a single file. Will exclude ALL files with the same name

```bash
rsync --exclude='FILE_TO_EXCLUDE' /SOURCE_DIR /TARGET_DIR
```

To exclude a single file from a specific dir only

```bash
rsync --exclude=DIR/FILE /SOURCE_DIR /TARGET_DIR
```

To exclude multiple files. In single quotes, separated with comma, no space

```bash
rsync --exclude={'FILE1','FILE2','FILEN'} /SOURCE_DIR /TARGET_DIR
```

To exclude dirs

```bash
rsync --exclude={'DIR1/','DIR2/','FILE1','FILE2'} /SOURCE_DIR /TARGET_DIR
```

FILE, DIR/FILE and DIR/ are called **patterns**. Trailing / means directory.

#### Including Files in Backup

Options:
- `--include=*/` : Traverse all directories in the hierarchy
- `--include=FILE` : Do not exclude FILE
- `--exclude='*'` : Exclude everything not included
- `-m`, prune : Do not transfer empty directories

Include means "do not exclude". Needs to following options:

`--include=*/` and `--exclude='*'`

All file paths are relative to the source directory

To transfer a single file 

```bash
rsync -av --include=*/ --include=ITEM --exclude='*' /SOURCE /DESTINATION
```

#### Managing Includes with a Simple Include File

Options:
- `--files-from=INCLUDE_FILE`

```
rsync -a --files-from=/tmp/foo /usr remote:/backup
```

If `/tmp/foo` contains string 'bin', the `/usr/bin` directory will be transferred to **/backup/bin** on the remote host. If it contains **bin/** (with trailing /), the directory and contents will be transfered

```
# include list
#
# Transfer the FILE only
/DIR/FILE
#Transfer the dir and contents
/DIR/
/FILE
```

#### Managing Includes and Excludes with Exclude Files

`--exclude-from=EXCLUDE_FILE`

Specific include rules must be added for all parent directories that need to be traversed

Simple exclude file. Contains only items to exclude

```
DIR/
/ITEM
```


Exclude file 2

```
# Begins with the source dir
+ /backup/
+ /FILE
# Wildcards are allowed
+ /backup/Documents/
+ /backup/Documents/*.doc
+ /backup/Music
+ /backup/Music/Rap
+ /backup/Music/Rap/Ice-Cube*
# Exclude everything else
- *
```

EXAMPLE

```bash
rsync -av --exclude-from=/home/kimchen/exc_file /home/kimchen/backup 
```

`kimchen@192.168.137.33:/home/kimchen/ --log-file=/home/kimchen/rsync.log`

This will transfer files from the /home/kimchen/backup source dir to /backup on the remote host. Will include FILE from the source dir

Lidl Bub4naP!stolet

#### rsyncd Backup Server

**man rsyncd.conf**

`/etc/rsyncd.conf`

Allows users to backup their own data without having shell accounts on the backup server
File transfers and authentication are NOT encrypted. Use OpenVPN for that

On the backup server
1. `rsyncd.service` must be running
2. Edit the `/etc/rsyncd.conf` to create an **rsync** module defining the archive

```
[backup_name] -> module name
	path = /DIR
	comment = "Server 1 Archive"
	list = yes -> no hides the module
	read only = no
	uid = 0 -> preserves permissions
	gid = 0 -> -> preserves permissions
	chroot = no
```

3. Restart the **rsyncd**
4. Test if the server is listening for incoming connections. Applies to servers and clients
	1. `rsync BACKUP_HOST::`  double colon
	2. Open port **873** in the firewall
	3. SELinux booleans. getsebool -a | grep rsync
		1. rsync_full_access --> on
	
Push files to the `rsync` server

```bash
rsync -av /DIR SERVER::MODULE_NAME
```

 Pull files from the backup server by swapping source / dest places

To view files on the backup server:

```
rsync BACKUP_SERVER::MODULE_NAME
```

#### Limiting Access to `rsyncd` Modules

On the server

Create the password file `/etc/rsyncd-users`, mode **0600**, containing USER:PASS key-pairs. 
Not related to system accounts

```
# secrets file
USER1:PASS
USER2:PASS
USERN:PASS
```

Create users folders in the backup folder

`/etc/rsyncd.conf`
```
[userX_backup]
	path = /backup/USERX
	comment = "User X Archive"
	auth users = USERX
	secrets file = /etc/rsyncd-users or whatever
	list = yes
	read only = no
	uid = 0
	gid = 0
	chroot = no
	strict mode = yes
	hosts allow = *.example.com | 192.168.11.* | 192.168.136.2 -> optional
	hosts deny = ... -> not needed if hosts allow exists
```

Restart `rsyncd.service`

Test the module

```
rsync USERX@BACKUPSERVER::USERX_MODULE
```

To push files from the client

```bash
rsync -av /FILE USERX@BACKUP_SERVER::USERX_MODULE
```

#### Message of the Day for `rsyncd`

Create a plain-text file `/etc/rsyncd-motd` and insert a greeting, tip, message, whatever

`/etc/rsyncd.conf`, at the top of the file put
```
[global]
	motd file = /etc/rsyncd-motd
```

Restart the `rsyncd` service

