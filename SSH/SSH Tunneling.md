#### SSH Connection Forwarding and Tunneling

When opening an SSH tunnel, the `start of the tunnel` is considered the point where the command, initiating the tunnel is executed. 

##### Local Port Forwarding

SSH local port forwarding allows you to securely tunnel traffic from a local port on your machine to a remote destination through an SSH server. This feature is useful when you want to access services on a remote server that are not directly accessible due to network restrictions or security concerns.

In this case we have a web server that is allowing http connections on port 80 to the jump host only. With local port forwarding, this will open a tunnel from the local host to the web server through the jump host and forward local connections on 8080 to port 80 on the webserver.

``` bash
ssh -L 8080:"WEB_SERVER":80 "USER"@"JUMP_HOST"
```

```
Local connections to LOCALHOST:8080 forwarded to remote address 10.0.2.153:80
debug1: Local forwarding listening on ::1 port 8080.
```

```
debug1: channel 3: free: direct-tcpip: listening port 8080 for 10.0.2.153 port 80, connect from 127.0.0.1 port 57964 to 127.0.0.1 port 8080, nchannels 4
debug1: Connection to port 8080 forwarding to 10.0.2.153 port 80 requested.
```

```
curl localhost:8080
This is WebServer!
```

Example: Connect to a remote docker daemon that is not open for remote connections. SSH is piping traffic directly into the remote UNIX socket.

```bash
ssh -fN -L 2375:/var/run/docker.sock 'USER'@'REMOTE-DOCKER'
```

- `2375`: Local port
- `/var/run/docker.sock`: Remote docker socket

`client`
```
State                  Local Address:Port                 Peer Address:Port  LISTEN                     127.0.0.1:2375                         0.0.0.0:*
```

```bash
curl http://127.0.0.1:2375/containers/json
```

This, is a secure way of connecting to remote daemons without exposing them to the network.

Jump hosts can be used

``` bash
 ssh -L 8080:localhost:80 -J "JUMP_HOST" USER@"WEB_SERVER"
```

To simply SSH into a remote server with restricted access to a jump host only

``` bash
ssh -J "JUMP_HOST" "USER@REMOTE_HOST"
```

##### Dynamic Port Forwarding

To create a SOCKS proxy

``` bash
ssh [-D "LOCAL_PORT"] -J "JUMP_HOST" "USER@TARGET_HOST"
```


Same effect can be achieved by using `ProxyCommand`
`ProxyCommand ssh -W %h:%p "JUMP_HOST"` `

``` bash
ssh dc
```

```
[kimchen@dc ~]$ w
 11:52:37 up  2:45,  1 user,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
kimchen  pts/1    10.0.2.24        11:51    5.00s  0.12s  0.03s w
```

10.0.2.151 host shows that the incoming SSH connection comes from the jump host

#### SSH Tunnels

TCP Forwarding uses:
* secure connection to a mail server
* going through firewalls

* **Local Port Forwarding**: create a local port that is connected to a remote service
* **Remote Port Forwarding**: make a port available on a local host to users on the internet

`sshd_config`
```
GatewayPorts yes | no | clientspecified
```

The `GatewayPorts` directive controls access to a forwarded port. Applies only to the starting point of the tunnel. Binds to the loopback interface by default.

`no`: restricts SSH forwarded ports to the loopback interface only

```
Netid       State        Local Address:Port             Peer Address:Port
tcp         LISTEN           127.0.0.1:8088                        *:*
```

`yes`: allows remote hosts to access the forwarded port

```
Netid       State        Local Address:Port             Peer Address:Port
tcp         LISTEN                   *:8088                        *:*
```

- `-f`: run the SSH session in the background
- `-N`: Do not execute a remote command. Useful for just forwarding ports

##### Reverse SSH Tunneling

- Client: 10.0.2.66
- Server: 10.0.2.153

Reverse tunnels allow external clients to access services on a remote server as if they were local to that server. This can be useful for scenarios where direct access to the server is restricted or when services need to be accessed securely from external networks.

Once the reverse tunnel is established, incoming connections to the forwarded port on the jump host will be forwarded through the tunnel to the destination port on the server. This allows external clients to connect to the jump host's forwarded port and reach the web server as if they were connecting directly to the web server's destination port.


``` bash
ssh -fNR ["CLIENT":]"SPORT":"SERVER":"DPORT" "USER@CLIENT"
```

```
ssh -fNR 10.0.2.66:2222:10.0.2.153:22 kimchen@10.0.2.66
```

`debug1: remote forward success for: listen 10.0.2.24:2222, connect 10.0.2.153:22`

`-R 10.0.2.66:2222:10.0.2.153:22` : Any connection made on port `2222` on `10.0.2.66` will be forwarded to port `22` on `10.0.2.153`. Thus, `ssh 10.0.2.66 -p 2222` will ssh into `10.0.2.153`. `GatewayPorts` on `10.0.2.66` must be set to `yes`., otherwise it will listen on the loopback address only for incoming connections on port 2222.

The syntax is `-R [bind-address:]port:host:hostport`


The config file would look like this

```
Host gateway
        Hostname 10.0.2.24
        Port 22
        User kimchen
        RemoteForward 10.0.2.24:8090 10.0.2.153:80
```

`RemoteForward`: same as `ssh -R 10.0.2.24:8090:10.0.2.153:80`

It's commonly used for scenarios where direct access to a remote machine is not possible due to firewall restrictions or NAT configurations.

To make a tunnel persistent across reboots, create a service file

`/etc/systemd/system/docker-ssh-tunnel.service`
```
[Unit]
Description=Persistent SSH tunnel to remote Docker daemon
After=network.target

[Service]
Type=notify
Environment=/home/vagrant
User=vagrant

Restart=on-failure

ExecStart=/usr/bin/ssh -i /home/vagrant/.ssh/id_rsa -N -L 2375:/var/run/docker.sock vagrant@dockerhos1t

[Install]
WantedBy=multi-user.target
```