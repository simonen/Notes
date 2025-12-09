**man sshd_config**

* disable root login
* disable password authentication
* non-default ssh port
* specific users login

Create drop-in files in `/etc/ssh/sshd_conf.d/*.conf` rather than editing the original `sshd_conf` file

`StrictModes yes | no`: checks if file ownerships and permissions of the user's files and homedir are correct before accepting logins. Yes by default

The host keys must be owned by root. The host private keys are 600, the public keys are 644: world-readable

```
-rw-------.   1 root root    994 Mar 17 22:55 ssh_host_key
-rw-------.   1 root root   1675 Mar 17 22:55 ssh_host_rsa_key
-rw-------.   1 root root    227 Mar 17 22:55 ssh_host_ecdsa_key
-rw-r--r--.   1 root root    191 Mar 17 22:55 ssh_host_ecdsa_key.pub
-rw-------.   1 root root    419 Mar 17 22:55 ssh_host_ed25519_key
-rw-r--r--.   1 root root    111 Mar 17 22:55 ssh_host_ed25519_key.pub
```

##### Disable Root Login

By default, on most systems SSH root login is disabled. The parameter controlling root login is `PermitRootLogin` in the `/etc/sshd_config` config file. 

Options:
* yes
* prohibit-password: allows public key authentication only
* forced-commands-only
* no

##### Disable Password Authentication

`PasswordAuthentication no`

#### Add User Rules

`sshd_config.d/rule`
```
Match User *,!kimchen
    ForceCommand /bin/sh -c 'printf "You are not authorized to SSH into this server.\n"; sleep 20; exit 1'
```

This will print the message to any user that is NOT `kimchen` and close the session in 20 seconds.

`/bin/sh -c COMMAND`: Executes the command in a proper shell environment 
##### Change Default Port

Look for unused ports in `/etc/services`

``` bash
getent services | grep "PORT"
```

Add the `sshd NEW_PORT` to `/etc/services`

Change the SELinux label on the non-default port:

``` bash
semanage -a -t ssh_port_t -p tcp "PORT_NUMBER"
```

Firewall

``` bash
firewall-cmd --add-port="PORT_NUMBER"/tcp --permanent; firewall-cmd --reload
```

#### Changing the Passphrase

``` bash
ssh-keygen -p -i "PRIVATE_KEY"
```

#### SSH Agents

SSH agents keep unlocked private keys in memory for easy authentication without typing the passphrase every time when connecting to a remote host. The passphrase is only typed once. Works while the user session is active.

Add the ssh agent to the current shell

``` bash
ssh-agent /bin/bash
```

Cache the passphrase of the private key

``` bash
ssh-add "PRIVATE_KEY"
```

SSH knows about the agent when two environment variables are set

```
[root@delphos ssh]# ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-UJ826o6EwMHW/agent.21480; export SSH_AUTH_SOCK;
SSH_AGENT_PID=21481; export SSH_AGENT_PID;
```

Start the ssh-agent

``` bash
eval $(ssh-agent)
```

The ssh agent should now have a running process

```
echo $SSH_AGENT_PID
1937
```

Add the private key and enter its passphrase

``` bash
ssh-add "/PATH_TO_PRIVATE_KEY"
```

To stop the ssh agent when no longer needed

``` bash
eval $(ssh-agent -k)
```

##### Automatic Passphrase Management with Keychain 

> Doesnt Work

Package: 
`keychain`

`keychain` works like the ssh-agent for storing private key passphrases but it caches the passphrase until system shutdown

This goes in the `/home/.bash_profile`. Applies automatically on login

`eval keychain --eval ~/.ssh/PRIVATE_KEY`

Keychain will prompt for the passphrase once at first login

```
 * keychain 2.8.5 ~ http://www.funtoo.org
 * Starting ssh-agent...
SSH_AUTH_SOCK=/tmp/ssh-XXXXXXNgVj1u/agent.3171; export SSH_AUTH_SOCK;
SSH_AGENT_PID=3172; export SSH_AGENT_PID;

 * Adding 1 ssh key(s): /home/kimchen/.ssh/kimkey
Enter passphrase for /home/kimchen/.ssh/kimkey:
 * ssh-add: Identities added: /home/kimchen/.ssh/kimkey
```
##### Using Keychain to Make Passphrases Available to cron

Create the script that should be scheduled for execution with cron. Example with **rsync**

$ **chmod +x** *SCRIPT*

```
#!/bin/bash
source $HOME/.keychain/${HOSTNAME}-sh
/usr/bin/rsync -av $HOME/FILES_TO_BACKUP REMOTE_SERVER
```

The cronjob 

```
30 15 * * * /PATH_TO_SCRIPT
```

##### Limiting User Access

`sshd_config` parameters:
- `AllowUsers USER1@HOST USER2@HOST` 
- `DenyUsers *USER1@HOST USER2@HOST`
- `AllowGroups *GROUP1 GROUP2`
- `DenyGroups GROUP1 GROUP2` 
##### Session Options

- `GSSAPIAuthentication`: used with Kerberos. Otherwise should be switched off
- `UseDNS`
- `MaxSessions`: 10 by default

##### Connection KeepAlive Options

On the server
* `CPKeepAlive`
* `ClientAliveInterval`
* `ClientAliveCountMax`

On the client in `/etc/ssh/ssh_config`:
* `ServerAliveInterval`
* `ServerAliveCountMax`

Limit the length of time the server waits for a user to log in and complete the connection. The default is 2m:
`LoginGraceTime 90`

#### SELinux to allow port change

After modifying a port, SELinux needs to be configured to allow for the change
SELinux security labels on ports prevent services from accessing port they are not allowed to.

If not running a web server, SSH on port 443 can be configured

To avoid getting locked out of the server while changing ssh ports, open two sessions. Sessions are not automatically closed when successfully restarting the `sshd`.

To list all ports that have security labels

```bash
semanage port -l
```

To modify an existing port:

```bash
semanage port -m
```

To label port 2020 for access via SSH

```bash
semanage port -a -t ssh_port_t -p tcp 2020
```

Port should also be opened by the firewall

```bash
firewall-cmd --add-port=2020/tcp; $ firewall-cmd --add-port=2020/tcp --permanent
```