
Check an FQDN

```bash
hostname -f
```

Correct `hosts` order

`/etc/hosts`
```
192.168.99.100 FQDN HOSTNAME
```

Commands for validation network config on modern Linux

```bash
ip addr
ip route
ip link
```

The `ifconfig` utility is outdated and should not be used in modern Linux distros

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:0a:9c:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.137.29/24 brd 192.168.137.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever

```

- `mtu`: maximum transmission unit. Max packet size
- `qdisc`: queue discipline
- `fq_codel`: qdisc type
- `qlen`: ethernet buffer transmit queue length or simply put - max speed 1000 mbit.

Show neighbors 

```bash
ip nei
```
#### Default Route

To show IP routes:

``` bash
ip route
```

```[root@server1 ~]# ip route show
default via 192.168.4.2 dev ens33 proto dhcp metric 100
192.168.4.0/24 dev ens33 proto kernel scope link src 192.168.4.210
metric 100
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1
linkdown
```

- `default`: gateway goes through (via) IP address 192.168.4.2 (router IP) on device ens33
- `metric` - when multiple routes are available with the same destination, the lowest value is used.
- **192.168.4.0/24**: local connected network
- `src`: the preferred source address when sending to the destinations covered by the route prefix
- `dev`: output device
- `proto`:
	* `kernel` - connected
	* `boot` - DHCP obtained
	* `static` - manually added

Dump the contents of the routing table

``` bash
ip route show table all
```

To add a default route

``` bash
ip route add default via "ROUTER_IP" dev "INTERFACE"
```

To temporarily set a device up or down

``` bash
ip link set dev "DEVICE" { up | down }
```

To display IP addresses on one line

``` bash
ip -oneline -family inet address show
```

To temporarily assign | remove an IPv4 address:

``` bash
$ ip address add | del "IP_ADDRESS/MASK" dev "DEVICE"
```

Check driver information of an interface

``` bash
ethtool "INTERFACE"
```

#### Ports and Services

man ss

To verify listening ports on the host:

```
[kimchen@rhel71 ~]$ ss -lt
State       Recv-Q Send-Q        Local Address:Port     Peer Address:Port
LISTEN      0      5             192.168.122.1:domain            *:*
LISTEN      0      128                       *:ssh               *:*
LISTEN      0      128               127.0.0.1:ipp               *:*
LISTEN      0      100               127.0.0.1:smtp              *:*
LISTEN      0      128                       *:sunrpc            *:*
LISTEN      0      128                    [::]:ssh            [::]:*
LISTEN      0      128                   [::1]:ipp            [::]:*
```

Ports listening on the loopback address 127.0.0.1 means that they can be accessed only locally, cannot be reached externally.

Options

-l: show interfaces in LISTEN state
-t: TCP
-u: UDP
-4: ipv4 only
-n: shows port numbers of protocols
-a: all states
