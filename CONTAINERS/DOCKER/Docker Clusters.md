
Ports:
- `2377/tcp` - cluster management
- `7946/tcp,udp` - communication between nodes
- `4789/udp` - for overlay network traffic

#### Cluster Creation and Joining

Initialize the first node in the cluster

`manager`
```bash
docker swarm init --advertise-addr "MANAGER IP"
```

A join token is generated. To recover the token

```bash
docker swarm join-token -q worker
```

Join a worker to the cluster

`worker`
```bash
docker swarm join \
--token "TOKEN" \
--advertise-addr "WORKER_IP" "MANAGER_IP":2377
```

`manager`
```
dockerd[984]: ... msg="worker ilkp...7p0rm was successfully registered" method="(*Dispatcher)
dockerd[984]: ... msg="Node 995d5f7a2c72/192.168.99.102, joined gossip cluster"
dockerd[984]: ...msg="Node 995d5f7a2c72/192.168.99.102, added to nodes list"
```

List nodes 

`manager`
```bash
docker node ls
```

> Number of managers must be kept odd. Min of three is required for fault tolerance. Should be spread across different hosts and even switches.

It is not uncommon to put the manager in `drain` state so as not to accept tasks
#### Services

Deploy a service

`manager`
```bash
docker service create [--mode "MODE" or --replicas "NUM"] --name hello nginx
```

Deployment modes:
- `global`: Run one task on every active node in the cluster. If a drained node is brought back to active, the service starts immediately on that node.
- `replicated` (default): Sets a number of replicas. 
`
```
deploy:
  mode: replicated
  replicas: 3
---
deploy:
  mode: global
```

List and inspect

```bash
docker service [OPTIONS]
```

- `ls`: List services
- `inspect --pretty "SERVICE"`: Service info
- `ps "SERVICE`: See which nodes a service is running on

Replicas 1 means the service is running only on the host. To distribute it among the worker nodes:

```bash
docker service scale "SERVICE"="NUMBER_OF_NODES"
```

```bash
docker service ps "SERVICE"
```

```
ID             NAME       IMAGE           NODE              CURRENT STATE
cb3uwlnk2ifh   pinger.1   alpine:latest   docker1.do1.lab   Running         
6uytefvjifbh   pinger.2   alpine:latest   docker2.do1.lab   Running         
wdqu7v7bi9rc   pinger.3   alpine:latest   docker3.do1.lab   Running         
```

ID is **swarm task ID**, not container ID

> Replicate only **stateless** services


##### Run Services with Published Ports

```bash
docker service create --name app --publish published=8080,target=80 "IMAGE"
```

##### Service with custom data. 

Obtain container hostnames

`index.php`
```php
<?php
print "<h3>Hello Docker Swarm!</h3>\n";
if (getenv('APP_MODE')) print "Running in ".getenv('APP_MODE')." mode.<br />\n";
print "<hr />\n";
print "<small><i>Served by: <b>".gethostname()."</b></i></small>\n";
?>
```

Create the service with bind mount. Path must exist on all nodes.

```bash
docker service create --name app \
--publish published=8080,target=80 \
--replicas 5 \
--mount type=bind,src=/home/vagrant/app/web,dst=/var/www/html,readonly \ "IMAGE"
```

Environment variables can be set with:

```bash
--env APP_MODE=gasoline
```

```
[vagrant@docker1 app]$ curl http://docker1.do1.lab:8080
<h3>Hello Docker Swarm!</h3>
Running in gasoline mode.<br />
<hr />
<small><i>Served by: <b>19940b8f23f2</b></i></small>
```

##### Using Docker Config custom data

Swarm configs let you **store config files in the Swarm manager** and mount them into containers when services run.

```bash
docker config create 'NAME' 'STRING or BINARY'
```

```bash
docker config create custompage web/index.php
```

Or from `stdin`

```bash
echo "key=value" | docker config create 'NAME' -
```

Use config in a service

```bash
docker service create --name app \
--publish published=8080,target=80 \
--replicas 3 \
--config src=custompage,target=/var/www/html/index.php \
'IMAGE'
```

- `source=myconfig` → the Swarm config object name
- `target=/etc/myapp/app.conf` → path inside the container where it gets mounted

> Configs are immutable. Create a new config and update the service to use it.

To update a service to use a new config:

```bash
docker service update --config-rm custompage \
--config-add source=newpage,target=/var/www/html/index.php \
'SERVICE'
```

##### Secrets

Docker Swarm does not recognize the `.env` file as `docker compose up` does, therefore variables, like secrets are best used with `docker secret`. 

Create a docker secret from `stdin`

```bash
echo "SECRET" | docker secret create 'NAME' -
```

Or from a file

```bash
docker secret create 'NAME' 'FILE'
```

Secrets are stored in containers in `/run/secrets/"SECRET_NAME"` folder

##### Docker Compose with Swarm

Example. Volume path must exist in every node.

`docker-compose-swarm.yaml`
```
services:
  web:
    image: 192.168.99.101:5000/bgapp-web
    deploy:
      replicas: 3
    ports:
      - target: 80
        published: 8081
        protocol: tcp
        mode: host
    volumes:
      - "/home/vagrant/bgapp/web:/var/www/html:ro"
    networks:
      - app-network

  db:
    image: 192.168.99.101:5000/bgapp-db
    networks:
      - app-network
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
    secrets:
      - db_root_password

secrets:
  db_root_password:
    external: true # tells docker swarm to use an existing secret
networks:
  app-network:
```

Deploy

```bash
docker stack deploy -c docker-compose-swarm.yaml "STACK"
```

`bgapp` will be the namespace of the service. The actual service names will be `bgapp_web` and `bgapp_db`.

Bring down the cluster

```bash
docker stack rm "STACK"
```
##### Constraints

Constraints define which nodes a service can run on. Multiple constraints can be set on a service with the final result being the intersection of all of them.

To constraint a service to run on nodes with specific labels:

```bash
docker service create --constraint "node.labels.env == dev" --name ...
```

##### Stopping Services

Docker swarm does not have means to stop a service. It assumes that it is no longer needed, thus it can only be removed.

```bash
docker service rm "SERVICE"
```

To work around this, set the number of replicas to 0

```bash
docker service update --replicas 0 'SERVICE'
```

#### Load-Balancing

Docker swarm automatically provides **service discovery** and **load balancing** when a service is created.

1. **Embedded DNS**
	- Every service gets a DNS name `<service_name>` on the network it is attached to.
	- Example: if a service named `db` is deployed, any container on the same overlay network can connect to `db:3306`
2. **Virtual IP (VIP)**
	- That DNS resolves to a single VIP
	- Example: `db` --> `10.0.0.2`
3. **IPVS Load Balancing**
	- The **VIP** is backed by **IPVS** (Linux Kernel L4 load balancer)
	- **IPVS** keeps a list of all service tasks (containers)

For published ports, there are two modes:

1. **Routing Mesh (ingress load balancing)** (default)
	- LB is handled by **IPVS**
	- Any node in the swarm will accept requests on the published port
	- That node doesn't need to be running the container, it will redirect to a node that does.
	- Connections are distributed round-robin across tasks.
	- `http://any-node:8080` in the Swarm can be hit, serving as a front-end.
	- Useful for web apps and general services.
2. Host:
	- No routing mesh, no load balancing.
	- The published port is bound **only to the node(s) actually running the task.**
	- Needs to be complemented by an external LB
	- Best for high-throughput, latency-sensitive workloads (databases, streaming, HPC)


```
services:
  web:
    image: nginx
    ports:
      - target: 80
        published: 8080
        protocol: tcp
        mode: ingress|host <---
```

##### Service Discovery:

1. **VIP mode** (default). Virtual IP mode.
	- DNS resolves to a **single VIP**
	- VIP load balances across all tasks

2. **DNSRRR** (DNS Round Robin):
	- DNS returns all task IPs
	- Load balancing is handled by the app
	- Useful for stateful services
	- Cannot publish ports and thus be used in `ingress` mode.

Controlled by the `endpoint_mode` directive

```
services:
  web:
    image: nginx:alpine
    deploy:
      replicas: 3
      endpoint_mode: dnsrr   # 👈 force DNSRR instead of VIP
```

Architecture Diagram

```
                 ┌───────────────────────────┐
                 │        HOST / USER        │
                 │    curl http://:8080      │
                 └─────────────┬─────────────┘
                               │
                        (published port)
                               │
                     ┌─────────▼─────────┐
                     │   Frontend (VIP)  │
                     │   web service     │
                     │   ports: 8080:80  │
                     └───────┬───────────┘
                             │
         ┌───────────────────┴───────────────────┐
         │                 overlay network       │
         │         (internal service discovery)  │
         └───────────────────┬───────────────────┘
                             │
         ┌─────────────┐     │     ┌─────────────┐
         │   db_rr.1   │◄────┼────►│   db_rr.2   │
         │ 10.0.5.15   │     │     │ 10.0.5.16   │
         └─────────────┘     │     └─────────────┘
                             │
                          ┌───────┐
                          │db_rr.3│
                          │10.0.5.17
                          └───────┘

```

- **Frontend** (`web`, **VIP** mode)
	- Exposes port **8080** to the host
	- Swarm load balancer spreads incoming requests across replicas
- **Backend** (`db_rr`, **DNSRR** mode)
	- Not exposed to the host
	- When the `web` service resolves `db_rr`, it gets all 3 replica IPs.
	- Swarm does not do load balancing, i.e., `web` (or a proxy) decides which replica to talk to(round robin, sticky, etc.)

**DNSRR**

The service name `db` resolves to all task IPs, not a single VIP. In this case the client decides which IP to pick.

```
/ # nslookup db
Server:         127.0.0.11
Address:        127.0.0.11#53

Non-authoritative answer:
Name:   db
Address: 10.0.5.4
Name:   db
Address: 10.0.5.3
Name:   db
Address: 10.0.5.2
```

```
curl -vv http://db
```

```
14:22:17.178460 [0-0] * Host db_rr:80 was resolved.
14:22:17.185886 [0-0] * IPv4: 10.0.5.15, 10.0.5.17, 10.0.5.16 <----
14:22:17.187854 [0-0] * [SETUP] added
14:22:17.189061 [0-0] *   Trying 10.0.5.15:80...
```

**Virtual IP**

Frontend. The service name resolves to a single VIP. Load balancing is handled by Swarm.

```
Name: db_vip
Address: 10.0.0.5   # VIP, not individual task IPs
```

`docker-compose-swarm.yaml`
```
services:
  db_vip:
    image: nginx:alpine
    deploy:
      replicas: 3
      endpoint_mode: vip       # default, VIP load balancing
    networks:
      - appnet

  db_rr:
    image: nginx:alpine
    deploy:
      replicas: 3
      endpoint_mode: dnsrr     # DNS Round Robin
    networks:
      - appnet

  web:
    image: alpine
    command: ["sh", "-c", "sleep 3600"]
    networks:
      - appnet

networks:
  appnet:
    driver: overlay
```

#### Node Maintenance

```bash
docker node update --availability "STATE" "NODE-ID"
```

States:
- `pause`: Prevents the scheduler from assigning new tasks to the node. Existing tasks continue to run.
- `drain`: Do not assign new tasks. Running containers are stopped and moved to other available nodes. Useful when upgrading a node.
- `active`: The node is ready to accept tasks.

> Tasks are only assigned when a new scheduling event, such as starting a service, happens, so reactivating a node might not immediately bring moved containers back.

If a node must be put into maintenance mode (drained) it transfers the containers to other nodes.

```bash
docker node update --availability drain "NODE HOSTNAME'
```

```
Availability:          Drain
```

Inspect a node

```bash
docker node inspect --pretty "NODE HOSTNAME"
```

Bring the node back to active state

```bash
docker node update --availability active "NODE"
```

> Draining a node moves tasks away, reactivating it doesn't automatically move them back
> Swarm only schedules tasks according to **desired state, placement and available resources**.

To redeploy tasks

```bash
docker service update --force "SERVICE"
```

Group nodes with labels

```bash
docker node update --availability --label-add 'KEY=VALUE' 'NODE'
```
##### Removing Nodes

First drain the node to move running containers elsewhere

Leave the swarm

```bash
docker swarm leave
```

> Leaving managers must first be demoted to workers. Quorum must be maintained!

Once the node status has been marked `Down`, it can be removed from the swarm

```bash
docker node rm [--force] "NODE"
```

#### Updating Services

A **rolling update** is when Swarm updates a service gradually, one task at a time (or batches), instead of stopping everything at once. This ensure 0 downtime - users keep being served while new constraints replace old ones.

```bash
docker service update OPTIONS
```

To update an image

```bash
docker service create --name web --replicas 3 nginx:1.24
```

```bash
docker service update --image nginx:1.25 web
```

Update options:

- `--update-parallelism 1`: Update 1 task at a time - default. `0` - all at once. SQL
- `--update-delay 10s`: Wait 10 seconds between each update

Control what happens if tasks are failing during the rolling update

- `--update-max-failure-ratio [RATIO]`: Controls failure ratio tolerance
	- number of failed tasks / total number of tasks updated.
		- `0.0`: Zero failure tolerance (default)
		- `0.5`: Up to half of the tasks can fail before rollback/pause
		- `1.0`: All can fail (not very useful)
- `--update-failure-action rollback`: Rollback to previous version if failed ratio is reached.

If updating breaks the service, it can rolled back

```bash
docker service rollback 'SERVICE'
```

If an image is not found on a node, it will be pulled from the registry, slowing down the process. Make sure to have the images everywhere prior to updating.

#### Swarm and Databases

> Do not scale DB containers naively in Swarm - always rely on DB-native clustering. Swarm handles container orchestration, not replication logic.

DB containers are deployed as separate services and load balancing is done externally


#### Disaster Recovery

##### Restarting the Cluster

Shutdown the workers first, then the managers. Start the managers first, then the workers. Managers must have static IPs.

##### Backup and Recovery

- swarm state data
- application data

##### Backing up the Swarm

Each manager keeps the cluster information in:

`/var/lib/docker/swarm/raft`

Back the `raft` directory.

##### Recovering a Swarm

1. Stop Docker
2. Copy the `raft` data back to `/var/lib/docker/swarm/raft`. 
3. Start Docker with:

```bash
docker swarm init --force-new-cluster --advertise-addr manager
```

##### Backing up Services

The `raft` database contains service info.

`Dockerfiles` and `docker compose` files should be in version control.