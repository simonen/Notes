
Features of Stratis:
* **thin provisioning**
* **snapshots**: a saved state of the system as of the moment it was taken
* **cache tier**
* **API**
* **monitoring and repair**
#### Stratis Architecture

The base level of Stratis is the **pool** - **blockdev**. Similar to the LVM **Volume Group**.
**/dev/stratis/poolname**: pool location

Uses about 500M per device up front.

Works with **XFS** only. FS size is not specified. Works with thin provisioning. Each FS can grow up to the size of the available storage. The volume grows as more data is put in it.

Install stratis and enable it:
**$ dnf install stratis-cli stratisd**
**$ systemctl enable --now stratisd**

Create a pool
**$ stratis pool create** *POOLNAME* */dev/DEV*

Add a device to an existing pool:
**$ stratis pool add-data** *POOL* */dev/DEV*

Add a filesystem to the pool
**$ stratis fs create** *POOL* *FS_NAME*

To mount a **stratis** filesystem through **/etc/fstab** use **UUID**, device name does NOT work!
Mandatory mount option: **defaults**,**x-systemd.requires=stratisd.service**

#### Managing Stratis

Traditional Linux tool cannot handle thin-provisioned volumes

Stratis tools:
**stratis blockdev**: lists all storage devices used by stratis
**stratis pool**: info about pools
**stratis filesystem**: fs info
#### Snapshots

To make a filesystem snapshot
**$ stratis fs snapshot** *POOLNAME* *FS_NAME* *SNAPSHOT_NAME*

To recover a snapshop, mount the newly created filesystem(snapshot) into a directory.
**$ mount /dev/stratis/**POOL/SNAPSHOT /*MOUNTPOINT*

