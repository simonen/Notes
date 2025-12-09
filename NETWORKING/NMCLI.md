---
tags:
  - centos
  - networking
---
Network configuration files (profiles) in CentOS
- `/etc/sysconfig/network-scripts` : CentOS 7
- `/etc/NetworkManager/system-connections` : CentOS 8 and up

For more info on all network options
`/usr/share/doc/initscripts-\<version>/sysconfig.txt`

To FILL IN: NetworkManager - when it is used, relation to /etc/resolv.conf

NMCLI HELP:
`man 5 nm-settings`
`man nmcli-examples`

General dev info

``` bash
nmcli
```

List general networking permissions

``` bash
nmcli general permissions
```

```
[root@localhost ~]# nmcli general permissions
PERMISSION                                                        VALUE
org.freedesktop.NetworkManager.checkpoint-rollback                yes
org.freedesktop.NetworkManager.enable-disable-connectivity-check  yes
org.freedesktop.NetworkManager.enable-disable-network             yes
[...]
```

Get connection | dev help

```
nmcli con | dev help
```

Show a list of all connections

``` bash
nmcli con show 
```

Show detailed info about a connection

``` bash
nmcli con show "CONN"
```

```
[root@localhost ~]# nmcli con sh enp0s3
connection.id:                          enp0s3
connection.uuid:                        b0e51bf1-cc11-4ddd-81f9-5849df033521
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              enp0s3
connection.autoconnect:                 yes
```

Properties that can be empty values ( -- ) can be reset by entering '' (empty string)  value

Display device connection state

``` bash
nmcli dev
```

Show an overview of the specific device

``` bash
nmcli dev show "DEVICE"
```

nmcli works with tab completion
Get available option for nmcli by double tapping TAB

#### Add a Network Connection

Get connection type help

``` bash
nmcli con add help
```

Add a new dhcp configured connection

``` bash
nmcli con add con-name "CONN" type ethernet ifname "DEV" ipv4.method auto
```

Add a new static connection

``` bash
nmcli con add con-name "CONN" ifname "DEV" autoconnect no type ethernet ip4 "IP_ADDRESS/MASK" gw4 "GATEWAY"
```

Activate / Deactivate a connection. Done after making changes to a conn.

``` bash
nmcli con {up | down} "CONN"
```

#### Modify Connection Settings

``` bash
nmcli con mod "CONN" OPTIONS
``` 

Disable | enable autoconnect

``` bash
nmcli con mod "CONN" connection.autoconnect {yes | no}
```

Add DNS server to a connection

``` bash
nmcli con mod "CONN" ipv4.dns "DNS_SERVER_ADDR"
```

Add a second address (DNS)

``` bash
nmcli con mod "CONN" +ipv4.dns "IP/MASK"
```

To ignore getting DNS server IP from a DHCP server, if configured use:

``` bash
nmcli con mod "CONN" ipv4.ignore-auto-dns yes
```

Restart the connection after modifications

#### NetworkManager Connection Conf file

- `/etc/NetworkManager/system-connections`
- `/usr/lib/NetworkManager/conf.d/10-globally-managed-devices.conf`: controls unmanaged devices

Naming convention
connection_name.nmconnection

`enp0s3.nmconnection file`
```
[connection]
id=enp0s3
uuid=9bacc790-de41-4f40-8a3d-0015fbbc682d
type=ethernet
autoconnect=false
interface-name=enp0s3
timestamp=1702828728

[ethernet]
mac-address=08:00:27:3D:B8:50

[ipv4]
address1=192.168.134.9/24,192.168.137.1
method=manual
dns=192.168.137.1;8.8.8.8;
[ipv6]
addr-gen-mode=default
method=link-local

[proxy]
```


#### NMTUI

nmtui is a text-based connection editor

$ nmtui