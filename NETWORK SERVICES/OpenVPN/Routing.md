
#### Allow VPN Clients to Access Internal Network Behind VPN Server

The VPN clients should be able to access the 10.0.6.0/24 network that is local to the OpenVPN server.

1. OpenVPN server:
	VPN endpoint: 10.200.0.1/24
	LAN: 10.0.6.2

2. OpenVPN client:
	VPN endpoint: 10.200.0.2/24

3. Internal host:
	LAN: 10.0.6.11

`server`
```bash
openvpn --ifconfig 10.200.0.1 10.200.0.2 --dev tun
```

`client`
```
openvpn --ifconfig 10.200.0.2 10.200.0.1 --dev tun
```
##### Static Routes

`client`
```
Network Destination        Netmask          Gateway       Interface  Metric
          10.0.6.0    255.255.255.0       10.200.0.1       10.200.0.2     26
```

The client can now ping the `10.0.6.2` address on the OpenVPN server but cannot access the `10.0.6.0/24` network yet.

Enable IPv4 forwarding on the OpenVPN server

`server`
```
sysctl -w net.ipv4.ip_forward=1
```

Check the default forward policy in `iptables`

`server`
```bash
iptables -L FORWARD
```

If it is set to ACCEPT, this is enough. There is no need to configure `iptables`.

The packets arrive but the internal host does not know where to return them yet. Configure the internal host(s) to return packets to the VPN tunnel via the OpenVPN LAN address serving as the gateway.

`internal host`
```
10.200.0.0/24 via 10.0.6.2 dev enp0s8
```

Now the client can return packets to the internal host

To add the appropriate routes when the tunnel comes up

```
openvpn --ifconfig LOCAL_VPN_ENDPOINT REMOTE_VPN_ENDPOINT --dev tun0 \
--route INTERNAL_NETWORK MASK
```

```bash
openvpn --ifconfig 10.200.0.1 10.200.0.2 --dev tun0 \
--route 10.0.6.0 255.255.255.0
```

`--route 10.0.6.0 255.255.255.0` is the same as `route add 10.0.6.0/24 via 10.200.0.2`
routes the internal network through the remote end of the VPN tunnel.

#### Routing Subnets

VPN Server -> GW-A <-NAT NETWORK-> GW-B <- VPN CLIENT 

In this client/server setup, the client will be aware of internal networks behind the VPN server and be able to reach LAN Client 10.0.7.3

VPN Server: IP 10.0.7.2/24, Tunnel end: 10.200.0.1
LAN: 10.0.7.0/24 

LAN Host:  IP 10.0.7.3/24

GW-A: Public IP: 10.0.2.7, LAN: 10.0.7.1
GW-B: Public IP: 10.0.2.8, LAN: 10.0.6.2

VPN Client1: IP 10.0.6.3. Tunnel end: 10.200.0.2

##### Gateways

Enable forwarding and Static SNAT (Source NAT). This will rewrite the source address of packets coming from `10.0.7.0/24` to `10.0.2.7`

Enable forwarding on both routers

```gw-a,b
sysctl -w net.ipv4.ip_forward=1
```

Add NAT to both routers

`gw-a`
```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s \  10.0.7.0/24 -d 10.0.2.0/24 -o enp0s8 -j SNAT --to-source 10.0.2.7
```

`gw-b
```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s \  10.0.7.0/24 -d 10.0.2.0/24 -o enp0s8 -j SNAT --to-source 10.0.2.8
```

`-j MASQUERADE`: if dynamic IP.

Enable port forwarding to redirect incoming connection on the router:1194 to the VPN server:1194

`gw-a`
```bash
firewall-cmd --permanent --direct --add-rule ipv4 nat PREROUTING 0 -i enp0s8 -p \ udp --dport 1194 -j DNAT --to-destination 10.0.7.2:1194
```

The VPN client can now connect to the VPN Server at GW-A:1194 UDP.

To route VPN traffic (i.e., coming from `10.200.0.0/24`) back to the VPN server and then through the tunnel

`gw-a`
```bash
ip route 10.200.0.0/24 via 10.0.7.2 <- VPN server
```
##### VPN Server Settings

VPN Server End: 10.200.0.1
VPN Client End: 10.200.0.2

The OpenVPN host should be able to forward packets

```bash
sysctl -w net.ipv4.ip_forward=1
```

Push the route to the internal network to connecting clients

`server.conf`
```
push "route 10.0.7.0 255.255.255.0"
```

This will result in adding `10.0.7.0/24` via `10.200.0.1` to the routing tables on connected clients.

Optionally, use `client-config-dir` to push it only to specific clients.

Filename of the client configuration must match the CN on client's certificate. 

`/etc/openvpn/server/ccd/client1`
```
iroute 10.0.6.0 255.255.255.0
```

Thе `iroute` directive tells OpenVPN that `10.0.6.0/24` is behind `client1` client, and OpenVPN will internally handle routing via the correct VPN tunnel. In this case - `10.200.0.2`. This has nothing to do with the kernel routing table.

#### Redirecting the Default Gateway

To redirect all client's traffic through the VPN tunnel

`server.conf`
```
push "redirect-gateway def1"
```

- `redirect-gateway`: Rewrites the client's default gateway so that all Internet-bound traffic goes through the VPN
- `def1`: This adds two more routes `0.0.0.0/1` and `128.0.0.0/1`. This avoids overwriting the default route, making it more compatible and reversible.

Three new bypass routes are added to preserve connectivity to the VPN server through the original route.

`client`
```
1. net_route_v4_add: 10.0.2.7/32 via 10.0.6.2 dev [NULL] table 0 metric -1
2. net_route_v4_add: 0.0.0.0/1 via 10.200.0.1 dev [NULL] table 0 metric -1
3. net_route_v4_add: 128.0.0.0/1 via 10.200.0.1 dev [NULL] table 0 metric -1
```

1. This new explicit route ensures that the client can reach the VPN server via its own default gateway. Without it, the client will use the new default gateway via `tun0` for everything, including reaching the VPN server, which would break the connection!

Instead of

`client`
```
default via 10.200.0.1 dev tun0
```

It creates

`client`
```
0.0.0.0/1 via 10.200.0.1 dev tun0
128.0.0.0/1 via 10.200.0.1 dev tun0
10.0.2.7 via 10.0.6.2 dev enp0s8 <- explicit route to VPN server
```

Which **together cover the same IP space** but let OpenVPN **leave the original default route technically in place**, allowing fallback routes to the VPN server IP.

Running a traceroute confirms that all traffic moves through the VPN tunnel

```
Host
1. 10.200.0.1 <- VPN Server End
2. 10.0.7.1 <- GW-A
3. 10.0.2.1 <- NAT Network Gateway
4. 192.168.1.1 <- Default Gateway
5. 10.107.64.2 <- First outside hop
```

##### Redirect-Gateway Parameters

- `local`: Replaces the original default route with `default via 10.200.0.1 dev tun0`. Unlike `def1` it does not create a split trick `0.0.0.0/1` + `128.0.0.0/1`. Used when server and client are on the same lan.
- `bypass-dhcp`: This flag ensures that the client does not lose access to the local DHCP server. OpenVPN will **preserve the route to the DHCP broadcast address** `255.255.255.255` via the client’s original gateway, preventing it from being routed through the VPN.
- `bypass-dns`: Preserves the original route to the local DNS server (assigned by DHCP before VPN is established for example). The flag ensures that 


#### Split-Tunneling 

Split-tunneling controls what traffic goes through the VPN tunnel and what does not.

Gives granular control over:

| Direction          | Description                                                                    |
| ------------------ | ------------------------------------------------------------------------------ |
| Into the tunnel    | Which destination networks should be reached via the VPN                       |
| Outside the tunnel | Which destination networks should bypass the VPN and use local network instead |

This routing behavior is controlled by two special directives `vpn_gateway` and `net_gateway
##### Bypass the Tunnel `net_gateway`

After `redirect-gateway def1` is used to push all traffic through the tunnel, split tunneling allows to bypass certain networks by routing them through the client's pre-VPN default gateway.

`server.conf`
```
route <network> <netmask> net_gateway <- clients original default gw
```

- `net_gateway`: Special OpenVPN variable used to refer to the **default gateway of the client system before the tunnel is established**. Allows to bypass the VPN tunnel for specific routes - by sending traffic directly to the client's original gateway instead of through the VPN tunnel.

> This adds the following route to the client. Works only on the client. 

`client`
```
route 192.168.4.0/24 via <local gateway> dev <original interface>
```

If pushed from the server:

`server.conf`
```
push "redirect-gateway def1"
push "route <network> <netmask> net_gateway"
```

All traffic uses VPN (0.0.0.0/1 + 128.0.0.0/1) but the specified network bypasses the tunnel

##### Force Local Traffic Through The VPN `vpn_gateway`

In enterprise or cloud setups, it is common for multiple networks to share overlapping subnets (like 10.0.8.0/24).

Without explicitly defining `vpn_gateway`, the OS might:
- Route traffic incorrectly to a local interface
- Leak sensitive traffic outside the tunnel

Benefits:
- Prevents route conflicts
- Ensures access to remote-only subnets over VPN, even if same subnet exists locally

When not using full-tunnel (no `redirect-gateway def1`), but wat to send particular traffic over the VPN:

`client.conf`
```
route 10.0.8.0 255.255.255.0 vpn_gateway
```

Or, if pushed from the server:

`server.conf`
```
push "route 10.0.8.0 255.255.255.0 vpn_gateway"
```

- `vpn_gateway`: Refers to the VPN server's gateway IP (Remote endpoint of the VPN tunnel). Used to explicitly route traffic through the VPN.

This will route traffic for 10.0.8.0/24 via the VPN interface, and not the local network even if there is more general route outside the tunnel. Overrides any existing routes to that network. Useful when there are overlapping subnets on both sides.

##### Avoid DNS leaks

To keep DNS private when tunneling all traffic, ensure:

`server.conf`
```
push "dhcp-option DNS <ip>"
```



