More info in
Apress - Pro Linux System Administration 2nd Edition

Common chains in the `nat` table

- PREROUTING - Before routing decisions DNAT
- INPUT - For locally destined packets
- OUTPUT -  Locally generated packets SNAT
- POSTROUTING - After routing, before transmission SNAT/Masquerading 
- FORWARD - This rule applied to packets being routed through the system - not destined for it.


To list current rules

``` bash
sudo iptables -L
```

```
# iptables -A FORWARD -i eth1 -o eth2 -m state --state NEW,ESTAB- LISHED,RELATED -j ACCEPT

ChatGPT

The command you provided is an `iptables` rule that adds a rule to the FORWARD chain. Let's break down the components of the command:

- `iptables`: This command is used for administering the Netfilter firewall on Linux systems.
    
- `-A FORWARD`: This option specifies that the rule should be appended to the FORWARD chain. The FORWARD chain is responsible for packets that are being routed through the system.
    
- `-i eth1 -o eth2`: These options specify the network interfaces. `-i` specifies the incoming interface (`eth1` in this case), and `-o` specifies the outgoing interface (`eth2` in this case).
    
- `-m state --state NEW,ESTABLISHED,RELATED`: This option uses the `state` module to match packets based on their connection state. In this case, it matches packets that are in the NEW, ESTABLISHED, or RELATED state.
    
- `-j ACCEPT`: This specifies the action to take if a packet matches the criteria defined by the rule. In this case, it accepts (allows) packets that match the criteria.
    

Putting it all together, the command adds a rule to the firewall that allows forwarding packets from the `eth1` interface to the `eth2` interface if they are in the NEW, ESTABLISHED, or RELATED state. This rule is commonly used in scenarios where the Linux system acts as a router or gateway between two networks (`eth1` and `eth2` in this case), allowing established and related connections to pass through while also permitting new connections.
```

List the FORWARD rules

```bash
iptables -L FORWARD
```

```
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
```

The default FORWARD policy is set to ACCEPT which means that enabling forwarding via `net.ipv4.forward=1` is enough to forward packets.



#### Saving and Restoring the Configuration

This works for [[gitea/LINUX 1/FIREWALLS/Firewalld]] as well. Both services must not be running at the same time

To dump the current configuration on screen

``` bash
iptables-save
```

To save it, we can use redirection

``` bash
iptables-save > 'firewall-backup'
```

To restore the configuration first clear the current config

``` bash
iptables -F
iptables-restore 'firewall-backup'
iptables -L
```

NAT (MASQUERADE) traffic from LAN out through NAT:

To rewrite the source address of packets coming from the VPN client with that of the VPN server

`server`
```bash
iptables -t nat -A POSTROUTING -s VPN_NET/24 -d DEST_NET/24 -o DEV -j MASQUERADE
```

- `-t nat`: Specifies the `nat` table, used for changing source/destination IP addresses.
- `-A POSTROUTING`: Appends a rule to the `POSTROUTING` chain, which is used after the routing decision - just before the packet leaves the system.
- `-s NETWORK`: Matches packets **originating from the VPN subnet**
- `-d NETWORK`: Matches packets **going to** the destination network
- `-o DEV`: Applies the rule if only the packets is going out via the DEV interface
- `-j MASQUERADE`: Masks (NATs) the source IP address of the packet with the IP of the outgoing interface, so that return traffic can find its way back

#### DNAT Rule (Destination NAT)

Use `iptables` to forward traffic destined to `10.0.2.7:1194` to the VPN server `10.0.7.2:1194`:

```bash
iptables -t nat -A PREROUTING -i eth0 -p udp --dport 1194 -j DNAT --to-destination 10.0.7.2:1194
```

