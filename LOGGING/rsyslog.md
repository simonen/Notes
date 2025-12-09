---
tags: []
---
www.rsyslog.com

The `rsyslog` utility generates, processes and stores meaningful event notification messages. The `rsyslogd` is a passive tool that listens for messages from OS and apps.

- `RELP`: Reliable Event Logging Protocol. Allows `rsyslog` to send log data across the network.
- `rsyslog` traffic is transmitted on 514/tcp

Config file: 
`/etc/rsyslog.conf`

Drop-in files: 
`/etc/rsyslog.d`

#### rsyslog.conf

The configuration file contains information about what devices and programs `rsyslog` is listening for, where that information is to be stored, and what actions are to be taken when info is received. 

[MODULES](https://www.rsyslog.com/doc/configuration/modules/index.html) add to `rsyslog` functionality

* **input modules** (`im*`):  Specify where **rsyslog** receives msgs from
* **output modules** (`om*`): By default messages are sent to destination specified in `/etc/rsyslog.conf`. By using OM messages can be sent elsewhere
* **other modules**: parser modules, msg modification modules, etc

GLOBAL DIRECTIVES
RULES

Each line in the `rsyslog.conf` is structured into two fields: a **selector** field and an **action** field. 

```
selector             action
facility.priority    write to destination
cron.=debug          -/var/log/cron.debug
```

Breakdown:
- `cron.=debug -/var/log/cron.debug`
- `=`: to specify only one priority
- `-/var/log/cron.debug`: Destination where the log is written. 
- `-`: Log writes are buffered rather than synced to disk immediately after every message.

- `*.emerg    :omusrmsg:*`
- `om`: Output module
- `im`: Input module

This will send all messages of the `emerg` priority, regardless of facility, to everyone who is logged in.

#### Facilities 

Logging facilities are categorized sources of log messages.

| Facility  | Purpose                                  |
| --------- | ---------------------------------------- |
| auth      | Security-related messages                |
| auth-priv | Access control messages                  |
| cron      | cron-related messages                    |
| daemon    | Daemons and process messages             |
| kern      | Kernel messages                          |
| lpr       | Spooling (printing) messages             |
| mail      | Mail-related messages                    |
| mark      |                                          |
| news      | Network news-related messages            |
| syslog    | syslog-related                           |
| user      | The default facility when none specified |
| uucp      |                                          |
Special facilities:
* `*`: Wildcard. All.
- `none`: Negates a facility selection

`kern.none    /var/log/messages`: Do not log kernel messages to `/var/log/messages`
#### Priorities 

Priorities are organized in an escalating scale of importance. Each priority selector applies to the priority stated and all higher priorities. 

0: Emergency (`emerg`)
	Critical system failure that requires immediate attention.
	Kernel panic or critical hardware failure
1: Alert (`alert`)
	Immediate action required.
	Corrupted database or loss of a major service like a disk failure
2: Critical (`critical`)
	Major functionality affected. Requires immediate attention
	Critical software component crashes or service crashes
3: Error (`err`)
	Non-critical errors that need attention but don't affect the overall system
	File system errors or non-critical software errors
4: Warning (`warning`)
	Conditions that could potentially cause issues, but not errors yet
	Low disk space, resource limits nearing thresholds.
5: Notice (`notice`)
	Normal but significant condition. Information on events that may require attention
6: Informational (`info`)
	General information about the system state or operations
	A successful user login, or a service start
7: Debug (`debug`)
	Detailed information for developers or system administrators for troubleshooting

`mail.err` indicates all `mail` facility messages of priority `err` and higher ( `crit, alert, and emerg`).

Additional priority modifiers "=" and "!"

- `=`: Only one priority is selected
- `!`: Except
- `cron.=crit`: Only `cron` facility messages of priority `crit` are to be selected
- `cron.!=crit`: Selects all `cron` facility messages except those of `crit` priority.
- `cron.!crit`: Only `cron` facility messages except those of `crit` **or higher priority**!

Only one priority and one priority wildcard can be listed per selector.
#### Log Destinations (Actions)

Actions tell `rsyslogd` what to do with the event notification messages it receives. 
Depending on the output module `om*` loaded, actions can be:

* Logging to a file
* Logging to a device
* Logging to a named pipe. Allows `rsyslog` to send data to another app, e.g., a log aggregation engine or database
* Logging to a specific user or the console
* Sending logs to another host
* Logging to a database table
* Executing a command
* Discarding

```
cron.err    /var/log/cron - file
auth.!=emerg    /dev/lpr1 - device
news.=notice    |/tmp/pipe - pipe
auth-priv    root,kimchen - user
```

#### Combining Multiple Selectors

Filtering works from left to right. Place broader filters at the left, narrow criteria to the right

`auth,auth-priv.crit    /var/log/auth`

`auth;auth-priv.debug;auth-priv.!=emeg   /var/log/auth`
```
auth    /var/log/auth
auth.crit    kimchen
auth.emerg    /dev/pts/0
```

The `omfile` module gives extended control over the destination log file

`etc/rsyslog.conf`
```
mail.*    action(type="omfile" dirCreateMode="0700" FileCreateMode="0644"
            File="/var/log/messages")
```

For a list of `facility.priority`:

``` bash
logger -p TAB-TAB
```

To direct debug severity messages to a specific file

``` bash
echo "*.debug /var/log/messages-debug" > /etc/rsyslog.d/debug.conf
```

To generate a debug level message and send it to the daemon facility:

``` bash
logger -p daemon.debug "Daemon Debug Message"
```

To see the message

``` bash
tail -f /var/log/messages-debug
```

```
Dec 24 20:54:06 rhel9 systemd[1]: Started Hostname Service.
Dec 24 20:54:18 rhel9 kimchen[6242]: Daemon Debug Message
```

#### Remote Logging with rsyslog

Setting up a remote logging server can be summarized as following:
* Configure `rsyslogd` to receive messages from remote servers using `imudp` and `imtcp` modules
* Configure the firewall to accept traffic from port 514
* Add rules to the client servers to send messages to the remote server

On the log server:

`/etc/rsyslog.conf`
```
module(load="imtcp" MaxSessions="500")
input(type="imtcp" port="514")
```

Restart `rsyslogd.service`

```bash
systemctl restart rsyslogd.service
```

Accept traffic from port 514

``` bash
firewall-cmd --add-port=514/tcp --permanent ; firewall-cmd --reload
```

On the client server

`/etc/rsyslog.conf`
```
*.* @@LOG-SERVER:514
```

- `*.* `: All facilities and priorities are sent to the remote server
- `@@`: Send over TCP protocol
- `@`: Send over UDP protocol

To send only `err` priority messages over TCP:

`*.err @@LOG-SERVER:514`

Sending logs to a remote server with the `omfwd` module

`/etc/rsyslog.conf`
```
mail.*    action(type="omfwd" Target="192.168.1.13" Port="10514" Protocol="tcp" NetworkNamespace="ns_eth0.0")
```

##### Remote Logging Over TLS

Packages:
`rsyslog-gnutls`

On the server

`/etc/rsyslog.d/relp.conf`
```
$ModLoad imtcp

$DefaultNetstreamDriver gtls # GnuTLS

$DefaultNetstreamDriverCAFile /etc/pki/tls/certs/cacert.pem
$DefaultNetstreamDriverCertFile /etc/pki/tls/certs/db1.ohio.cc.cert
$DefaultNetstreamDriverKeyFile /etc/pki/tls/certs/db1.ohio.cc.key

$InputTCPServerStreamDriverAuthMode x509/name
$InputTCPServerStreamDriverPermittedPeer *.ohio.cc
$InputTCPServerStreamDriverMode 1 # use TLS
$InputTCPServerRun 6514
```

`$InputTCPServerStreamDriverAuthMode x509/name`: Accept connections from all peers signed by our CA with .ohio.cc in their common name (`PermittedPeer`). `rsyslogd` on CentOS hosts is handled by the root user. On Ubuntu, the .key file must be accessible to the `rsyslog` user. 

Accept connections on 6514/tcp

``` bash
firewall-cmd --add-port=6514/tcp --permanent ; firewall-cmd --reload
```

Restart the service

``` bash
systemctl restart rsyslog.service
```

On the client

```
$DefaultNetstreamDriver gtls

$DefaultNetstreamDriverCAFile /etc/pki/tls/certs/cacert.pem
$DefaultNetstreamDriverCertFile /etc/pki/tls/certs/headoffice.ohio.cc.crt
$DefaultNetstreamDriverKeyFile /etc/pki/tls/certs/headoffice.ohio.cc.key

$ActionSendStreamDriverAuthMode x509/name
$ActionSendStreamDriverPermittedPeer db1.ohio.cc
$ActionSendStreamDriverMode 1
*.* @@db1.ohio.cc:6514
```

Again, we verify if the receiving server hostname matches the common name on its certificate.

Restart the `rsyslog` service

``` bash
systemctl restart rsyslog.service
```

Test the connection

Client

```
logger -p mail.err "This is a MAIL ERROR"
```

Server

`/var/log/maillog`
```
Oct 19 22:15:55 headoffice root[14050]: This is a MAIL ERROR
```

To pipe a log file into `rsyslog`. Now mail facility messages on the client will be piped to the `/var/log/messages` on the server.

``` bash
tail -f /var/log/maillog | logger
```

#### RELP 

Packages:
`rsyslog-relp`

RELP allows to set up a central logging server that can collect and store logs from any number of client servers. RELP is more mature than the `journald` remote logging.
If sending log messages via UDP, use RELP to mitigate potential transport losses. It is more 

Minimal server RELP configuration

[imRELP](https://www.rsyslog.com/doc/configuration/modules/imrelp.html)

``` bash
module(load="imrelp")
input(type="imrelp" port="2514" maxDataSize="10k")
```

Minimal client RELP configuration

[omRELP](https://www.rsyslog.com/doc/configuration/modules/omrelp.html)

```
module(load="omrelp")
action(type="omrelp" target="centralserv" port="2514")
or
*.* :omrelp:LOG-SERVER:2514
```

##### RELP over TLS

Server

`/etc/rsyslog.d/relp.conf`
```
module(load="imrelp" tls.tlslib="openssl")
input(type="imrelp" port="6514"
             tls="on"
             tls.cacert="/etc/pki/tls/certs/cacert.pem"
             tls.mycert="/etc/pki/tls/certs/db1.ohio.cc.cert"
             tls.myprivkey="/etc/pki/tls/certs/db1.ohio.cc.key"
             tls.authmode="name"
             tls.permittedpeer="*.ohio.cc")
```

Client

`/etc/rsyslog.d/relp.conf`
```
module(load="omrelp" tls.tlslib="openssl")
action(type="omrelp"
             target="db1.ohio.cc" port="6514" tls="on"
             tls.cacert="/etc/pki/tls/certs/cacert.pem"
             tls.mycert="/etc/pki/tls/certs/headoffice.ohio.cc.crt"
             tls.myprivkey="/etc/pki/tls/certs/headoffice.ohio.cc.key"
             tls.authmode="name"
             tls.permittedpeer="db1.ohio.cc")
```

