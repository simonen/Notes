---
tags:
  - dns
  - dhcp
---
**`dnsmasq`** is a lightweight DHCP/caching DNS server. Provides DHCP, BOOTP, PXE, TFTP services. Dnsmasq reads entries from the **/etc/hosts** file

Package
**dnsmasq**

Service
dnsmasq

If **/etc/resolv.conf** is a symlink, means that it is run by **systemd-resolved.service**. Stop and disable it.
If it is a regular file, it means it is managed by **NetworkManager**
#### Installing dnsmasq

>The **systemd-resolved.service** should NOT be running along with **dnsmasq**.

**NetworkManager** should not be managing nameservers on a **dnsmasq** server To prevent the NetworkManager.service from rolling back changes after restarting the service:

`/etc/NetworkManager/NetworkManager.conf`
```
dns=none 
```

`/etc/resolv.conf`
```
nameserver 127.0.0.1
````

#### Configuring dnsmasq for LAN DNS

Save the original  file as backup.

`/etc/dnsmasq.conf`
```
# global options
resolv-file=/etc/resolv.conf
domain-needed # prevents dnsmasq from forwarding hostname-only queries upst
bogus-priv # blocks private bogus reverse lookups
expand-hosts #adds private domain to plain hostnames in /etc/hosts
listen-address=127.0.0.1
listen-address=<OWN_IP_ADDRESS>

# upstream name servers
server=<NS1_IP>
server=<NS2_IP>
```

Check the `dnsmasq.conf` syntax

``` bash
dnsmasq --test
```

Allow DHCP and DNS through the firewall

``` bash
firewall-cmd --add-service={dns,dhcp} --permanent ; firewall-cmd --reload
```

To query a specific **DNS** server

``` bash
dig @"DNS_SERVER" ["DOMAIN"]
```

#### DHCP with DNSMASQ

[[LINUX/NETWORK SERVICES/DHCP]]

`dhclient` : provides DHCP client daemon

>**DHCP** clients must not exist in the `/etc/hosts` file! They will be rejected a lease!

Historical DHCP connections in `/var/lib/NetworkManager`

To define a DHCP pool of IP addresses, add these lines to the 

`/etc/dnsmasq.conf`
```
# DHCP Server
dhcp-range=START_IP,END_IP,LEASETIME (ex. 12h)
dhcp-lease-max=25
```

To request an IP address over DHCP for a specific interface

``` bash
dhclient "INTERFACE"
```

To release a leased config

``` bash
dhclient -r
```

To send a lease request with verbose output

``` bash
dhclient -v
```

```
root@server15:~# dhclient -v
Internet Systems Consortium DHCP Client 4.4.3-P1

Listening on LPF/enp0s3/08:00:27:e5:2f:9d
Sending on   LPF/enp0s3/08:00:27:e5:2f:9d
Sending on   Socket/fallback
DHCPREQUEST for 192.168.137.55 on enp0s3 to 255.255.255.255 port 67
DHCPACK of 192.168.137.55 from 192.168.137.23
RTNETLINK answers: File exists
bound to 192.168.137.55 -- renewal in 16949 seconds.
```

On Windows

``` cmd
ipconfig /release
ipconfig /renew
```
#### Advertising Important Services via DHCP

Apart from offering IP addresses, **DHCP** can offer other network configuration settings, such as subnet masks, def GW, DNS servers, NTP servers, etc

In `dnsmasq.conf`, different services are specified by a code. To get a list of service codes:

```
[root@dnsserver ~]# dnsmasq --help dhcp
Known DHCP options:
  1 netmask
  2 time-offset
  3 router
  6 dns-server
  7 log-server
  9 lpr-server
 13 boot-file-size
 15 domain-name
```

`/etc/dnsmasq.conf`
```
# DHCP Server
dhcp-range=START_IP,END_IP,LEASETIME (ex. 12h)
dhcp-lease-max=25 # max num of leases at a time

# set a default gateway
dhcp-option=3,GW_IP_ADDR

# advertise a DNS server
dhcp-option=6,DNS_SERVER_IP_ADDR

# advertize an SMTP server
dhcp-option=69,SMPT_SERVER_IP_ADDR
```

#### DHCP Zones for Subnets

Different zones can be configured for different subnets. Names are arbitrary, like *zone1, zone2*

`/etc/dnsmasq.conf`
```
dhcp-range=zone1,192.168.137.100,192.168.137.200
dhcp-range=zone2,10.106.0.20,10.106.0.100
# default gateways for different zones
dhcp-options=zone1,3,192.168.137.1
dhcp-options=zone2,3,10.106.0.1
dhcp-options=ZONE_NAME,SERVICE_CODE,IP_ADDRESS
```

#### Assigning Static IP Addresses with DHCP

The static IP addresses must fall within the range of the already defined `dhcp-range`

```
dhcp-host=CLIENT_HOSTNAME_OR_MAC,IP_ADDRESS
dhcp-host=...
```

#### Configuring DHCP Clients for Automatic DNS Entry

To automatically put a client's record in the `/etc/hosts` on the `dnsmasq` server, the DHCP client must send its hostname to the server.

`/etc/dhcp/dhclient.conf`
```
send host-name = gethostname(); # working
or
send host-name = <MYHOSTNAME>
```

If the `dhclient` does not exist on the client machine, **DHCP** is handled by the `NetworkManager`

#### Managing dnsmasq Logging

`dnsmasq` uses `syslog` and `journalctl` for logging

Create a `dnsmasq` folder in `/var/log`. 

`/etc/dnsmasq.conf`
```
log-facility=/var/log/dnsmasq/dnsmasq.log
```

Restart the service

Configure log rotation. 

`/etc/logrotate.d/dnsmasq`
```
/var/log/dnsmasq/dnsmasq.log {
missingok
compress
notifempty
rotate 4
weekly
create
}
```

Test the log rotation

```bash
logrotate dnsmasq --debug
```

```
[root@dnsserver logrotate.d]# logrotate dnsmasq --debug
WARNING: logrotate in debug mode does nothing except printing debug messages!  Consider using verbose mode (-v) instead if this is not what you want.

reading config file dnsmasq
Reading state from file: /var/lib/logrotate/logrotate.status
Allocating hash table for state file, size 64 entries
Creating new state

Handling 1 logs

rotating pattern: /var/log/dnsmasq/dnsmasq.log  weekly (4 rotations)
empty log files are not rotated, old logs are removed
considering log /var/log/dnsmasq/dnsmasq.log
  Now: 2024-03-23 12:18
  Last rotated at 2024-03-23 12:00
  log does not need rotating (log has already been rotated)
```

#### Configuring Wildcard Domains

Subdomains are resolved without manually adding them to DNS. Useful when working with complex Kubernetes environment
Make sure to use address ranges that are different from the ranges on your LAN’s name server, and are available only to LAN clients

`/etc/dnsmasq.conf`
```
address=/DOMAIN/IP_ADDRESS
# address=/example.com/192.168.137.59
```

```
[root@dnsserver ~]# nslookup server5.example.com
Server:         127.0.0.1
Address:        127.0.0.1#53

Name:   server5.example.com
Address: 192.168.137.59

[root@dnsserver ~]# nslookup server33.example.com
Server:         127.0.0.1
Address:        127.0.0.1#53

Name:   server33.example.com
Address: 192.168.137.59
```
