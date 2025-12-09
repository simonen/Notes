
* The GlusterFS network filesystem is a "no metadata" distributed filesystem. It does not have a dedicated metadata server that is used to handle file location data. Uses a deterministic hashing technique to discover file location
* Exports a fully POSIX-compliant filesystem, which means you can mount, read and write to GlusterFS from Unix and Unix-like OS
* GlusterFS is a user space filesystem, meaning it does not run in the Linux kernel but uses the FUSE (UserspacE Filesystem) module
* A `trusted pool` is a network of servers operating in the GlusterFS cluster. Each server is called a `peer`.
* A `brick` is a basic unit of storage for the GlusterFS. 
* A `volume` is a logical collection of bricks

Volume types:

* `Distributed`: files are distributed across bricks, no redundancy
* `Replicated`: files are replicated across all bricks in the volume. Min of 2 bricks. 
* `Distributed replicated`: distribute files across replicated bricks in the volume
* `Striped`: files are divided into chunks and distributed across bricks (chunks = bricks). Large files
* `Distributed striped`: deprecated
* `Dispersed`:
* `Distributed dispersed`:

Ports:
- `TCP/24007` (Gluster management)
- `TCP/24008` (Gluster self-heal)
- `TCP/49152-49156` (Brick ports)

> Make sure DNS is configured properly and all hostnames are resolvable, glusterd glusterfsd services are running, firewall ports are added

Package
centos-release-gluster (packages from the CentOS Storage SIG repository)
glusterfs-server

At 3 peers are needed to create a volume. The number must be odd.

Format the storage disk with xfs fs. 
[[LINUX/STORAGE/Filesystems]] 

```bash
mkfs.xfs -L 'brick1_r1' /dev/'DEVICE'
```

Create the `brick` directory

```
mkdir -p /data/brick1_r1 
```

`/etc/fstab`
``` 
LABEL=brick1_r1    /data/brick1_r1    xfs    defaults    1    2
```

Mount the new filesystem to the brick directory

``` bash
mount -a
```

``` bash
findmnt
# TARGET                           SOURCE              FSTYPE
# └─/data/brick1_r1                /dev/sdb1           xfs
```

Firewall and SELinux

``` bash
firewall-cmd --add-port=24007-24008/tcp --permanent && firewall-cmd --reload
firewall-cmd --add-port=49152-49156/tcp --permanent && firewall-cmd --reload
```

``` bash
sudo setsebool -P gluster_use_execmem on
sudo setsebool -P cluster_use_execmem on
```

Add the peers

``` bash
gluster peer probe 'PEER1'
```

`peer`
```
State   Local Address:Port   Peer Address:Port    Process
ESTAB   192.168.137.16:24007  192.168.137.5:49149  cgroup:/system.slice/glusterd.serv
ESTAB   192.168.137.16:24007  192.168.137.10:49150 cgroup:/system.slice/glusterd.serv
```

See hosts in the gluster trusted storage pool. Works on every peer

``` bash
gluster pool list
```

```
Number of Peers: 2

Hostname: rocky.ohio.cc
Uuid: 730e6c85-a1d0-4f92-bfe7-c7fe7ac9b246
State: Peer in Cluster (Connected)

Hostname: york.ohio.cc
Uuid: 61504af2-ef3e-4c96-87d0-981b458610b3
State: Peer in Cluster (Connected)
```

Create a gluster volume. Create a /vol1 directory in each brick

``` bash
gluster volume create vol1 replica 3 \
 vm1.ohio.cc:/data/brick1_r1/vol1 \
 rocky.ohio.cc:/data/brick1_r1/vol1 \
 york.ohio.cc:/data/brick1_r1/vol1
 # volume create: vol1: success: please start the volume to access data
```

Get volume info 

``` bash
gluster volume info 'vol1'
```

```
Volume Name: vol1
Type: Replicate
Volume ID: cb0df995-7461-492f-a872-8c193d1a1116
Status: Created
Snapshot Count: 0
Number of Bricks: 1 x 3 = 3
Transport-type: tcp
Bricks:
Brick1: vm1.ohio.cc:/data/brick1_r1/vol1
Brick2: york.ohio.cc:/data/brick1_r1/vol1
Brick3: rocky.ohio.cc:/data/brick1_r1/vol1
transport.address-family: inet
```

Start the volume

``` bash
gluster volume start 'vol1'
```

#### Mounting GlusterFS volumes

Package: 
`glusterfs-fuse`

`man mount.glusterfs` (for mount options)

Mount the gluster volume 

``` bash
mount -t glusterfs 'GLUSTER_PEER:/VOLUME' /'MOUNTPOINT
```

```
TARGET          SOURCE                FSTYPE           OPTIONS  
└─/gluster      vm1.ohio.cc:/vol1     fuse.glusterfs   rw,user_id=0,group_id=0,
```

``` bash
df /'MOUNTPOINT'
```

```
Filesystem        1K-blocks   Used Available Use% Mounted on
vm1.ohio.cc:/vol1   5175296 121124   5054172   3% /gluster
```

#### Managing Volumes

Volume configuration

``` bash
gluster volume set 'VOLUME' "OPTION" "VALUE"
# gluster volume set "VOLUME" <TAB><TAB> for a list of options
```

Change the ping timeout setting

``` bash
gluster volume set 'VOLUME' network.ping-timeout 15
```

#### Expanding a Volume

Creating an adding another set of 3 bricks to the same volume

```
Volume Name: vol1
Type: Distributed-Replicate <---
Volume ID: cb0df995-7461-492f-a872-8c193d1a1116
Status: Started
Snapshot Count: 0
Number of Bricks: 2 x 3 = 6 <----
Transport-type: tcp
Bricks:
Brick1: york.ohio.cc:/data/brick1_r1/vol1
Brick2: rocky.ohio.cc:/data/brick1_r1/vol1
Brick3: vm1.ohio.cc:/data/brick1_r1/vol1
Brick4: vm1.ohio.cc:/data/brick1_r2/vol1
Brick5: rocky.ohio.cc:/data/brick1_r2/vol1
Brick6: york.ohio.cc:/data/brick1_r2/vol1
Options Reconfigured:
```

Replacing failed bricks

Brick vm1.ohio.cc:/data/brick1_r2/vol1 is dead

```
Status of volume: vol1
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick york.ohio.cc:/data/brick1_r1/vol1     49152     0          Y       1335
Brick rocky.ohio.cc:/data/brick1_r1/vol1    49152     0          Y       1472
Brick vm1.ohio.cc:/data/brick1_r1/vol1      49152     0          Y       1803
Brick vm1.ohio.cc:/data/brick1_r2/vol1      N/A       N/A        N       N/A
```
