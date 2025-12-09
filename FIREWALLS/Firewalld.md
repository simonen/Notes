---
tags:
  - security
  - firewalls
  - firewalld
  - centos
---

- `firewalld` is part of the `firewalld` package
- `firewalld-config` - graphical configuration tool
- `ebtables`: used for managing firewalling on Linux bridges

>!!! `iptables` and `firewalld` services must **NOT** be running together. Running iptables while `firewalld` is running messes up `firewalld` configuration

To make sure iptables (or any unwanted service) is not started by accident

``` bash
systemctl mask 'SERVICE'
```

To disable iptables, ip6tables and ebtables and prevent them from starting:

``` bash
for i in iptables ip6tables ebtables; do systemctl mask $i; done
echo "iptables ip6tables ebtables" | xargs systemctl mask -> does the same 
```

Reload the `firewalld` configuration after making permanent changes

``` bash
firewall-cmd --reload
```

See what is being blocked by the firewall

``` bash
sudo firewall-cmd --set-log-denied=[all,off,unicast,anycast]
```

```
journaltl -f
```

Firewall is blocking incoming requests on port 80 (DPT=80)

```
Oct 27 22:39:31 localhost.com kernel: filter_IN_public_REJECT: IN=ens18 OUT= MAC=bc:...:33:ff:d0:08:00 SRC=192.168.137.1 DST=192.168.137.103 DF PROTO=TCP SPT=53214 DPT=80
```
#### Network Ports and Numbering

Port reference:  
`/etc/services`

port range 0-65535

0 - 1023: a.k.a **well-known ports** for common services, SSH, IMAP, HTTP, etc
1024 - 49151: registered ports for additional services. Used for custom port assignments
49152 - 65535: ephemeral ports or private, dynamic ports. These are not listening ports. Only opened on the client machine to complete the connection.

Add non-default service ports to `/etc/services
`
```
root@server15:/home/kimchen# ss -ltna
State     Recv-Q    Send-Q          Local Address:Port           Peer Address:Port     Process
LISTEN    0         128                 127.0.0.1:631                 0.0.0.0:*
LISTEN    0         128                   0.0.0.0:22                  0.0.0.0:*
ESTAB     0         0              192.168.137.12:44664         34.107.243.93:443
ESTAB     0         64             192.168.137.12:22            192.168.137.1:53586
```

**ephemeral ports**: are not listening ports for services. Created dynamically as replies to outgoing requests

To add a port to the firewall

``` bash
firewall-cmd --add-port='port/protocol' --permanent ; firewall-cmd --reload
```

#### Direct Rules and NAT

Direct rules are stored in: `/etc/firewalld/direct.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<direct>
  <rule ipv="ipv4" table="nat" chain="POSTROUTING" priority="0">-s 10.0.6.0/24 -d 10.0.2.0/24 -o enp0s8 -j MASQUERADE</rule>
</direct>
```

Masquerade traffic. Rewrite local source addresses to that of the outgoing interface. For Dynamic IPs. 

```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s 10.0.7.0/24 -d 10.0.2.0/24 -o enp0s8 -j MASQUERADE
```

If IP is static, a slightly more efficient way to do address translation.

```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s 10.0.7.0/24 -d 10.0.2.0/24 -o enp0s8 -j SNAT --to-source 10.0.2.8
```

- `--to-source "iP"`: The source address outbound packets should have.
- `0`: Rule priority (Insert at the top)

Masquerade entire zone (no destination match)

```bash
firewall-cmd --zone='ZONE' --add-masquerade --permanent ; firewall-cmd --reload
```
##### Static DNAT (Destination NAT)

Redirect traffic to an internal host (port forwarding)

```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat PREROUTING 0 -i enp0s8 -p udp --dport 1194 -j DNAT --to-destination 10.0.7.2:1194
```

List direct rules

```bash
firewall-cmd --direct --get-all-rules
```

```
ipv4 filter FORWARD 0 -i tun0 -o tun0 -j ACCEPT
ipv4 nat POSTROUTING 0 -s 10.0.7.0/24 -d 10.0.2.0/24 -o enp0s8 -j MASQUERADE
ipv4 nat PREROUTING 0 -i enp0s8 -p udp --dport 1194 -j DNAT --to-destination 10.0.7.2:1194
```
#### firewalld Services

`Firewalld` has its own services. Not to be confused with `systemd` services
`firewalld` services serve to allow ports through the firewall. They do not change config files of other services

> `Firewalld` services are pre-defined as xml files in:
> `/usr/lib/firewalld/services` -> should not be modified
> `/etc/firewalld/services`: Custom `firewalld` services go here:

To get a list of available services

``` bash
firewall-cmd --get-services
```

To list the available services in N columns

``` bash
firewall-cmd --get-services | xargs -n "N"
```

To get info about a service

``` bash
firewall-cmd --info-service="SERVICE"
```

To see which services are currently active in the active zone, or a particular zone

``` bash
firewall-cmd --list-services [--zone="ZONE"]
```

To get an overview of the current firewall configuration

``` bash
firewall-cmd --list-all
```

To allow a service through the firewall

``` bash
firewall-cmd --add-service="SERVICE" [--zone="ZONE"]
```

To make run-time configurations permanent

``` bash
firewall-cmd --runtime-to-permanent
```

##### Create a custom service
1. Create a new xml file or copy one from `/usr/firewalld/services` over to `/etc/firewalld/services`
2. Make the configurations in the file
3. Reload `firewall-cmd`
4. `firewall-cmd --get-services` now shows the custom service
5. `firewall-cmd --add-service=CUSTOM-SERVICE --permanent` 
6. Reload `firewall-cmd`

#### firewalld Zones

**firewall zones** apply to incoming packets only by default. Good for servers with multiple network interfaces

Firewall zones are names given to various levels of trust and are applied to network interfaces. An interface can have only one zone at a time. A zone can be applied to multiple interfaces

To list the interfaces a zone is applied to

``` bash
firewall-cmd --zone="ZONE" --list-interfaces
```

To get a list of all options that can be applied to a zone

``` bash
firewall-cmd --zone="ZONE" \<TAB> \<TAB>
```

Selecting zones depends on the type of services the host is running. If none are running and it is used only for outgoing connections, it is good to use **block** or **drop** zones, which reply only to connections initiated by the host.

To list all zones and their settings:

``` bash
firewall-cmd --list-all-zones
```

Default zones

* **block**: incoming connections are denied with icmp-host-prohibited message. Internal system connections are allowed
* **drop**: incoming packets are dropped with no reply
* **block** and **drop** reply to connections initiated only from the host
* **dmz**: For computers accessible from the Internet. Allow incoming SSH connections only. Should be in a separate subnet
* **external**: for use on external networks with masquerading ( NAT ). Used on routers. Only selected networks are allowed
* **home**: only selected incoming connections are allowed
* **internal**: for use in internal networks. Only selected incoming connections are allowed
* **public**: for use in public areas. Computers on the same network are not trusted
* **trusted**: all network connections are accepted
* **work**: most computers on the same network are trusted. Selected incoming connections allowed

#### firewalld Rich Rules

> By default, `firewalld` denies any traffic that doesn't match an allow rule

* **direct rules**: allow to add hand code to **firewalld** configuration. Similar to *iptables*
* **rich rules**: allows the use of expressive language to easily define custom rules that cannot be created by default **firewalld** syntax

**Rich rules** add traffic management capability to **firewalld**. Used to create **allow/deny** rules with advanced options:

* logging
* port forwarding
* masquerading
* rate limiting
* allow/deny connections for a specific zone

##### Rich Rule Syntax

**man 5 `firewalld.richlanguage`**

* **Rule**
	* \[source] \[destination]
	* { service | port | protocol | icmp-block | masquerade | forward-port | source-port}
	* \[log] \[audit]
	* \[accept | reject | drop]

* **Ordering**
	* direct rules
	* port forwarding and masquerading rules
	* logging rules
	* allow rules
	* deny rules

A rule not matched by anything is denied, depending on the zone. Trusted zone allows packets even if no rule is matched.

`--timeout`: this option allows a rule to expire after a specified period of time. 

To block a connection from a specific IP address:

``` bash
firewall-cmd --zone-public --add-rich-rule='rule family="ipv4" source address="SOURCE_IP" port port="PORT" protocol="PROTOCOL" reject'
```

To allow incoming http connections to a specific address only

``` bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="IP_OR_NETWORK/MASK" service name="SERVICE" accept'
```

Think of this as adding a service with access control. For it to work properly, the service should not be added to services, as this will match the allow rule and let everyone on the port, regardless of the rich rule.
#### Masquerading 

Cannot be used with IPv6 firewalld

To add masquerading 

``` bash
firewall-cmd --zone-ZONE --add-masquerade
```

To add more specific masquerading

``` bash
firewall-cmd --zone="ZONE" --add-rich-rule='rule family ipv4 source address="NETWORK/MASK" masquerade' --permanent
```
#### Port Forwarding

``` bash
firewall-cmd --zone=public --add-forward-port=port="PORT":proto="PROTOCOL":toport="LOCAL_PORT":toaddr="LOCAL_IPADDRESS"
```

#### Saving and Restoring the Configuration

`firewalld` must be stopped while doing configuration save and restore with iptables

[[gitea/LINUX 1/FIREWALLS/iptables#Saving and Restoring the Configuration]]