
The **SCSI** protocol was developed to connect local disks to servers
The **iSCSI** encapsulates **SCSI** commands in the IP packets using **TCP** to guarantee delivery
Works best with at least **10G** network infrastructure

Hardware **iSCSI SAN** could offer better performance over software **iSCSI SAN** depending on how it is optimized. No fundamental difference.

* **host bus adapter** (**HBA**): special **iSCSI** optimized network cards that offer a TCP offload engine that allows TCP packets to be handled by the card, not the CPU

Hardware SAN could require licensing for additional feature that could add up to cost as compared to a software SAN which is free.

**iSCSI shared storage devices**: typically **LVM logical volumes**. Could be **entire disks**, **partitions** or even **image files** (empty file that acts as the storage backend)

iSCSI is used with a dedicated network with redundancy. iSCSI traffic only.
A redundant network could cause the **iSCSI** device to appear multiple times. To avoid this, the **iSCSI initiator** should be configured to use the **multipath driver**
The **multipath driver** provides one interface the initiator could talk through allowing continuous operation if some of the redundant lines fail

Terminology

* **IQN**: iSCSI Qualified Name. A unique name used for identifying targets and initiators
* **Backend Storage**: the storage device the iSCSI target is providing access to
* **iSCSI target**: the server that provides the shared storage
* **iSCSI initiator**: the client, identified by an IQN, that connects to the SAN storage
* **ACL**: Access Control List. Based on the **iSCSI initiator IQN** that should be provided access to the target
* **LUN**: **Logical Unit Number**. The backend storage devices that are shared through the target. Could be any device supporting read/write ops: disks, partitions, logical volumes, tapes and files. The **LUN**s are needed to associate a block device with a **TPG**
* **Portal**: or **node**. The IP and port address that targets and initiators use to establish connections
* **TPG**: **Target Portal Group.** A collection of IP addresses and ports a specific iSCSI target listens to
* **Discovery**: the initiator finds a target and saves information about it locally for future reference. Done by the `iscsiadm` command
* **Login**: Authentication that gives the initiator access to the target LUN. Done with the `iscsiadm` command


#### Setting Up an iSCSI Target

* **targetcli**: the default interface to manage **LIO iSCSI** targets. Uses familiar Linux commands: ls, cd, pwd...
* **LIO** (Linux I/O): the **iSCSI** target package used by RHEL. Has native support for **OpenStack**

##### Create the iSCSI Target

Start the target service

```
systemctl enable --now target
```

**IQN** convention
iqn.yyyy-mm.inverteddomain:target

Steps to create a target:
1. Backstores
2. IQN
3. ACL
4. LUNs
5. Portal

6. Install the `targetcli` package
7. Create the backed storage devices
8. Configure the backstore
	`cd /backstores` ; **block/ create** block1 *BACKEND_STORAGE_DEV*
		the **create** command comes from the **block/** dir
	To create a file-backed storage device:
		**/fileio>** **create file1 /root/** *FILENAME* *SIZE*
4.  Configure the **IQN** of the iSCSI target:
	1. **/iscsi>** **create** iqn.yyyy-mm.inverteddomain:**target**
5. Create the **ACL**:
	1. /iscsi/iqn.../tpg1/acls> **create** *iqn.yyyy-mm.inverteddomain:initiator*
	2. Add the *initiator* in **/etc/iscsi/initiatorname.iscsi**
6. Create the LUNs
	1. `/iscsi/iqn.20...get/tpg1/luns> create /backstores/block/block1`
	2. `/iscsi/iqn.20...get/tpg1/luns> create /backstores/block/block2`
	3. `/iscsi/iqn.20...get/tpg1/luns> create /backsores/fileio/file1`
		to assign a LUN a specific number: **create lun**=*NUMBER* **storage=/backstores/**block/block1
7. Create the Portal:
	1. `/iscsi/iqn.20.../tpg1/portals>**create** *IP_ADDRESS PORT*`
8. Overview and **exit**

##### Firewall

Open port 3260

```bash
firewall-cmd --add-port=3260/tcp --permanent ; firewall-cmd --reload
```

#### iSCSI Initiator

Package: `iscsi-initiator-utils` (provides `iscsiadmin`)

`/etc/iscsi/initiatorname.iscsi`: initiator name is specified
`iscsid`: the main service that accesses all config files
`iscsi`: the service that establishes iSCSI connections
`iscsiadm` is used to discover iSCSI targets and connect to them

Start the `iscsid` service:

```bash
systemctl enable --now iscsid
```

Perform a target discovery:

```bash
iscsiadm -m discovery -t sendtargets --portal 'TARGET_IP:PORT' --discover
```

if successful

```
192.168.137.13:3260,1 iqn.2024-01.lab.lsaa:target
```
##### Establish a connection

The initiator name in `/etc/iscsi/initiatorname.iscsi` must be the same as specified in the ACL on the target


```
/iscsi/iqn.20...get/tpg1/acls> ls
o- acls 
  o- iqn.2024-01.lab.lsaa:server1 
```
```
[root@server1 ~]# cat /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2024-01.lab.lsaa:server1
```

```bash
iscsiadm -m node -t 'TARGET_IQN' --portal 'TARGET_IP:PORT' --login
```

`--mode node`: the mode in which an actual connection to the target can be established
`--login`: this authenticates to the target and keeps the connection after a reboot

```
Logging in to [iface: default, target: iqn.2024-01.lab.lsaa:target, portal: 192.168.137.13,3260] (multiple)
Login to [iface: default, target: iqn.2024-01.lab.lsaa:target, portal: 192.168.137.13,3260] successful.
```

To list scsi devices:

```bash
lsscsi
```

```
[root@server1 ~]# lsscsi
[0:0:0:0]    disk    VBOX     HARDDISK         1.0   /dev/sda
[0:0:1:0]    disk    VBOX     HARDDISK         1.0   /dev/sdb
[6:0:0:0]    disk    LIO-ORG  block1           4.0   /dev/sdc
[6:0:0:1]    disk    LIO-ORG  block2           4.0   /dev/sdd
[6:0:0:2]    disk    LIO-ORG  file1            4.0   /dev/sde
```

To get info about the current session

```bash
iscsiadm -m session -P \[0-3]
```

##### Making connections persistent

After connecting to the target, connections are persistent by default

- `/var/lib/iscsi`: Relevant iSCSI conf information is stored here 
- `/var/lib/iscsi/nodes`: contains info about previous **iSCSI** connections. Contains dirs named after the target IQNs/Portals/default. The default file contains session information. This dir has to have at least one node for the `iscsi.service` to be able to start

To make sure a connection is not stored after a reboot:
1. Close an iSCSI session:

```bash
iscsiadm -m node --targetname 'TARGET_IQN' --logout
```

```bash
iscsiadm -m node --targetname 'TARGET_IQN' --op=delete 
```

or delete the corresponding IQN dir in `/var/lib/iscsi/nodes`
##### Mounting iSCSI devices

The `fstab` file gets processes before the network is available.
It should looke like this:

```
UUID=XXXXX /MOUNTPOINT FILESYSTEM _netdev 0 2
```

