
#### The Docker Architecture

- `docker`: The frontend client used to interact with `dockerd`: `docker ps, docker run`
- `dockerd` **Docker Engine**: The **high-level orchestration layer**. 
	- Parses CLI/API requests. All `docker` commands are essentially translated into API calls.
	- Manages images, networks, volumes
	- Provides the Docker API
- `containerd`: The **low-level container runtime**
	- Manages container lifecycle.
	- Talks to the OS via `runc`
- `runc`: Lower level container runtime. Executes containers using Linux `namespaces` and `cgroups`

`client --> dockerd --> containerd --> runc`

`ps -ef | grep dockerd`
```
root 1017 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

`dockerd` communicates with the `containerd` service, which must be running on the host.

`dockerd` can be manually started with options, or options can be specified in `/etc/docker/daemon.json`

```bash
dockerd --help
```

#### Docker Images

Containers are run from images

Search for an image

```bash
docker search debian
```

Pull an image

```bash
docker pull debian
```

List downloaded images

```bash
docker images
```

Remove images

```bash
docker image rm 'IMAGE' -f
```

Remove all images

```bash
docker rmi -f $(docker images -q)
```

#### Docker Containers

Containers are created from images. This is done by the `docker create` command. The only mandatory parameter is the name of the image used by the container.

```
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

##### Running Containers

Create and start a new container from an image. Downloads the image if not present. Running `docker run` again creates **another** container.

```bash
docker run [OPTIONS] 'IMAGE' 'CMD'
```

Created containers do not start automatically. To start an **existing** container:

```bash
docker container start [OPTIONS] "CONTAINER"
```

The container runs an associated command, in the case of `almalinux`, it is `/bin/bash` and then immediately exits. 

```
CONTAINER ID   IMAGE       COMMAND      STATUS              NAMES
f6097ce320aa   almalinux   "/bin/bash"  Exited (0) 3s ago   rng_mkn
```

The `docker attach` command  plugs your terminal into the container's **main process**. Terminating the process stops the container as well.

```bash
docker attach 'CONTAINER'
```

The `docker exec` command runs another process inside a **running container**

```bash
docker exec [OPTIONS] 'CONTAINER' 'CMD' [ARGS]
```

Options:

- `-i`: Interactive mode. Keep `stdin` open
- `-t`: Allocate a pseudo-terminal
- `-a`: Attaches your terminal to the container’s `stdout` and `stderr`, prints container's output to your terminal.
- `-d`: Run the container in the background
- `--label='LABEL'`: Assign a label

To copy files from the host to the container

```bash
docker container cp 'FILE' 'CONTAINER':'PATH'
```

Delete all containers 

```bash
docker stop $(docker ps -q)
```

As every process a container has its own PID. 

We can gather info about that process in the `/proc` filesystem. For example, let's have a look at the container's root directory as seen from the host.

```bash
sudo ls /proc/PID/root
```

```
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

##### Applying Filters and Formatting

To get the image name of a container

```bash
docker inspect 'CONTAINER_NAME' --format='{{.Config.Image}}'
```

Get the IPs of containers

```bash
docker network inspect bridge -f '{{range .Containers}}{{.Name}},{{println .IPv4Address}}{{end}}'
```

To filter out containers by label

```bash
docker container ls -a --filter --label="LABEL"
```

Search running containers by name

```bash
docker container ls -f name='CONTAINER_NAME"
```

#### Connecting to Docker

By default, the docker engine listens on local `/var/run/docker.sock` socket. To make it listen for incoming remote network connections and allow remote docker clients to connect to it:

Ports: 2375 (Insecure, no auth), 2376 (TLS, secure)

Open port 2375/2376 in the firewall

```bash
sudo firewall-cmd --add-port=2375/tcp --permanent
sudo firewall-cmd --reload
```

Add a drop-in configuration file for the docker service

```bash
mkdir -p /etc/systemd/system/docker.service.d/
```

`/etc/systemd/system/docker.service.d/override.conf`
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
```

`server`
```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Connect to a remote docker host

`client`
```bash
docker -H tcp://<remote_docker_host> 'DOCKER_COMMAND'
```

At this point basic connectivity is set up. Now, create a docker context

##### Docker Context

A Docker context is like a profile for telling the Docker CLI which daemon to talk to.

- `default`: Use this context to work with Docker locally
- Create SSH context when connecting to Docker on a remote VM via SSH
- Create a TCP context when connecting to Docker via TCP

```bash
docker context OPTIONS
```

- `show`: Show current context
- `ls`: List all contexts
- `use <name>`: Switch to a context
- `rm <name>`: Remove a context

Create a context. Assumes passwordless SSH or agent is set up for SSH connection.

```bash
docker context 'NAME' --docker "host={tcp|ssh}://user@IP[:2375]"
```

The docker client is connected to a remote daemon over TCP. 

```
Current context is now "docker-99-100"
[vagrant@docker docker2]$ docker ps
CONTAINER ID   IMAGE   COMMAND              STATUS          NAMES
a82528d5d999   f92480  "tail -f /dev/null"  Up 2 minutes    foo
```

Switching back to the `default` context shows no locally running containers. 

##### Other Connection Methods

Remote connections to the docker engine can be achieved, even if the docker daemon is still listening to its own unix socket. This is done by SSH local forwarding.

`client`
```bash
ssh -fN -L 2375:/var/run/docker.sock user@dockerhost
```

Now, connecting to the localhost on port 2375 make SSH forward the connection directly to the remote docker's socket `/var/run/docker.sock`. 

```bash
curl http://localhost:2375/containers/json
docker -H http://localhost:2375 ps
```

To make a raw connection to the docker engine without a docker client. Same as `docker ps`

```bash
curl http://<docker_host:2375/containers/json
```

##### Remote Docker Over TLS

SSH tunneling is convenient for small teams / personal use / DevOps admining one-off servers.
For large setups, TLS is more suitable.

TO-DO
#### Docker Volumes

Docker containers add a writable layer over the image layer to store modified information. For that, a storage driver is needed.

Docker containers store information in folders inside the `/var/lib/docker` directory

```bash
sudo find /var/lib/docker -name 'FILE'
```

```
/var/lib/docker/overlay2/da335e7...47a34b864fc878a686aa/diff/test_file.txt
```

Storage drivers available in docker

- `overlay2`
- `fuse-overlayfs`
- `btrfs and zfs`
- `vfs`

Get the active storage driver and backing filesystem

```bash
docker info
```

```
Storage Driver: overlay2
  Backing Filesystem: xfs
```

```bash
docker container inspect "CONTAINER"
```

```
 "GraphDriver": {
	"Data": {
		"LowerDir": "/var/lib/docker/overlay2/da335e7
		"MergedDir": "/var/lib/docker/overlay2/da335e723e1
		"UpperDir": "/var/lib/docker/overlay2/da335e72
		"WorkDir": "/var/lib/docker/overlay2/da335e72
```

- `GraphDriver`: Plug-in used by the `overlay` storage driver
- `LowerDir`: Base image read-only layer
- `UpperDir`: Writable container layer

Create a volume and assign a label

```bash
doker volume create "VOL" --label mode=prod
```

Search volumes by label

```bash
docker volume ls -f label=mode=prod
```

Format the volume list

```bash
docker volume ls --format "{{.Name}}: {{.Driver}}: {{.Mountpoint}}"
```

```
vol-1: local: /var/lib/docker/volumes/vol-1/_data
vol-2: local: /var/lib/docker/volumes/vol-2/_data
```

##### Ephemeral Volumes

Create a truly ephemeral (unnamed, on-the-fly) volume.

```bash
docker run -v /data alpine
```

- Docker creates a volume with a random name and mounts in at `/data`

##### Bind Mounts

> Bind mount can be set up on new containers only, cannot be added afterwards.


```bash
docker run -v /SOURCE_PATH/:/CONTAINER_PATH -name "CONT_NAME" -itd "IMAGE"
```

```bash
docker inspect
```

```
"Mounts": [
            {
                "Type": "bind",
                "Source": "/home/vagrant/VOLUMES",
                "Destination": "/VOLUMES",
```

The new volume `VOLUMES` has been created in the container.

```
[vagrant@docker ~]$ docker container attach "CONTAINER"
[root@4bd557a95d39 /]# ls
VOLUMES  afs  bin   opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

Create a file in the `/VOLUMES` dir in the container and it should appear in the host mount point.

Using the `--mount` parameter (recommended)

```bash
docker run --mount type=bind,source="SOURCE_DIR/",target="TARGET_DIR/" \
--name "CONT_NAME" -itd "IMAGE"
```

##### Deleting a Bind Mount on a Running Container

Bind mounts are **live references** to host paths. They track **inodes**, not paths. Deleting the host's path while the container is running breaks the container's reference to that **inode** and thus the mount.

Recreating the path on the host is not enough because the old mount reference is lost.

This can be observed by checking the state of the current container PID.  

Get the container PID

`host`
```bash
docker inspect -f '{{.State.Pid}}' "CONTAINER"
2388
```

`host`
```bash
sudo ls -lid /proc/2388/cwd
```

The `cwd` of the process is `symlinked` to the container's working directory.

```
22859 lrwxrwxrwx 1 root root 0 Aug 28 12:04 /proc/2388/cwd -> /var/www/html
```

The contents of a healthy bind mount can be listed.

If the directory on the host is deleted, the `cwd` of the process loses the link to the **inode**

```
22859 lrwxrwxrwx 1 root root 0 /proc/2388/cwd -> '/var/www/html (deleted)'
```

Restart the container to fix the mount.

> Renaming the mount directory on the host is automatically reflected in the container, so it doesn't break the mount.

##### Named Volumes

Docker volumes allow to share data between containers

Create a docker volume

```bash
docker volume create "VOL_NAME" [--label mode=prod]
```

List docker volumes

```bash
docker volume ls
```

Inspect a volume

```bash
docker volume inspect "VOL"
```

```
[
    {
        "CreatedAt": "2025-06-29T13:45:26+03:00",
        "Driver": "local",
        "Labels": {
            "mode": "prod",
        "Mountpoint": "/var/lib/docker/volumes/volume_one/_data",
        "Name": "volume_one",
        "Options": null,
        "Scope": "local"
    }
]
```

Create a volume `myvolume` if it does not exist, and mount it at `/data` in the container.

```bash
docker run -v myvolume:/data alpine
```

Filter volumes by label

```bash
docker volume ls -f label=mode=prod
```
##### tmpfs Volumes

Temporary volumes. Removed when container is stopped.

```bash
docker container run -it --tmpfs /"CONT_TMP_DIR" "IMAGE"
```

##### Sharing Volumes Between Containers

A container can inherit volumes, including bind mounts and named volumes from another container.

Run a new container and use the volume from the previous container

```bash
docker run --rm -it --volumes-from="CONTAINER" 'IMAGE'
```

- `--rm`: Automatically remove the container when it exits

To check where data is being stored on the host

```bash
docker container inspect c1 | grep -i source
```

##### Remote Volumes

When creating volumes, the `local` driver is used by default.

Install the `sshfs` plugin for docker

`docker client`
```bash
sudo docker plugin install vieux/sshfs --grant-all-permissions
```

List installed plugins

```bash
docker plugin list
```

`docker remote volume host`
```bash
[root@docker ~]$ ls /EXT_VOLUME/
check1  check2  check3  check4  check5
```

Create the docker volume

`docker client`
```bash
docker volume create \
--driver vieux/sshfs \
-o sshcmd='USER'@'IP':/EXT_VOLUME \
-o password='PASS' \
SSH_volume
```

Private keys can also be used if SSH passwords are disabled

```bash
docker volume create \
  -d vieux/sshfs \
  -o sshcmd=user@remotehost:/remote/path \
  -o identity=/root/.ssh/id_rsa \
  remote_volume
```

The remote volume is created. Can be inspected. Password is in plain-text!

```
DRIVER               VOLUME NAME
vieux/sshfs:latest   SSH_volume
```

```
[vagrant@docker ~]$ docker volume inspect SSH_volume
[
    {
        "CreatedAt": "0001-01-01T00:00:00Z",
        "Driver": "vieux/sshfs:latest",
        "Labels": null,
        "Mountpoint": "/mnt/volumes/b5a6a235c616483069bba424476d26eb",
        "Name": "SSH_volume",
        "Options": {
            "password": "Password1",
            "sshcmd": "root@192.168.89.100:/EXT_VOLUME"
        },
        "Scope": "local"
    }
]
```

Create a container and mount the remote volume

```bash
docker run --rm -it --name cont_ssh \
--mount source=SSH_volume,target=/vol_ssh \
almalinux
```

```
[root@e523f5b721f2 /]# ls vol_ssh/
check1  check2  check3  check4  check5
```

##### Deleting and Pruning Volumes

To remove unused volumes

```bash
docker volume prune
```

To remove a single volume

```bash
docker volume rm 'VOL'
```

#### Docker Networking

Communication between host and container is possible because docker automatically creates a network object that associates the `docker0` interface with containers

```
[root@docker ~]$ ip -br -4 a
lo               UNKNOWN        127.0.0.1/8
enp0s3           UP             10.0.2.15/24
enp0s8           UP             192.168.89.100/24
docker0          UP             172.17.0.1/16
```

```
[root@docker ~]$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
a0d0a524aa62   bridge    bridge    local
0c4dfb20c6ae   host      host      local
9e4d65c83585   none      null      local
```

- `bridge`: The default docker network. allows communication between container and host
- `host`: Removes network isolation — container shares the host’s network stack.
- `none`: No network interface. When complete isolation is needed.

Create a container with a host network. Container sees the host's interfaces.

```bash
docker run --rm -it --network=host busybox sh
```

```
/ # ip link
1: lo: 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3:
    link/ether 08:00:27:94:92:19 brd ff:ff:ff:ff:ff:ff
3: enp0s8:
    link/ether 08:00:27:17:85:45 brd 
4: docker0:
    link/ether b2:03:c1:a3:9c:6f brd ff:ff:ff:ff:ff:ff
29: veth02b1038@enp0s3: noqueue master docker0
    link/ether 5e:ea:81:88:a0:b7 brd 
```

`--network=none`: The container will show only its loopback address, i.e., isolated

##### Creating a New Network

Create a new docker network using the `bridge` driver

```bash
docker network create --driver bridge "NET_NAME"
```

Create a new network with specific address space

```bash
docker network create -d bridge --subnet 'NETWORK/MASK' 'NETWORK_NAME'
```

Connect a container to a network

```bash
docker run -d -it --network="NET_NAME" --name='CONT1' 'IMAGE'
docker run -d -it --network="NET_NAME" --name='CONT2' 'IMAGE'
```

Both containers should be able to communicate with each other


```bash
docker exec 'CONT1' ping 'CONT2'
```

```
PING cont2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.106 ms
```

Filter out containers belonging to a network and their IPv4 addresses.

```bash
docker network inspect 'NETWORK' -f '{{range .Containers}}{{.Name}},{{println .IPv4Address}}{{end}}'
```

```
co2,10.0.0.3/24
co1,10.0.0.2/24
```
##### Mapping Ports

```bash
docker run -d -p 'PUBLISHED_PORT':'TARGET_PORT' 'IMAGE'
```

Run `nginx` container with port forwarding. Requests coming on localhost:8080 will be directed to port 80 in the container

```bash
docker run -d -p 8080:80 nginx
```

`-p, --publish`: Forward host port 8000 to container port 80

##### DNS Resolution

> DNS-based Name Resolution is only enabled in user-defined networks!

When a container is attached to a **user-defined network** Docker automatically registers two names in the network's internal DNS.

- `db-app`: Name of the container
- `8e32dd48b55c`: Container id prefix (first 12 chars of the id)

`db-app container`
```
"Networks": {
	"app-network": {
		"Aliases": [],
		"Gateway": "172.18.0.1",
		"IPAddress": "172.18.0.2",
		"DNSNames": [    # These are the internally reachable names
			"db-app",
			"8e32dd48b55c"
		]
```

Additional host aliases can be specified

```bash
docker network connect --alias 'ALIAS' 'NETWORK' "CONTAINER"
```

Now the new alias is included in the internal DNS as well.

```
"Aliases": [
  "db",
],
"DNSNames": [
	"db-app",
	"db",
	"8e32dd48b55c"
]
```
#### Customizing Images

Create an image from a container

```bash
docker container commit [OPTIONS] 'CONTAINER' 'NEW_IMAGE_NAME'
```

Options:
- `--author 'AUTHOR`: Assign author metadata

##### Dockerfile

`Dockerfile`
```
FROM 'IMAGE':tag
RUN dnf update -y
```

- `RUN`: Every command creates a new layer.
- `WORKDIR /DIR`: Sets the container working directory. `/` is the default.

`Dockerfile`
```
COPY file.txt .
COPY DIR/ /DIR/
```

- `COPY` `source` `dest` from the build context (usually the local directory)
- ...into the current working directory in the image (set by `WORKDIR`, or `/` by default).

> Do not use wildcards when copying all files from a directory. Just specify the directory.

`Dockerfile`
```
EXPOSE 80
```

> Adding `EXPOSE 80` in the `Dockerfile` does not publish port. It is just for documentation. It has to be published with `-p`

The `ENTRYPOINT` instruction defines the **main command** that is always executed when a container starts, regardless of any arguments to `docker run` passed during runtime. Runs a specific program (e.g., web server, script, CLI tool). Makes the container behave more like an executable.

##### ENTRYPOINT and CMD

`Dockerfile`
```
ENTRYPOINT ["executable", "arg1", "arg2"]
```

- No shell involved
- Arguments passed to `docker run` are **appended** to this command.

`Dockerfile`
```
FROM alpine
ENTRYPOINT ["echo"]
```

```bash
docker build -t say .
docker run --rm say "Hello, world!"
```

```
[vagrant@docker ~]$ docker run --rm say "hello world!"
hello world!
```

The `CMD` instruction specifies the **default command and arguments** to run when a container starts, **unless they are overridden** in `docker run`.

- Sets the **default executable or arguments** for the container
- Acts as a fallback if the user doesn't provide a command

`Dockerfile`
```
CMD ["executable", "arg1", "arg2"]
CMD ["nginx", "-g", "daemon off;"]
```

- Runs the command without a shell

Shell form

`Dockerfile`
```
CMD command arg1 arg2
```

Equivalent to:

```
/bin/sh -c "command arg1 arg2"
```

`Dockerfile`
```
CMD echo "Hello from container"
```

Start the container with no command and arguments

```bash
docker run hello
```

It defaults to the `CMD` instruction

```
Hello from container
```

**CMD** as default arguments for **ENTRYPOINT**

`Dockerfile`
```
ENTRYPOINT ["echo"]
CMD ["Hello from container"]
```

```bash
docker run hello
```

```
Hello from container
```

The CMD arguments can be overridden

```bash
docker run hello "Goodbye!"
```

```
Goodbye!
```

Directory structure

```
.
├── Dockerfile
└── file.txt
```

##### Build the image

```bash
docker build [-t 'IMAGE_NAME'] .
```

Or manually specify the `Dockerfile`

```bash
docker image build -f Dockerfile .
```

The `.` is the **build context** - it should be the root directory containing all files referenced in the `Dockerfile` (`COPY`, `ADD`, etc.). 

Retag an existing `<none>` image

```bash
docker tag "IMAGE_ID" "IMAGE_NAME"[:VERSION]
```

Clean up untagged images

```bash
docker image prune
```
#### Logging in Docker

Docker has a built-in mechanism that captures the output of containers -- what the app writes to `stdout` and `stderr`.

- All output from a container (e.g. `console.log`, `print`, `echo`, etc.) goes to the container's logs.
- Docker stores this using a logging driver (default: `json-file`)

```
Plugins:
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
```

```bash
docker logs [OPTIONS] 'CONTAINER'
```

Options:

- `-f`: Follow. Like `tail -f`
- `--timestamps`

To set a logging driver when running a container

```bash
docker run --log-driver=syslog 'CONTAINER'
```

Set default log driver

`/etc/docker/daemon.json`
```
{
        "log-driver": "journald"
}
```

Restart docker and run `docker info | grep -i logging`

```
Logging Driver: journald
```

This will send container `stdout` and `stderr` to the host's `journald`

**Log rotation**. This keeps up to 30 MB of logs per container.

`/etc/docker/daemon.json`
```
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Filter out `journald` entries by container name

```bash
journalctl CONTAINER_NAME='CONTAINER'
```

#### Archiving Containers

When running a container, it is created from an image, and:
- Any changes (file edits, installs, config changes) happen inside the container's writable layer.
- When a container is deleted, changes are lost.

To commit a container to a new image

```bash
docker container stop "CONTAINER"
docker container commit "CONTAINER" "NEW_IMAGE"
```

##### Docker Save

Saves a full docker image (with layers and history). Useful for backup, registry-free transfers.

```bash
docker save -o 'MY_IMAGE'.tar 'MY_IMAGE':tag
```

To load an image from a tar file

```bash
docker image load -i "IMAGE".tar
```

##### Docker Export

- Exports a container's flattened filesystem
- Useful for extracting raw data or creating a new image from a snapshot
- Does not include image layers, metadata or volumes

```bash
docker export -o 'CONTAINER'.tar 'CONTAINER'
```

To import an image. CMD and ENV are lost during export, need to be set anew. 

```bash
docker import 'IMAGE'.tar [--change "CMD /bin/sh"] 'NEW_IMAGE_NAME'
```

This creates a new image, but will lose docker-specific config like `ENV`, `CMD` and history

To export the entire container filesystem

```bash
docker export 'CONTAINER' > 'CONTAINER'.tar
```

#### Building Images with heredoc

Create a simple Docker image from a `Dockerfile` on the fly

```bash
docker image build -t alp-htop - << EOF
FROM alpine
RUN apk --no-cache add htop
EOF
```

- `-`: Tells Docker to read the `Dockerfile` from `stdin`

#### Best Practices and Troubleshooting

Add labels to a docker container

`Dockerfile`
```
FROM nginx
LABEL vendor="DevOps International"
LABEL version="1.0"
LABEL description="Web App"
COPY html/ /usr/share/nginx/html/
```

Filter out the image labels only

```bash
docker image inspect --format='{{json .Config.Labels}}' labels
```

Multiple labels can be added like this 

`Dockerfile`
```
FROM nginx
LABEL key="value" \
	key="value" \
	key="value"
COPY ...
```

#### Distributed Apps

Linking containers (Legacy)

Run the DB container

```bash
docker run -d --name "DB_CONT" -e MYSQL_ROOT_PASSWORD=Parolka-12345 "DB_IMAGE"
```

```bash
docker run -d --name "WEB_CONT" -p 8080:80 -v "DIR":/var/www/html --link "DB_CONT":"DB_HOSTNAME" "WEB_IMAGE"
```

##### Isolated Networks

Create a new network

```bash
docker network create --driver bridge app-network
```

Create and start the DB container

```bash
docker run -d --net app-network --name db -e MYSQL_ROOT_PASSWORD=Parolka-12345 img-db
```

Create and start the web container

```bash
docker run -d --net app-network --name web -p 8080:80 -v $(pwd)/web:/var/www/html img-web
```

##### Docker Compose

`docker-compose.yaml`
```
services:
  web:
    build: .
    ports:
      - 8080:80
```

Build the image

```bash
docker compose build
```

Create and start the container from the image

```bash
docker compose up [-d]
```

```
[+] Running 2/2
Attaching to web-1
web-1  | 10.0.2.2 - - [15/Aug/2025:19:41:06 +0000] "GET / HTTP/1.1" 200 381
```

```
docker compose logs
```

Example of a distributed app

`docker-compose.yaml`
```
services:  
  web:  
    build:  
      context: .  
      dockerfile: Dockerfile.web  
    ports:  
      - 8080:80  
    volumes:  
      - "/home/vagrant/mybgapp/web:/var/www/html:ro"  
    networks:  
      - app-network  
    depends_on:  
      - db  
  
  db:  
    build:  
      context: .  
      dockerfile: Dockerfile.db  
    networks:  
      - app-network  
    environment:  
      MYSQL_ROOT_PASSWORD: 12345  
  
networks:  
  app-network:
    external: true    # means that the app-network is already existing
				        otherwiwse, it is created automatically
```



Environment variables can be exported to external `.env` file

`.env`
```
PROJECT_ROOT=/home/vagrant/mybgapp/web
DB_ROOT_PASSWORD=12345
```

`docker-compose.yaml`
```
volumes:  
      - "${ROJECT_ROOT}:/var/www/html:ro" 
...
environment:  
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD}"
```

#### Create a Local Registry

Insecure. For testing purposes only!

Run the registry container

```bash
docker run -d -p 5000:5000 --name registry registry:2
```

Tag the image for the local registry

```bash
docker tag "IMAGE":latest "IP":5000/"IMAGE":latest
```

Push the image:

```bash
docker push "IP":5000/"IMAGE":latest
```

On every node that is going to be pulling the image

`/etc/docker/daemon.json`
```
{
          "insecure-registries": ["REGISTRY_IP:5000"]
}
```

Restart Docker

