
#### Routing

Routing is the process of forwarding packets between networks

Basic components needed to route:
* Routable Packet (IPv4, IPv6, etc) L3
* Network Address
* Subnet mask
* Next Hop
* Metric

If a routing table has multiple matching networks, the most specific entry is used.

Types of routes:
* `connected` - created when an IP address is assigned to an interface that is UP
* `Static` - manually created
* `Dynamic` - Routes that are learned from other routers

Routers will only use routes with reachable "next hops"
Routers will only use the best routes. More specific.
Routers will only accept routes that match its own, active protocols

When configuring static routing, every host must have a route back to the source in order to be able to return packets. 
##### Routing Components

- `IGP` - Interior Gateway Protocol  - meant to run only within an organization and propagate route changes to neighbors asap.
- `EGP` - Exterior Gateway Protocol

**Autonomous System number AS**. Issued by IANA. Collection of networks managed by a central authority. This collection has an AS.
* 16-bit numbering system
* Group of devices under a single technical administration
* Usually an IGP is considered an AS
* Ranges from 1 through 65535

**Administrative Distance (AD)**  - They way a route is learned
* Defines Trustworthiness of a routing protocol
* 8-bit numbering system
* Ranges from 0 - 255
* The lower the AD, the more believable the router is

AD Values

| Protocols   | AD Value |
| ----------- | -------- |
| Connected   | 0        |
| Static      | 1        |
| EIGRP       | 90       |
| OSPF        | 110      |
| IS-IS       | 115      |
| RIP         | 120      |
| iBGP/eBGP   | 200/20   |
| Unreachable | 255      |
**Routing Metric**
* Used for best path selection process
* IGPs use metric for shortest path calculation
* Lower value is preferred
* Depends on the routing protocol architecture
	* EIGRP metric = composite formula utilizing link bandwidth + delay
	* RIP metric = hop count
	* OSPF metric = link bandwidth (metric is called "cost")
	 
##### Routing Updates

- Incremental Update - only changes are sent in the routing update
- Full Update - RIP uses.
- Periodic Update - Sent in the specified time interval
- Triggered Update - Sent whenever change is detected

#### Static Routes and Routing Tables

> The ip command changes are not permanent and should be used only for testing and validating configurations

Show static routes

``` bash
ip route
```

Check which network interface and gateway will be used to reach the destination

``` bash
ip route get 'DESTINATION'
```

```
8.8.8.8(dest) via 192.168.137.1(gw) dev enp0s3 src 192.168.137.10 uid 1000
    cache
```

Add a default gateway

``` bash
sudo ip route add default via "GATEWAY_IP" dev "INTERFACE"
```

Add a static route

``` bash
sudo ip route add "NETWORK"/"MASK" via "GATEWAY_IP" dev "INTERFACE"
```

In Windows

```powershell
route add [-p] <destination> MASK <subnet_mask> <gateway_ip> [METRIC <metric>] IF <interface_index>
```

`-p`: Persistent

To remove a static route

```bash
sudo ip route delete "NETWORK"/"MASK" via "GATEWAY_IP" dev "INTERFACE"
```

Flush the routing table

``` bash
ip route flush table main
```

To add a route permanently using nmcli

``` bash
nmcli con mod "CONNECTION" +ipv4.routes "NETWORK/MASK NEXT_HOP"
```

#### Allow Forwarding

The router device must have the appropriate interfaces configured. This example will be routing traffic between networks 10.0.2.0/24 and 10.0.6.0/24. Proto kernel means the route is type "connected" and is added automatically when the interface has an IP address and is UP. These interfaces will be "next hops" for clients

```
[root@localhost sysctl.d]# ip route
default via 192.168.137.1 dev enp0s3 proto static metric 101
10.0.2.0/24 dev enp0s9 proto kernel scope link src 10.0.2.1 metric 103
10.0.6.0/24 dev enp0s8 proto kernel scope link src 10.0.6.1 metric 102
```

Next. forwarding must be enabled

``` bash
sysctl -a | grep "forward"
...
net.ipv4.ip_forward = 0
```

``` bash
sysctl -w net.ipv4.ip_forward=1
```

This is also temporary. To make changes permanent

``` bash
echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/99-ipforward.conf
```

Apply the changes

``` bash
sysctl --system
```

The clients must have either default gateways pointing to the router, or additional static routes added. Client 1 is directly connected to 10.0.2.0/24 but needs to reach Client 2 on 10.0.6.0/24

``` 
# CLIENT 2
default via 192.168.137.1 dev enp0s3 proto static metric 100
10.0.2.0/24 dev bond0 proto kernel scope link src 10.0.2.23 metric 300
10.0.6.0/24 via 10.0.2.1 dev bond0
```

`10.0.6.0/24 via 10.0.2.1 dev bond0` this is the static route with 10.0.2.1 being the next hop.

``` 
# CLIENT 2
default via 192.168.1.1 dev enp0s3 proto static
10.0.6.0/24 dev enp0s8 proto kernel scope link src 10.0.6.24
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.23
```

Source -> Transient Host -> Destination
#### Port Forwarding

To allow incoming connection to 192.168.0.1 on port 22

``` bash
firewall-cmd --permanent --add-forward-port=port=22:proto=tcp:toaddr=192.168.0.1:22
```
