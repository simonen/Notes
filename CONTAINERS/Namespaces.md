---
tags:
  - containers
---
Namespaces are kernel feature that isolates and virtualizes system resources for a group of processes, enabling them to have different views of the underlying system.

> A particular namespace lives as long as there is a process running in it.

There are currently six namespaces used by containers

- Mount NS. 
- `UTS (Unix Time-sharing)` NS: Hostname and domain name isolation
- `IPC`
- `PID`
- User name spaces
- Network NS

#### Creating Namespaces

Namespaces are created with the `unshare` tool. 

```bash
unshare "NS" [OPTIONS] "CMD"
```

To create a namespace and execute a process in it. if no command is given, defaults to `/bin/bash` which becomes its main process.

```bash
unshare -m
```

List information about the currently accessible namespaces

```bash
lsns -T --output NS,TYPE,NPROCS,PID,PPID,USER,UID,COMMAND,PNS,ONS
```

`root`
```
NS           TYPE   NPROCS   PID PPID USER UID COMMAND      PNS         ONS
4026531837   user      141     1     0 root   0 /sbin/init    0           0
├─4026531862 mnt         1    62     2 root   0 kdevtmpfs     0  4026531837
```

`vagrant`
```
NS           TYPE   NPROCS   PID PPID USER    UID  COMMAND   PNS        ONS
4026531837   user        3  1742    1 vagrant 1000 /lib/s...   0          0
└─4026531841 mnt         7  1742    1 vagrant 1000 /lib/s...   0 4026531837
```

- `4026531837`: The Parent NS. The **root user namespace**.
- `NS`: The namespace's `inode` number
- `net, mnt`:  Type of the namespace
- `NPROCS`: Number of processes residing in that namespace
- `PID`: A process inside that namespace
- `--user`: User instance of `systemd`, not the **system-wide** int.
- `/lib/systemd/systemd --user`: **Per-user service manager** that runs when the user logs in.
- `PNS`: Parent NS
- `ONS`: Owner NS

In the listings above, the PNS shows different number of processes for the namespace. From the `vagrant`'s user perspective, there are only 3 processes inside the namespace.

Get the PID of the current shell session

```bash
echo $$
```

See which processes share a namespace

```bash
# Find all processes with the same network namespace as PID 1649
find /proc -maxdepth 3 -name net -lname "*4026531840*" 2>/dev/null | awk -F/ '{print $3}' | xargs -I {} ps -p {} -o pid,user,comm --no-headers
```

```
   1649 vagrant  systemd
   1660 vagrant  bash
   1718 vagrant  bash
  66549 vagrant  ssh
  66563 vagrant  xargs
```

Namespace references are listed in `/proc/PID/ns`

```
user -> 'user:[4026532209]'
pid -> 'pid:[4026531836]'
```

Example: See which user namespace you are currently in and refer to the `lsns` output

```bash
readlink /proc/$$/ns/user
# 4026531837 <- root user namespace
```

#### Mount Namespaces

Mount namespaces isolate the filesystem mount points. Processes inside that namespace have a unique view of the filesystem hierarchy.  They can have directories mounted or unmounted without affecting the host or other namespaces.

Ge the PID of the current shell session

```bash
echo $$
2738
```

`root PID.2738`
```bash
mkdir /tmp/mount_ns
```

Start a new shell that is in its own, private mount namespace (`-m` flag).

`root PID.2738`
```bash
sudo unshare -m /bin/bash
```

The `unshare` command creates new namespaces and then executes the specified program. Defaults to `/bin/bash`. The PID of the new shell session is 2767, the parent PID - 2738.

`root PID.2767
```
NS           TYPE   NPROCS   PID  PPID UID USER COMMAND        PNS        ONS
4026531837   user      135     1     0   0 root /sbin/init       0          0
├─4026532200 mnt         2  2767  2738   0 root -bash            0 4026531837
```

The new `4026532200` mount namespace is now created and has just the new shell process living in it. Terminating that shell process (2767) will kill the namespace too.

Mount a `tmpfs` into the newly created folder

`root PID.2767`
```bash
mount -n -t tmpfs tmpfs /tmp/mount_ns
```

The new mount is ready

`root PID.2738`
```bash
cat /proc/mounts | grep mount_ns
```

```
tmpfs /tmp/mount_ns tmpfs rw,relatime,inode64 0 0
```

From another root shell on the same host.

`root PID.2981`
```
NS           TYPE   NPROCS   PID  PPID UID USER COMMAND        PNS        ONS
4026531837   user      141     1     0 root   0 /sbin/init       0          0
├─4026532200 mnt         1  2767  2738 root   0 -bash            0 4026531837
```

The new `4026532200` mount namespace has only the shell process `PID.2767` that it was created with, (`NPROCS` 1). Therefore, `PID.2981` is outside that namespace and will not be able to see the `mount_ns` mount, or any mounts created in that namespace.

`root PID.2981`
```bash
cat /proc/mounts | grep mount_ns # expectedly will show nothing
```

Another way to prove it is that `PID.2981` belongs to a different mount space.

`root PID.2981`
```
/proc/2981/ns/mnt -> 'mnt:[4026531841]'
```

Conversely, mounts done by the rest of the system will not propagate into the new namespace mount table.

Perform mount operations in the specified namespace

```bash
mount -N "NS" ...
```

#### UTS Namespaces

**Unix Timesharing Namespace** is used to isolate two system identifiers - hostname and domain name from one another. Allows containers to have their own hostname and domainname.

`root PID.2738`
```
# hostname
docker
```

Create a new UTS namespace and execute a bash shell in it

`root PID.2738`
```bash
unshare -u
```

As seen from another session on the same host

`root PID.3574`
```
NS           TYPE   NPROCS   PID  PPID UID USER COMMAND      PNS        ONS
4026531837   user      142     1     0 root   0 /sbin/init     0          0
├─4026532200 uts         1  3553  2738 root   0 -bash          0 4026531837
```

Change the hostname within that new namespace

`root PID.3553`
```bash
hostname uts-namespace
```

Sessions outside the new `uts` namespace still see the original hostname.

`root PID.3574`
```
# hostname
docker
```

> Changing the hostname inside a new namespace does not affect DNS resolution automatically.

#### PID Namespaces

Process ID namespaces provide the ability for a process to have a PID that already exists in the default namespace.

Create a new PID namespace

`root PID.2738
```bash
unshare -p -f --mount-proc
```

- `-p`: New PID namespace
- `-f`: Fork the parent process. The child process becomes main process (PID 1) in the new NS. Parent stays in its original NS. Mandatory.
- `--mount-proc`: Remounts `/proc` so that the kernel generates the new PID tree.

**PID Namespaces are hierarchical** - every new PID ns is child of the one that spawned it.
We can also see that a new mount ns has also been created. That is so to provide an isolated mount point for the new `/proc`. 

`root PID.4289`
```
NS           TYPE   NPROCS   PID  PPID UID USER COMMAND        PNS        ONS
4026531837   user      142     1     0 root   0 /sbin/init       0          0
├─4026531836 pid       140     1     0 root   0 /sbin/init       0 4026531837
├─4026532200 mnt         2  5186  2738 root   0 unshare -p -f... 0 4026531837
├─4026532201 pid         1  5187  5186 root   0 -bash   4026531836 4026531837
```

The process tree with forking:

`root PID.4289`
`bash(2738)───unshare(5186)───bash(5187)`

- `bash(2738)`: the original shell (in the host PID namespace)
- `unshare(5186)`: the short-lived `unshare` helper that creates the new namespace and executes the `bash` shell in it.
- `bash(5187)`: the new shell running as PID 1 inside the new namespace.

The process isolation. If the `/proc` fs is not remounted, the underlying process tree will be shown.

`root PID.5187
```
USER         PID COMMAND
root           1 -bash
root          40 ps -eo user,pid,command
```

`/proc` filesystem is populated by the kernel **per PID namespace**. Inside that PID namespace, we want to see only processes and their children belonging to that PID namespace.

After entering a new PID namespace, the existing `/proc` still shows the host’s PID numbers. To obtain a `/proc` that lists only the PIDs within the new namespace, a fresh `proc` filesystem must be mounted. A private mount namespace is therefore typically created so that this new `/proc` can be mounted at `/proc` independently.

#### User Namespaces

`capsh`: Utility that inspects Linux capabilities **relative to the namespace of the current process**

The **user namespace** allows a process inside a namespace to have a different UID and GUID than that in the default namespace.

User Namespaces isolates:
- UID and GUID numbers themselves
- The interpretation of capabilities: a user can have its own "SYS" in its own namespace, while being unprivileged outside of it.
- Restrictions on many other namespaces: 

To execute a shell command inside a new user namespace as "root"

```bash
$ unshare --map-root-user --user sh -c whoami
root
```

```bash
unshare -U -r /bin/bash
```

- `-U`: new user namespace
- `-r`: map your UID GUID to 0 inside the new namespace

As an unprivileged user

```bash
unshare -U --map-root-user /bin/bash
```

(Works on modern kernels with `kernel.unprivileged_userns_clone=1`.)

```
vagrant@docker:~$ unshare -U --map-root-user
root@docker:~# id
uid=0(root) gid=0(root) groups=0(root),65534(nogroup)
```

**UID/GID Mapping**: inside this namespace UID 0 means UID 1000 on the host. 1 -> number of UIDs being mapped.

`/proc/$$/uid_map`
```
0       1000          1
```

`/proc/<pid>/uid_map` is **write-once per user-namespace lifetime**. Cannot be changed the initial generation.

```
NS           TYPE   NPROCS   PID  PPID UID USER COMMAND        PNS        ONS
4026531837   user      142     1     0 root   0 /sbin/init       0          0
├─4026531836 pid       140     1     0 root   0 /sbin/init       0 4026531837
├─4026532200 user        1 18051  1878 vagrant 1000 -bash 40.31837   40.31837
```

Inside the new user namespace, create the PID and `mnt` namespaces

`root PID.18051`
```bash
unshare -p -f --mount-proc
```

The new namespace tree

```
NS           TYPE   NPROCS   PID  PPID UID USER COMMAND        PNS         ON
4026531837   user      147     1     0 root       0 /sbin/init   0          0
├─4026531836 pid       147     1     0 root       0 /sbin/init   0 4026531837
├─4026532200 user        3 20688  1878 vagrant 1000 /bin/bash .837 4026531837
│ ├─4026532201 mnt       2 20958 20688 vagrant 1000 unshare ..   0 4026532200
│ └─4026532202 pid       1 20959 20958 vagrant 1000 -bash ROOT.PID 4026532200
```

> `CAP_SYS_ADMIN`, only gives authority over **resources in the user namespace**. 

#### Network Namespaces

`openvswitch-switch`: Multilayer, software-based, Ethernet virtual switch

Network namespaces creates a logical copy of the network stack, allowing multiple processes to listen on the same port from multiple namespaces.

Network namespaces provide isolation of networking resources
- network interfaces
- routing tables (IPv4/IPv6)
- firewall rules
- sockets, ports, per-interface stats, ARP/ND tables, etc.

Create a new network namespace

```bash
ip netns add 'NS'
#ns2
#ns1
```

Execute a network command in a network namespace

```bash
ip netns exec "NS" ip "COMMAND"
```

```bash
ip netns exec ns1 ip link
# 1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group efault 
```

Execute a bash shell into a net namespace

```bash
ip netns exec ns1 bash
```

Now we are switched to the new namespace. Regular `ip` commands can be used

`ns1`
```bash
ip a
# 1: lo: <LOOPBACK> mtu 6553
```

To forward frames between virtual ports, the `OpenVSwitch` bridge is used.

Create a network namespace with `unshare`

```bash
unshare -n
```

Create a pair of virtual interfaces

```bash
ip link add veth0 type veth peer name veth1
```

To move a virtual interface to the root namespace

```bash
ip link set veth1 netns 1
```


##### OpenVSwitch

Create a bridge

```bash
ovs-vsctl add-br OVS-1
```

List bridges

```bash
ovs-vsctl list-br
# OVS-1
```

```
root@docker:~# ovs-vsctl show
cb3e9e29-8916-4725-b299-8f20a9ee25db
    Bridge OVS-1
        Port OVS-1
            Interface OVS-1
                type: internal
    ovs_version: "3.1.0"
```

The newly created bridge now appears in the root net namespace

```bash
ip a
# 9: OVS-1: <BROADCAST,MULTICAST> mtu 1
```

Create virtual interfaces for the net namespaces `ns1` and `ns2`

```bash
ip link add eth1-ns1 type veth peer name veth-ns1
ip link add eth1-ns2 type veth peer name veth-ns2
```

```bash
ip link
# veth-ns1@eth1-ns1
# eth1-ns1@veth-ns1
# veth-ns2@eth1-ns2
# eth1-ns2@veth-ns2
```

Move the interfaces to their namespaces

```bash
ip link set eth1-ns1 netns ns1
ip link set eth1-ns2 netns ns2
```

Confirm they are in the right namespaces

```bash
ip netns exec ns1 ip link
```

Or just `ip link` in the root ns

```
10: veth-ns1@if11: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 6a:13:47:58:48:d1 brd ff:ff:ff:ff:ff:ff link-netns ns1
```

Connect the `veth` interfaces to the switch

```bash
ovs-vsctl add-port OVS-1 veth-ns1
ovs-vsctl add-port OVS-1 veth-ns2
```

```bash
ovs-vsctl show
```

```
cb3e9e29-8916-4725-b299-8f20a9ee25db
    Bridge OVS-1
        Port veth-ns1
            Interface veth-ns1
        Port OVS-1
            Interface OVS-1
                type: internal
        Port veth-ns2
            Interface veth-ns2
    ovs_version: "3.1.0"
```

Bring the interfaces up

`root ns`
```bash
ip link set veth-ns1 up
ip link set veth-ns2 up
```

`ns1,2`
```
# NS1
ip netns exec ns1 ip link set dev lo up
ip netns exec ns1 ip link set dev eth1-ns1 up
ip netns exec ns1 ip addr add 192.168.0.1/24 dev eth1-ns1
# NS2
ip netns exec ns2 ip link set dev lo up
ip netns exec ns2 ip link set dev eth1-ns2 up
ip netns exec ns2 ip addr add 192.168.0.2/24 dev eth1-ns2
```

Ping the interfaces between namespaces

```bash
ip netns exec ns1 ping -c 4 192.168.0.2
```

```
64 bytes from 192.168.0.2: icmp_seq=1 ttl=64 time=1.29 ms
```

#### IPC (Inter-process Communication) Namespace

IPC Namespaces provide isolation for a set of IPC and synchronization facilities. These facilities provide a way of exchanging data and synchronizing the actions between processes and threads. They provide primitives such as file locks, semaphores, mutexes, that are needed to have true process separation.
#### Resource Management with CGroups

`Cgroups` are a kernel feature that allow fine-grain allocation of resources to a single process or a group called `tasks`. 

>TO DOOOOOO
>
#### Putting it all together

The container will needs to have its own isolated root directory with the necessary binaries. 

A minimal `rootfs` can be obtained via the `alpine-minirootfs` or `busybox` to provide the container with a directory structure and basic tools, such as `ls, sh, chroot, ip, ping`, i.e., bootstrapping.

`alpine-minirootf`
```bash
ls /root/rootfs
bin  dev  etc  home  lib  proc  root  run  sbin  srv  sys  tmp  usr  var
```

Create the namespace subtree

```bash
unshare --mount --uts --ipc --net --pid --fork --user /bin/bash
```

Here is the entire namespace sub-tree with user namespace 4026532418 as its root

```
NS            TYPE  NPROCS  PID PPID USER   UID  COMMAND       PNS      ONS
4026531837    user      90    1 0     root    0  /sbin/init      0        0
├─4026532418  user       2 1444 967   root    0  unshare      .837     .837
│ ├─4026532419 mnt       2 1444 967   root    0  unshare         0     .209
│ ├─4026532420 uts       2 1444 967   root    0  unshare         0     .209
│ ├─4026532421 ipc       2 1444 967   root    0  unshare         0     .209
│ ├─4026532422 pid       1 1445 1444  root    0  -bash        .836     .209
│ └─4026532423 net       2 1444 967   root    0  unshare         0     .209
```

We are now inside the new user namespace. However, we are still in the host root fs and reading the host `/proc`. 

`ns`
```bash
root@docker:/# ls
bin  boot  dev  etc  home  in
```

Switch to the container root filesystem. 

```bash
chroot /root/rootfs /bin/sh
```

`/bin/sh` is the new `alpine-minitoortfs` provided shell

Check the system file paths

```bash
export
PATH="/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```

In this case the new root filesystem shell will not be able to find commands like `ls`. 

Export the new file paths 

`new ns`
```bash
export PATH=/bin:/sbin:/usr/bin:/usr/sbin
```

```bash
/ \# ls
# bin    dev    etc    home   lib  mnt proc   root sbin  sys    tmp    usr    var
```

`ps` won't show anything if the new `proc` filesystem is not mounted

```bash
mount -t proc proc /proc
```

```
ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/bash
   24 root      0:00 /bin/sh
   30 root      0:00 ps
```

Optionally, display a proper prompt

`new ns`
```bash
export PS1='\[\e[0;36m\][\u@\h \W]# \[\e[m\]'
```

To make the process a bit cleaner, `chroot` to the container's root directly when entering the ns

```bash
nsenter --target 42380 --mount --uts --ipc --net --user \
chroot /root/zibiten/rootfs /bin/sh
```

- `-t PID`: Specify a PID of a process inside the target namespace

We've created the container's base layer, which has to be read-only (immutable).

Mount namespaces only controls visibility of mounts, not storage separation. Real containers need layered filesystems (OverlayFS) for true separation.

#### OverlayFS

Container filesystem structure

```
mycontainer/
├── lower/   ← your rootfs image
│   ├── bin/
│   ├── lib/
│   ├── etc/
│   └── ...
├── upper/   ← COW changes written here
├── work/    ← overlayfs internal. Do not touch!
└── merged/  ← final root filesystem
```

`rootfs` goes in `lower/` and the container root is then mounted in `merged/`

`container space`
```bash
mount --bind rootfs/ mycontainer/lower
```

```bash
mount -t overlay overlay -o \ lowerdir=$HOME/overlay/lower,upperdir=$HOME/overlay/upper,workdir=$HOME/overlay/work $HOME/overlay/merged
```

```bash
chroot /root/overlay/merged /bin/sh
```