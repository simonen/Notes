ipv6 is a 64-bit address, 8 groups of hex numbers

fe80:0000:0000:0010:29ff:fee4:714a:0001

leading 0s can be skipped:
fe80::10:29ff:fee4:714a:1

Notable ipv6 addresses:
* **::1/128** - localhost
* :: - all addresses
* **::/0** - default gateway
* **2000::/3** - Global unicast address
* **fc00::/7** - Unique local address
* **fe80::/64** - link-local address
* **ff00::/8** - multicast. IPv6 does not use broadcast
* **2001:db8/32** - addresses reserved for documentation
* **ff02::1:2** - all-dhcp-servers link-local multicast group

**link-local**: fe80::/64 prefix + mac address. **ffee** is inserted in the middle of the mac part Automatically assigned. Not routable.
**ff02::1** - multicast address. All-nodes link-local address

08:00:27:3D:B8:50
fe80::683a:ce70:c5e2:b8e8/64

#### Managing IPv6 Configurations

**DHCP**

**ff02::1:2** to port 547/UDP : multicast ipv6 address in the all-dhcp-servers link-local multicast group. DHCP6 returns an answer to port 546
**SLAAC**, Stateless Address Auto Configurations: The host brings an iface with **fe80::/64** and sends a request to **ff02::2:2** (all-router link-local multicast group)