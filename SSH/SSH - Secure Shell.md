
Provides remote connection to a server running `sshd` daemon*, using an ssh client

**man sshd**

Config files:
- `/etc/ssh/ssh_config`: client
- `/etc/ssh/sshd_config`: daemon

**host keys**: authenticate hosts
**public keys**: authenticate users

Transmissions are encrypted with the public key and decrypted with the private key

Test the **ssh_config** for syntax errors

``` bash
sshd -t
```

To display the active SSH sessions on a host

``` bash
echo $SSH_CLIENT
```

```
192.168.137.1 49926 22
CLIENT_IP CLIENT_PORT SERVER_PORT
```

Remote servers' public keys are stored in `~/.ssh/known_hosts`

To remove a server key of a remote host

```bash
ssh-keygen -R "REMOTE_HOST"
```

#### Listing Supported Encryption Algorithms

**man ssh**
The supported encryption algorithms can be found under the -Q options

To list supported key types:

``` bash
ssh -Q key
```

To list supported ciphers

``` bash
ssh -Q cipher
```

Gather the public SSH host keys from a server

```bash
ssh-keyscan [KEY TYPE] 'SERVER'
```

Types:

`rsa`, `ecdsa`...
#### SCP - Securely Copying Files 

``` bash
scp "SOURCE" "DESTINATION"
```

**Examples:** 

Copy (pull) a file from a remote host to local computer home dir

``` bash
scp "REMOTE_SOURCE" "LOCAL_DESTINATION"
```

Copy (push) a file from local computer to remote server

``` bash
scp "LOCAL_SOURCE" "REMOTE_DESTINATION"
```

- `-r`: copies the entire subdirectory structure
- `-P`: (uppercase) to specify non-default SSH port

#### Mounting Remote Filesystems with SSHFS

Packages:
`sshfs` `fuse`  - Filesystem in User space

The remote filesystems is mounted in a local directory with write permissions

``` bash
sshfs "USER@REMOTEHOST":"DIR" "/MOUNTPOINT"
```

To list **sshfs** mounts

``` bash
findmnt | grep sshfs
```

```
└─/home/kimchen/remotefs              kimchen@server23.example.com: fuse.sshfs rw,nosuid,nodev,relatime,user_id=1000,group_id=1000
```

Options

`-o reconnect`: Tells `sshfs` to reconnect if the connection is interrupted 

To safely unmount an SSH mounted fs

``` bash
fusermount -u "MOUNTPOINT"
```

#### SFTP - Securely Transferring Files

Secure FTP

`sshd` service is required to be running on the remote host

Uses the FTP protocol to securely transfer file.
- `put` - upload
- `get` - download

Current working directory is used to download to and upload from files

```bash
sftp 'user@remotehost'
sftp>** 
```

- `lpwd` - local current working directory
- `lcd` - change local working directory
- `pwd` - print working directory on the remote host
- `lls` - list local directory contents

**$ put / get** *source destination*

#### Using Graphical Applications in SSH environment

The remote host must have an **X** server running to allow graphical apps to be displayed over SSH
The remote host must be allowed to display screens on the local host

**-Y** allows the remote host to display screens
**$ ssh -Y** *username@remotehost *

**ForwardX11 yes**  in the config file enables remote graphical display by default