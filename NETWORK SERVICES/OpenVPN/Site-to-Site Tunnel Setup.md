#### Basic Site-to-Site Tunnel

Server1
IP: 192.168.137.16
LAN: 10.0.6.2/24

Server2
IP: 192.168.137.10
LAN: 10.0.7.2/24

`server1`
```
dev tun
proto udp

local 192.168.137.16
remote 192.168.137.10

ifconfig 10.200.0.1 10.200.0.2
route 10.0.7.0 255.255.255.0

user nobody
group nobody

persist-tun
persist-key
keepalive 10 60
ping-timer-rem

verb 3
daemon
log-append /tmp/openvpn.log
```

`server2`
```
dev tun
proto udp

local 192.168.137.10
remote 192.168.137.16

ifconfig 10.200.0.2 10.200.0.1
route 10.0.6.0 255.255.255.0

user nobody
group nobody

persist-tun
persist-key
keepalive 10 60
ping-timer-rem

verb 3
daemon
log-append /tmp/openvpn.log
```

Now both VPN ends have the routes to the respective remote LANs and can ping the appropriate local network addresses

`server1`
```
10.0.7.0/24 via 10.200.0.2 dev tun0
10.200.0.2 dev tun0 proto kernel scope link src 10.200.0.1
```

`server2`
```
10.0.6.0/24 via 10.200.0.1 dev tun0
10.200.0.1 dev tun0 proto kernel scope link src 10.200.0.2
```

> The hosts on the 10.0.X.0/24 networks still have to have routes to the tunnel added manually

> TLS must be configured for encryption in production environments!

#### Three-way Routing

A three-way tunnel gives redundancy to a network.
##### Ports

> By default OpenVPN clients use port 1194 as source port as well, thus, running server and client on the same host simultaneously will cause conflicts. Use either different ports on the servers, or different local ports on the clients explicitly or `nobind`.

`default client`
```
UDP link local (bound): [AF_INET][undef]:1194
UDP link remote: [AF_INET]192.168.137.16:1194
```

Source and destination ports are the same (1194)

To explicitly set ports

`client`
```
lport 1194 (default)
lport 1195 (explicitly assigns port 1195 as local/source port)
lport 0 (same as nobind, random ephemeral port)
nobind (random ephemeral port)
rport 1194 (remote (server) port, default) 
```

Use `nobind` or `lport 0` if running server and client on the same host to assign ephemeral ports on the client.

`nobind or lport 0`
```
UDP link local: (not bound)
Local Address:Port    Process
0.0.0.0:58459         users:(("openvpn",pid=38676,fd=4))
```

SiteA (Headoffice):
VPN Server A-B: 10.200.0.1
VPN Client A-C: 10.200.0.4
LAN: 10.0.7.2/24

SiteB (Rocky):
VPN Client B-A: 10.200.0.2
VPN Server B-C 10.200.0.5
LAN: 10.0.7.3/24

SiteC (Git):
VPN Server C-A: 10.200.0.3
VPN Client C-B: 10.200.0.6
LAN: 10.0.8.4/24

`server A-B`
```
dev tun
proto udp
port 1194

ifconfig 10.200.0.1 10.200.0.2

route 10.0.6.0 255.255.255.0 vpn_gateway 5
route 10.0.8.0 255.255.255.0 vpn_gateway 10
route-delay

keepalive 10 60
verb 3
```

`server B-C`
```
dev tun
proto udp
port 1194

local 192.168.137.16
ifconfig 10.200.0.5 10.200.0.6

route 10.0.8.0 255.255.255.0 vpn_gateway 5
route 10.0.7.0 255.255.255.0 vpn_gateway 10
route-delay

keepalive 10 60

verb 3
```

`server C-A`
```
dev tun
proto udp
port 1194

ifconfig 10.200.0.3 10.200.0.4

route 10.0.7.0 255.255.255.0 vpn_gateway 5
route 10.0.6.0 255.255.255.0 vpn_gateway 10
route-delay

keepalive 10 60
verb 3
```

`client B->A`
```
dev tun
proto udp

remote 192.168.137.10
ifconfig 10.200.0.2 10.200.0.1
nobind

route 10.0.7.0 255.255.255.0 vpn_gateway 5
route 10.0.8.0 255.255.255.0 vpn_gateway 10
route-delay

keepalive 10 60
verb 3
```

`client A-C`
```
dev tun
proto udp

remote 192.168.137.18
ifconfig 10.200.0.4 10.200.0.3
nobind

route 10.0.8.0 255.255.255.0 vpn_gateway 10
route 10.0.6.0 255.255.255.0 vpn_gateway 10
route-delay

keepalive 10 60

verb 3
```

`client C-B`
```
dev tun
proto udp

remote 192.168.137.16
ifconfig 10.200.0.6 10.200.0.5
nobind

route 10.0.6.0 255.255.255.0 vpn_gateway 5
route 10.0.7.0 255.255.255.0 vpn_gateway 10
route-delay

keepalive 10 60
verb 3
```

- `vpn_gateway N`: Route metric = `ip route add ... metric N`
- `route-delay 5`: Waits 5 seconds after the tunnel comes up before applying any `route` directives. Useful when using `route` directives.

When OpenVPN establishes a tunnel, the routing might be added **too early**, before the interface is fully ready, especially on slower systems or with complex setups.
##### Inter-Tunnel Routing

Let's say we want to ping 10.0.6.2 on Site B from Site A via Site C tunnel

> Add the tunnel interfaces to the active zone in the firewall

`Site C`
```bash
firewall-cmd --zone=public --add-interface=tun{0..1} --permanent
firewall-cmd --reload
```

Enable forwarding

`Site C`
```bash
sysctl -w net.ipv4.ip_forward=1
```

Confirm forwarding

```
sysctl -a | grep net.ipv4.*.forward*
```

Routing Tables

`Site A`
```
10.0.6.0/24 via 10.200.0.3 dev tun0
10.0.7.0/24 dev enp0s8 proto kernel scope link src 10.0.7.2 metric 101
10.0.8.0/24 via 10.200.0.3 dev tun0
10.200.0.3 dev tun0 proto kernel scope link src 10.200.0.4 -> direct to Site C
```

`Site C`
```
10.0.6.0/24 via 10.200.0.5 dev tun1
10.0.8.0/24 dev enp0s8 proto kernel scope link src 10.0.8.2 metric 101
10.200.0.4 dev tun0 proto kernel scope link src 10.200.0.3 -> direct to Site A
10.200.0.5 dev tun1 proto kernel scope link src 10.200.0.6 -> direct to Site B
```

At this point the ICMP packet reaches Site B but it does not return. Add the route back to the source, i.e., 10.200.0.4 end of the Site A <-> Site C tunnel via 10.200.0.6

`Site B`
```
10.0.6.0/24 dev enp0s8 proto kernel scope link src 10.0.6.2 metric 101
10.200.0.3 via 10.200.0.6 dev tun0
10.200.0.4 via 10.200.0.6 dev tun0 -> Source address of the packet
10.200.0.6 dev tun0 proto kernel scope link src 10.200.0.5
```

Now, Site A can reach the 10.0.6.0 network behind Site B via Site C

`site A`
```
ping 10.0.6.2
PING 10.0.6.2 (10.0.6.2) 56(84) bytes of data.
64 bytes from 10.0.6.2: icmp_seq=1 ttl=63 time=2.39 ms
```

Repeat the config on each site.