
To execute a command in a running container from outside of it and display result to STDOUT:

**$ podman exec** *CONTAINER* *COMMAND*

To open a bash shell in a container:
**$ docker exec -it** *CONTAINER* */bin/bash*

-i: interactive mode. Allows **STDIN**
-t: creates a pseudo TTY terminal
-d: detached mode. The container runs in the background

Th **exit** command in this case will exit the container bash shell but the container will remain running in the background

#### Managing Container Ports

Rootless containers do not have IP addresses. 
Root containers run in the root user **namespace** and thus are not visible to non-root users
The **sudo podman ps** command will display root containers only

To run a root container:
**$ sudo podman run** *CONTAINER*

To run a container with port forwarding:
**$ podman run -d -p** *8080:80* *CONTAINER*

```
[kimchen@rhel9 exercise264]$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS         PORTS                 NAMES
dd58ed022ea7  docker.io/library/nginx:latest  nginx -g daemon o...  33 minutes ago  Up 33 minutes                        web2
00d40c4a5602  docker.io/library/nginx:latest  nginx -g daemon o...  3 seconds ago   Up 3 seconds   0.0.0.0:8080->80/tcp  nginxport
```

In order for the app to be accessed by other pc's, the port should be opened by the firewall
**$ firewall-cmd --add-port**=*PORT/protocol* --permanent

#### Container Environment Variables

Inspect an image for **usage** information, if available. Some applications require environment variables to be set.

**$ podman -e** *ENV_VARIABLE1 -e ENV_VARIBABLE2* etc... *CONTAINER*

#### Container Storage

Writable layer that is added to a container is ephemeral - it is lost when the container is stopped. Persistent storage is achieved with the **bind-mount**. A directory on the host is mounted on the container and info from the container is committed to the host directory as well.

* the host directory must be writable for the user that runs the container
* SELinux context label of the directory must be set to** container_file_t**. Can be set automatically if the user running the container is **owner** of the host dir

The SELinux context of а host dir
```
drwxr-xr-x.  6  100998  100998 system_u:object_r:container_file_t:s0:c510,c658     4096 Jan 11 17:06 dbfiles
```

If the right SELinux context isn't there, container data will not be saved.
The user running the container must be owner of the host dir
**$ sudo chown $(whoami):\$(whoami)** *DIR*

To set the SELinux context automatically:
**$ podman run -d -v** *HOST_DIR:CONTAINER_DIR*:**Z** *CONTAINER*

:Z - makes the SELinux context adjustment automatically

To remove a container:
**$ podman rm** *CONTAINER*

#### Bind-mounting in Rootless Containers

Rootless containers will not start if the proper permissions are not set. The directory of the host must be in the unprivileged user namespace home directory and the container UID must be owner of the directory

To get the container UID:
**$ podman inspect | grep User**

To run a command inside modified namespace:
**$ podman unshare** *COMMAND*

To make the container UID owner of the directory on the host:
**$ podman unshare chown** *UID:UID* *DIR*

#### Running Containers as Systemd Services

To run a service in user space, unprivileged, use the **--user** flag
**$ systemctl --user start** *SERVICE*

When services are enabled with **--user**, services are automatically started only when the user sessions is started

**loginctl**: systemd login manager
**linger**: allows services enabled with **--user** to start even if there is no user session opened.

**$ loginctl enable-linger** *USER*

To check if a user has linger enabled:
**$ loginctl show-user** *USER*

To make a container start automatically at system boot:
A container file must be generated in **~/.config/systemd/user/**
Open a **new ssh** session with the unprivileged user

To generate a **systemd unit** file to start a container
**$ podman generate systemd --name** *MYCONTAINER* **--files**

A file **container**-*SERVICE*.**service** is created in the **~/.config/systemd/user/** folder

To enable the service in unprivileged user space:
**$ systemctl --user enable container**-SERVICE.service 

To get the status of the service container:
**$ systemctl --user status container**-SERVICE