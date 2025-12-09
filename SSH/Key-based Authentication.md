Key-based auth is more secured than password-based auth

public keys are stored in `~/.ssh/authorized_keys` on the server

#### Generating New Host Keys

Virtual machine deployments require their own unique host keys

Remove the old keys

``` bash
rm -rf /etc/ssh/ssh_host_*
```

Generate the new keys all at once

``` bash
ssh-keygen -A
```

```
ssh-keygen: generating new host keys: RSA1 RSA DSA ECDSA ED25519
```

- **DSA**: keys generally should be avoided. Remove the `ssh_host_dsa.key`
- **RSA**: keys are oldest and most compatible. Use >=2048 bits strength. 
- **ECDSA** and **ED25519**: released 2014, very strong, and computationally less expensive.

Check the current cryptographic policies applying to OpenSSH

`/etc/crypto-policies/back-ends/opensshserver.config`
```
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
MACs hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256
CASignatureAlgorithms ecdsa-sha2-nistp256,sk-ecdsa-sha2-nistp256@openssh.com
```

[[LINUX/PKI/System Crypto Policies]] 
#### Private / public key pair

private keys are stored **locally only** by default in `~/.ssh/KEY`
public keys are stored in locally in `~/.ssh/KEY.pub`
The server's host public key goes to the client `known_hosts`

Create the key pair

``` bash
ssh-keygen -C "COMMENT" -t "TYPE" -f "KEY_NAME" -b "BITS"
```

Push the public key to the remote server by specifying the corresponding private key

``` bash
ssh-copy-id -i "PRIVATE_KEY" "SSH_SERVER"
```

Another way of sending the public key using pipes

``` bash
cat "PUB_KEY".pub | ssh "USER@REMOTE_HOST" "cat >> ~/.ssh/authorized_keys"
```

**Permissions**. If the **Strict** mode is set to yes, folder and keys permissions must be exactly as:

```
~/.ssh:  rwxr-xr-x 755
authorized_keys: rw---- 600
```

> PuTTY key authentication requires the .ppk format for the private key.

`authorized_keys`
```
[OPTIONS] KEY_TYPE KEY_DATA COMMENT
```

Example

`authorized_keys`
```
ssh-ed25519 AAAAC3NzaC..1SxbDLWRG72vTRSyU9CiIXgdWvlxqTLJ eddsa-key-20251116-hohol
```

With options

`authorized_keys`
```
from="192.168.1.50",no-agent-forwarding,no-port-forwarding,command="/usr/bin/backup"
```

- `from="IP"`: Only allow connection from this IP
- `command="PATH_TO_COMMAND"`: Execute this command
#### Key Fingerprint

To verify the host key fingerprint. Both keys should produce the same fingerprint

```bash
ssh-keygen -lf /etc/ssh/ssh_host_KEY.pub
ssh-keygen -lf /etc/ssh/ssh_host_KEY
```

#### SSH Connection Profiles

In order to avoid typing long SSH commands, specifying users, keys, etc.. a per-user config file containing pre-configured connections can be created.

Per-user configuration file. Must have strict permissions

`~/.ssh/config`

```
Host LABEL
 HostName REMOTE_SERVER_HOSTNAME
 User USER
  IdentityFile PATH_TO_PRIVATE_KEY
  IdentitiesOnly yes

Host <LABEL>
 HostName ...
 User ...
  IdentityFile ...
  IdentitiesOnly ...

Host LABEL
 ...
 ...
  ...
   ...
 
```

To connect to a pre-defined host

``` bash
ssh "LABEL" same as: ssh user@hostname -i private_key
```

#### Running Commands on Remote Host Using SSH Session with ssh

``` bash
ssh "USER@REMOTEHOST" "ONE_LINE_COMMAND"
```

Use public key auth to run root commands without sudo
