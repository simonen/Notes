
### Common reasons to use different network namespaces

1. **Containerization / virtualization**
    - Each container (Docker, Kubernetes pod, LXC, etc.) usually gets its own network namespace.
    - This way, containers can have independent IP addresses, routing tables, and firewalls, just like mini-VMs, without interfering with the host.
        
2. **Security isolation**
    - Running untrusted applications in their own namespace prevents them from sniffing or binding to host interfaces.
    - Example: a VPN client in its own namespace can’t leak traffic to the host unintentionally.
        
3. **Testing and development**
    - You can simulate entire networks on a single machine by creating multiple namespaces and connecting them with `veth` pairs.
    - Useful for experimenting with routing, NAT, firewalls, or distributed systems without needing multiple physical machines.
        
4. **Custom routing domains**
    - Different namespaces can have different routing tables.
    - Example: You want app A to go through a VPN and app B to go directly to the internet. Putting app A in a namespace with the VPN tunnel achieves that cleanly.
        
5. **Service separation**
    - Sometimes multiple versions of the same service need to run with conflicting ports.
    - By putting them in separate namespaces, both can bind to `0.0.0.0:80` without conflict.
        
6. **Network experiments / advanced setups**
    - For example:
        - Running BGP/OSPF daemons in isolated namespaces to simulate routers.
        - Building multi-homed setups where one namespace has two uplinks and another doesn’t

#### Configure Virtual Interfaces

> `ip addr`, `ip link`, `ip netns`, `ip route` are all ephemeral and for quick testing purposes only. They live only in kernel memory. For persistent configurations using the `ip` tools, use scripts in `/etc/network/if-up.d/ (Debian)` 

A virtual interface pair in Linux acts like a virtual patch cable. Packets entering one end leave through the other.

Set up a virtual interface pair

```bash
ip link add veth0 type veth peer name veth1
```

```bash
ip link set dev veth0 up
```

```bash
ip -br link
```

```bash
veth1@veth0      UP             62:6d:79:3f:48:0d 
veth0@veth1      UP             e2:0c:36:0b:ae:fe 
```

IP addresses are assigned with `ip addr add`

```bash
ip addr add 'IP/MASK' dev 'INTERFACE'
```

#### Network Namespaces

`ip netns`

Create a network namespace using the `ip` tool

```bash
ip netns add 'NAME'
```

Namespaces 

List net namespaces

```bash
ip netns list
# ns1
# ns1 (id: 0)
```

Move one end to a different net ns

```bash
ip link set 'DEV' netns 'NAME or PID of NS process'
# ip link set veth1 netns ns1
```

```
7: veth0@if6: <....> mtu 1500 qdisc noqueue state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether e2:0c:36:0b:ae:fe link-netns ns1
```

- `link-netns ns1`:  In **Root namespace** this indicates that the other end (peer) is in a namespace named `ns1`
- `state LOWERLAYERDOWN`: The link is up but the peer in the other namespace is down.

Interfaces in other namespaces can be inspected and manipulated by executing the network commands inside those namespaces, or entering the namespace by executing a bash shell process in it.

```bash
ip netns exec 'NS' 'COMMAND'
ip netns exec 'NS' /bin/bash
```

To inspect the `veth1` interface in `ns1`

```bash
ip netns exec ns1 ip link
```

```
6: veth1@if7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 62:6d:79:3f:48:0d brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

`link-netnsid 0`: In **non-root namespace** the kernel shows number id: 0 of the namespace the interface is currently in, instead of a name.

Too see which namespace the network stack belongs to

```bash
ls -l /proc/$$/ns/net
# 1 root root 0 Aug 30 12:51 /proc/968/ns/net -> 'net:[4026532209]'
```

When assigned IP addresses, the virtual interface ends should become reachable

