
Uncomplicated Firewall. Debian

Enable / disable `ufw`

```bash
ufw enable | disable
```

Show current rules

``` bash
ufw status [numbered]
```

```
root@server22:/home/kimchen# ufw status numbered
Status: active
     To                         Action      From
     --                         ------      ----
[ 1] 22                         ALLOW IN    Anywhere
[ 2] 22 (v6)                    ALLOW IN    Anywhere (v6)                    ALLOW       Anywhere (v6)
```

To delete a rule

``` bash
ufw delete "RULE_NUMBER"
```

Allow a port

``` bash
ufw allow "PORT"/"PROTOCOL"
```

Common service names can be added too (as defined in /etc/services )

```bash
ufw allow http
```

To delete a rule

``` bash
ufw delete allow http
```

Application profiles can be defined in drop-in files in `/etc/ufw/applications.d/`. The `openssh-server` drop-in file is already present, thus allowing SSH connections, even as ufw might not be installed

`/etc/ufw/applications.d/openssh-server`
```
[OpenSSH]
title=Secure shell server, an rshd replacement
description=OpenSSH is a free implementation of the Secure Shell protocol.
ports=22/tcp
```

Namespace OpenSSH followed by title and description. Similar to `firewalld` services

To define a custom rule (application profile) with multiple ports and protocols, separated by "|"

`/etc/ufw/applications.d/webserver`
```
[webserver] 
title=Application Webserver
description=Application X Webserver
ports=80/tcp|443/tcp|8080/tcp 
```

After the app file has been created

``` bash
sudo ufw app update 'webserver'
```

To verify the new rule

```
sudo ufw app info webserver
```

To list all app profiles from the `/applications.d` dir

```
ufw app list
```

```
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
  webserver
```

To allow | deny a client access to ports specified in an app profile

``` bash
ufw { allow | deny } from "REMOTE_IP_OR_NET" to any app 'webserver'
```

To allow specific traffic to be routed from one interface to another, destined to anywhere

``` bash
sudo ufw route allow in on "IN_INT" out on "OUT_INT" to any port 22 proto tcp
```