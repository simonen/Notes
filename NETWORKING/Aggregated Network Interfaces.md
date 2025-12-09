
[[gitea/LINUX 1/NETWORKING/NMCLI]]

* **bonding**: old method. Happens entirely in user namespace
* **teaming**: preferred method. Consists of the **teamd** daemon to allow interaction in user space and runners

##### Bonding

man `ifenslave`

Info: 
`/usr/share/doc/iputils/README.bonding`

Use `nmtui` or `nmcli` to configure **bonding**

Create the master bond interface:

``` bash
nmcli con add con-name "CONN" type bond ifname "CONN" mode active-backup
```

To print all bond modes

``` bash
nmcli con add type bond mode <TAB> <TAB>
```

Add physical slave interfaces to the master bond interface:

``` bash
nmcli con add type bond-slave ifname "SLAVE_DEV" master "BOND"
```

``` bash
nmcli con add type bond-slave ifname eth1 master mybond0
nmcli con add type bond-slave ifname eth2 master mybond0
```

Add an ip address to the bond interface:

``` bash
nmcli con mod "BOND" ipv4.method manual ipv4.addresses "IP/MASK" 
```

To verify the status of an operation bond using the /proc

``` bash
cat /proc/net/bonding/"BOND_INT"
```

```
[kimchen@ipa bonding]$ cat mybond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: enp0s8
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:e1:87:a3
Slave queue ID: 0
```

Starting up the master bond interface does not start the slaves automatically. Starting up a slave, starts up the master bond.

When bonding interface goes online, the bonding module is loaded 

```
lsmod | grep bonding
bonding               274432  0
```

To load the bonding mode manually

```
modprobe bonding
```

The `ifenslave` utility can also be used to attach or detach slave network devices to a bonding device
##### NIC Teaming

man teamd

Network teaming is new to RHEL7

Consists of:
* a small kernel driver handling packets, runner
* daemon **teamd** working in user space, handling logic and iface processing

Teaming hierarchy 
1. Team
2. Slave
3. Device

Teaming runners:
* roundrobin
* activebackup
* broadcast: sends traffic to all interfaces
* loadbalance
* lacp

Configuring teaming
**nmcli** and **nmtui**

1. Create the team interface
2. Assign the master interface IP address while no slave ports are attached to it.
3. Assign slave interfaces. Slave devices do not have IP addresses as they are managed by the master.
4. Bring team and slave interfaces up

Create the team interface. Runner configuration is provided in JSON format, either directly or in a json file.

``` bash
nmcli con add type team con-name "TEAM_CONN" ifname "TEAM_CONN" config "JSON.CONF"
```

Device name defaults to "nm-team" if *ifname* not specified

``` bash
nmcli con add type team con-name "Team1" config '{ "runner": {"name": "loadbalance"}}'
```

Assign an IP address to the team master iface:

``` bash
nmcli con mod "TEAM_CONN" ipv4.addresses "IP_ADDRESS/MASK"
```

Add the slave interfaces to the team master

``` bash
nmcli con add type team-slave ifname "SLAVE_DEV" master "TEAM_CONN"
```

Bring the Team connection up:

``` bash
nmcli con up "TEAM_CONN"
```

To verify the Team connection using the **teamd** user space daemon

``` bash
teamdctl "TEAM_DEV" state
```

```
[root@ipa system-connections]# teamdctl Team1 state
setup:
  runner: loadbalance
ports:
  enp0s10
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
        down count: 0
  enp0s9
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
        down count: 0
```

Use **teamnl** to debug teaming:

```
[root@ipa network-scripts]# teamnl Team2 ports
 5: enp0s10: up 1000Mbit FD
 4: enp0s9: up 1000Mbit FD
```

To dump the configuration in json format:

``` bash
teamdctl "TEAM" config dump
```

```
[root@ipa network-scripts]# teamdctl Team2 config dump
{
    "device": "Team2",
    "ports": {
        "enp0s10": {
            "link_watch": {
                "name": "ethtool"
            }
        },
        "enp0s9": {
            "link_watch": {
                "name": "ethtool"
            }
        }
    },
    "runner": {
        "name": "loadbalance",
        "tx_hash": [
            "eth",
            "ipv4",
            "ipv6"
        ]
    }
}
```
