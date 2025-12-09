---
tags:
  - centos
  - certificates
  - systemd
---
#### Configuring Remote Logging

`Systemd-journald` provides a `systemd-journal-remote` service that can receive journal messages from remote hosts and provide a centralized logging service. The service can act passively (listen for incoming messages) or actively (pull message from remote hosts). It uses https:// for transport by default.

Benefits of using a remote logging server:

* security
* more dedicated space and longer rotation period
* stores logs from multiple servers in one place

Packages:
`systemd-journal-remote` - on server and clients

Service configuration file: 
`/usr/lib/systemd/system/systemd-journal-remote.service`

Config files: 
- `/etc/systemd/journal-remote.conf`: (server)
- `/etc/systemd/journal-upload.conf`: (client)

#### Log Server Configuration

Put server certificates in `/etc/pki/tls/certs/journald`, mode 0755, key file must be accessible to the `systemd-journal-remote`

Permissions and ownership

```
[root@dnsserver remote]# ls -la /etc/pki/journald/
total 44
drwxr-xr-x.  2 root root                   159 Mar 25 17:43 .
-rwxr-xr-x. 1 root systemd-journal-remote 1164 Mar 24 23:24 ca.crt
-rwxr-xr-x. 1 root systemd-journal-remote 4554 Mar 24 23:24 logserver.crt
-r--r-----. 1 root systemd-journal-remote 1704 Mar 24 23:24 logserver.key
```

The key is this example is not encrypted

`/etc/systemd/journal-remote.conf`
```
[Remote]
Seal=false
SplitMode=host
ServerKeyFile=/etc/pki/journald/log-server.key
ServerCertificateFile=/etc/pki/journald/log-server.crt
TrustedCertificateFile=/etc/pki/journald/log-server.ca
```

Start the `systemd-journal-remote.socket` and `systemd-journal-remote.service`

``` bash
systemctl enable --now systemd-journal-remote.socket
```

See what port the the .socket is listening on.

`/usr/lib/systemd/system/systemd-journal-remote.service`
```
[Socket]
ListenStream=19532
```

Or 

``` bash
systemctl status systemd-journal-remote.socket
```

```
systemd-journal-remote.socket - Journal Remote Sink Socket
     Loaded: loaded (/usr/lib/systemd/system/systemd-journal-remote.socket)
     Active: active (running) since Fri 2024-10-18 16:48:19 EEST; 1h 31min ago
   Triggers: ● systemd-journal-remote.service
     Listen: [::]:19532 (Stream)
```

Configure the firewall to allow **https** service and open a custom port

``` bash
firewall-cmd --add-service=https
firewall-cmd --add-port=19532/tcp
firewall-cmd --runtime-to-permanent ; firewall-cmd --reload
```

SELinux context of the certificates files: `cert_t`

**On the clients**

`/etc/systemd/journal-upload.conf`
```
[Upload]
URL=https://<logserver_fqdn>:19532
ServerKeyFile=/etc/pki/journald/client.key
ServerCertificateFile=/etc/pki/journald/client.crt
TrustedCertificateFile=/etc/pki/journald/ca.crt
```

The `systemd-journal-upload.service` file says it needs a system user `systemd-journal-upload` to run as

`/usr/lib/systemd/system/systemd-journal-upload.service`
```
[Unit]
....
[Service]
User=systemd-journal-upload
```

Create the system user

```bash
useradd -r [-d /run/systemd] [-M] -s /usr/sbin/nologin [-U] systemd-journal-upload
```

Put server certificates in `/etc/pki/tls/certs/journald`, mode 0755, key file must be accessible to the `systemd-journal-upload`.

Permissions and ownership

```
root@server15:/etc/pki/journald# ls -la
total 28
drwxr-xr-x 2 root root                   4096 Mar 25 21:57 .
-rwxr-xr-x 1 root systemd-journal-upload 1164 Mar 25 21:38 ca.crt
-rwxr-xr-x 1 root systemd-journal-upload 4430 Mar 25 21:38 client.crt
-r--r----- 1 root systemd-journal-upload 1708 Mar 25 21:57 client.key
```

The key is this example is not encrypted

Enable and start the systemd-journal-upload.service

``` bash
systemctl enable --now systemd-journal-upload.service
```

Test the set up. Send a long message from the client

``` bash
logger "Test Message"
```

The server should now have the client's journal file

```
root@db1:/var/log/journal/remote# ll
-rw-r-----. 1 systemd-journal-remote systemd-journal-remote 83K 'remote-C=BG,ST=Sofia,O=Ohio\x20Inc,OU=IT,CN=headoffice.ohio.cc.journal'
```

To open the client's journals on the server

``` bash
journalctl [-f] --directory=/var/log/journal/remote
```

To read a journal (On the server). `-f` doesn't work with `--file=`

``` bash
journalctl --file=/var/log/remote/remote-CN=server15.journal
```
`
```
Mar 25 22:00:52 server15 root[3533]: Test Message
```

##### Systemd-Journal-Gateway HTTP Server for Journal Events

`systemd-journal-gatewayd` serves journal events over the network. Clients must connect using HTTP. The server listens on port 19531 by default. The `systemd-journal-gateway` system user must have read access to the journal directory.

- `systemd-journal-gatewayd.service`
- `systemd-journal-gatewayd.socket`
- `systemd-journal-gatewayd`

`man systemd-journal-gatewayd`

By default, the gateway will serve clients only its own runtime journal. Arguments must be passed to `systemd-journal-gatewayd` daemon to include other journals
- `-m, --merge`: Serves entries interleaved from all available journals
- `-D DIR, --directory=`: Serves the specified journal directory instead of the default runtime and system journal paths.
- `--file=GLOB`: Servers entries from the specified journal files instead of he default runtime journal. Can be specified multiple times

Arguments can be passed to the daemon by including them into the service file

`/usr/lib/systemd/system/systemd-journal-gatewayd.service`
```
[Unit]
Description=Journal Gateway Service
Requires=systemd-journal-gatewayd.socket

[Service]
DynamicUser=yes
ExecStart=/usr/lib/systemd/systemd-journal-gatewayd --merge
```

Open port 19531

``` bash
firewall-cmd --add-port=19531/tcp --permanent ; firewall-cmd --reload
```

The journal gateway now is serving http requests for journal data. The service can be reached by web browser at http://journal-gateway:19531/browse or

```bash
curl http://"journal-gateway":19531/entries[?option1&option2=value...]`
```

Some options after `entries?`
- `follow`: = `-f
- `boot`: Limit entries to the current boot
- `KEY=FIELD`: = `_COMM=sshd` for example

Return a list of values of the field present in the logs. Mimics the `journalctl -F _FIELD` 

``` bash
curl http://db1.ohio.cc:19531/fields/_HOSTNAME
```

This is particularly useful to check if the log server is correctly collecting the remote journals

```bash
db1.ohio.cc
localhost
headoffice.ohio.cc
```

To pull the journal from a gateway

``` bash
 curl --silent  'http://db1.ohio.cc:19531/entries'
```

The `follow` options behaves like `-f`

``` bash
curl --silent 'http://db1.ohio.cc:19531/entries?follow[&_COMM=sshd]'
```

To retrieve events in Journal Export Format. Similar to `-o verbose`

```bash
curl --silent -H 'Accept: application/vnd.fdo.journal' \
	'http://localhost:19531/entries'
```
