
RHEL containers

* **podman**: the tool that managers containers. Can start containers as regular users. *Rootless containers*
* **buildah**: custom image builder
* **skopeo**: tool for managing and testing container images
* **entrypoint**: the default command a container is configured to start when run. The CMD part of the image inspect shows the default command

Containers in RHEL run in user space. Containers run in the same kernel. 
Podman containers run on top of CRI-o container runtime, no need for daemon

**Rootless containers**:
* more secure
* cannot access resources that require root access
* do not have IP address
* can bind only non-privileged TCP/UDP ports
* if the container needs access to host based storage, the user running the container needs to be owner of the directory that provides the storage

##### Container Host Requirements

**namespace**: provides isolation for system resources

 Current namespaces:

* **mount**
* **process**
* **network**: similar to VLAN isolation.
* **user**
* **inter-process communication (ipc)**

**cgroup**: Control Group. Kernel feature that enables resource access limitation. There are no limitations by default as to how much memory or CPU cycles a process can access. Cgroups make it possible to create such limitations

**SELinux** secures access by using resource labels. Context labels are added to ensure the container can access only what it is supposed to access and nothing else.


#### Running a Container

To run a container
**$ podman run** *CONTAINER*

Containers run in the foreground if no arguments are provided.
To run a container in a detached mode:
**$ podman run -d --name**=*NAME* *CONTAINER*

Containers started in detached mode run like a daemon in the background

To run a container in TTY mode which displays process output of the container:
**$ podman run -it** *CONTAINER*

To start a container in interactive mode with shell access:
**$ podman run -it** *CONTAINER* **/bin/bash**

The **exit** command will exit the container and terminate it.
To detach from the active container and leave it running in the background:
$ CTRL + P, CTRL + Q

To show currently running containers:
**$ podman ps**

To show running and recently stopped containers:
**$ podman ps -a**

```
# [kimchen@rhel9 ~]$ podman ps -a
CONTAINER ID  IMAGE                           COMMAND               CREATED             STATUS                    PORTS       NAMES
6f8e714d6ef4  docker.io/library/nginx:latest  nginx -g daemon o...  4 hours ago         Exited (0) 3 hours ago                thirsty_noyce
32a978703b71  docker.io/library/nginx:latest  /bin/bash             3 hours ago         Exited (127) 3 hours ago              determined_sutherland
977f76d3adae  docker.io/library/nginx:latest  /bin/sh               About a minute ago  Up About a minute                     brave_wu

```

To reconnect to an active container, running in the background
**$ podman attach** *CONTAINER_NAME*

Non-root container files are copied to:
**~/.local/share/containers/storage**

