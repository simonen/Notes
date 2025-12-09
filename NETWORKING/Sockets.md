
#### Socket Relaying 

Package: `socat`: Bidirectional data relay between two data channels

`socat` is a simple way of forwarding between two end-points (TCP, UDP, UNIX sockets)

Open a bi-directional channel between point A and point B

`server`
```bash
 socat [-d -d -d] TCP-LISTEN:"LPORT",fork UNIX-CONNECT:"R_SOCKET" &
```

`server`
```
 socat [-d -d -d] TCP-LISTEN:2375,fork UNIX-CONNECT:/var/run/docker.sock &
```

- `,fork`: Spawn a new child process for each incoming connection, Keeps the listener open for new connections., How `sshd` works by default.

`-d`: Verbosity level

```
State            Local Address:Port                       Peer Address:Port 
LISTEN                 0.0.0.0:2375                               0.0.0.0:*
```

> Now the machine is listening on all incoming requests on port 2375 and forwards them to the UNIX socket. This is unencrypted and unsecure!

```
socat[8990] N listening on AF=2 0.0.0.0:2375
socat[8990] N accepting connection from AF=2 192.168.99.1:51872 on AF=2 192.168.99.100:2375
socat[8990] N forked off child process 8991
socat[8990] N listening on AF=2 0.0.0.0:2375
socat[8991] N opening connection to AF=1 "/var/run/docker.sock"
socat[8991] N successfully connected from local address AF=1
...
exiting with status 0
```

Each forked child process (like `[8991]`) is responsible for one client connection
The parent process \[8990] keeps listening for new connections

If `fork` is not specified, no new processes are created, which means that only one connected client can be handled at a time, when the task has been executed, the server `socat` process terminates.

Now the client can connect to the remote UNIX socket on `localhost:2375`. The server can receive input and return responses.
##### Sending Commands 

A command can be send to the listening server, which will execute it and return output back. This is not safe as a full bash shell is given.

`server`
```bash
socat -d -d TCP-LISTEN:12345,reuseaddr,fork SYSTEM:'bash -i'
```

`SYSTEM:'bash -i'`: Each command sent gets an interactive bash shell

Send a command from a client

`client`
```bash
echo "ls" | socat - TCP:"REMOTE":12345
```

`client`
```
bunar.txt
outfile
received_file.txt
```

The server receives the connections, starts a new bash shell, executes the command and exits the shell session.

`server`
```
2025/08/26 13:27:54 socat[9106] N forked off child process 9115
2025/08/26 13:27:54 socat[9106] N starting data transfer loop with FDs [6,6] 
2025/08/26 13:27:54 socat[9106] I executing shell command "bash -i"
vagrant@docker:~$ ls
vagrant@docker:~$ exit
2025/08/26 13:27:54 socat[9106] I exec'd process 9115 on socket 1 terminated
2025/08/26 13:27:54 socat[9106] I waitpid(): child 9115 exited with status 0
2025/08/26 13:27:54 socat[9106] N exiting with status 0
```

`-d`: A single d will output simply the received command
##### Transferring Files 

This is a **raw** transfer. It does not encrypt, compress and resume transfers. 

`server`
```bash
socat -u TCP-LISTEN:12345,reuseaddr,fork OPEN:received_file.txt,creat
```

`client`
```bash
socat -u OPEN:<file_to_send> TCP:<remote_host>:12345
```

- `-u`: Unidirectional.
- `,fork`: Spawn new child processes for new incoming connections
- `,reuseaddr`: Keep the main process alive if `,fork` not specified and established connection is closed

Data sent from a client can be piped directly to the server's pseudo-terminal

`server`
```bash
socat -d TCP-LISTEN:12345,reuseaddr,fork FILE:$(tty)
```

#### SOCKS Proxies

`ssh`, `microsocks`, `dante`, `shadowsocks`

SOCKS proxies are very versatile network tools. They act as a **general-purpose proxy** for TCP and sometimes UDP traffic, not limited to web browsing.  

- SOCKS proxies relay any TCP connections from a client to a target server
- Operate on the Transport and Session layer. 
- This lets you route traffic through another machine, effectively hiding your client's IP, access otherwise unreachable private targets

Set up a SOCKS5 forward proxy to forward TCP traffic

##### SSH Socks5 Proxy

SSH SOCKS Proxies target a single destination through an SSH tunnel.

```bash
ssh -v -D 1080 -N user@localhost
```

```bash
ss -4tlnp
```

```
State   Local Address:Port  Peer Address:Port  Process
LISTEN        0.0.0.0:1080          0.0.0.0:*  users:(("ssh",pid=17007,fd=4))
```

`lsof -i :1080`
```
COMMAND   PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
ssh     17007 vagrant    4u  IPv4 217010      0t0  TCP *:socks (LISTEN)
```

```
debug1: Local connections to LOCALHOST:1080 forwarded to remote address socks:0
debug1: channel 0: new port-listener [port listener] (inactive timeout: 0)
debug1: Local forwarding listening on 127.0.0.1 port 1080.
```

- `-N`: Do not open a shell session

Test connection 

```bash
curl --socks5 "PROXY":1080 http://"WEB_SERVER":PORT
```

```
channel 1: free: direct-tcpip: listening port 1080 for 192.168.99.101 port 80, connect from 10.0.2.2 port 58676 to 10.0.2.15 port 1080
```

The connection to the webserver goes through the proxy now.

##### Microsocks Proxy Server

https://github.com/rofl0r/microsocks

`Microsocks` is a lightweight SOCKS5 (TCP only) server.

Clone the project and compile the binary:

```bash
sudo yum install gcc make git -y
git clone https://github.com/rofl0r/microsocks.git
cd microsocks
make
sudo cp microsocks /usr/local/bin/
```

Start the server

```bash
microsocks -i 0.0.0.0 -p 1080
```

```
microsock 17808 vagrant  IPv4 229852  TCP *:socks (LISTEN)
microsock 17808 vagrant  IPv4 230579  TCP cent..lab:socks->_gateway:58946 (ESTABLISHED)
```

More options

```bash
microsocks -1 -q -i "LISTENIN_IP}" -p "PORT" \
-u 'USER' -P "PASS" -b "OUTGOING_IP" -w "IP1,IP2..."
```

- `w`: whitelist of comma-separated IPs

##### Dante 

Dante is a SOCKS5 server that supports TCP and UDP, as well as more authentication methods and rules

```bash
dnf install -y dante-server
```

Structure of `sockd.conf`

```
1. Server settings
2. Rules {socks, client}
3. Routes
```

##### Dante SOCKS Proxy Flow

```
(Client Req) -> [client rules check] -> SOCKS handshake -> [socks rules check] -> (Target IP/Port)
```

Dante has two layers of access control:

**Client rules** (rules on incoming requests)
- Matches incoming connections against the rules.
- Decide who even is allowed to talk to the SOCKS server

**SOCKS Handshake**:
- if the client passed the `client` stage, Dante negotiates the SOCKS version and authentication method. 
- This is where `pam` or `username` kicks in.

**Socks Rules** (rules on outgoing requests to destination)
- Evaluated after SOCKS negotiation and successful authentication.
- Decides what kind of proxy requests are allowed to leave the outgoing interface

`/etc/socksd.conf`
```
2. Rules Evaluated in top-down order. First match wins. ACL

# Rule 1 Allow from: to
pass {}

# Rule 2 Allow from: to
pass {}

# Rule 3 Block everything else
block {}
```

Client Rules

```
client pass { from: 192.168.0.0/24 to: 0.0.0.0/0 }
client block { from: 0.0.0.0/0 to: 0.0.0.0/0 }
```

Socks rules 

`Socks rules`
```
socks pass {
	from: ... to: ...
	method: authmethod 
}
socks block {from: 0.0.0.0/0 to: 0.0.0.0/0}
```

`authmethod`:
- `username`: Checks the `passwd/shadow` file
- `pam.*`: Passes the input to PAM which consults the modules in `/etc/pam.d` 
- `gssapi`: Kerberos

Minimal Configuration

`/etc/socksd.conf`
```
#### SERVER SETTINGS ####

debug: [0-2] # 0: no verbose, 1: some verbosity, 2: very
logoutput: stdout /var/log/sockd.log
logoutput: stderr /var/log/sockd_err.log

internal: [LISTEN INT] port = 1080
external: [OUTGOING IP]

### ----- AUTHENTICATION ---- ### 

user.privileged: root
user.unprivileged: nobody
method: username # Reads the passwd/shadow file

#### RULES #### 

client pass {
	from: 10.0.0.0/8 port 2000-65535 to: 0.0.0.0/0
}
socks pass { 
	from: 10.0.0.0/16 to: 0.0.0.0/0
	log: connect error
}

# Loggin

#### ROUTES

```



###### Securing Dante

Traffic going to Dante is not encrypted, therefore it has to be TLS wrapped. 

Connect to the proxy through an SSH tunnel

```bash
ssh -L 1080:localhost:1080 user@proxyhost
```

Now the SOCKS connection is inside SSH, encrypted.


##### Troubleshooting SOCKS

```

```

```bash
curl --socks5 "PROXY":1080 https://ipinfo.io
```

- `--noproxry '*'`: Force `curl` to use the proxy connection.

Make authenticated requests 

```bash
curl --socks5 "PROXY_SERV":1080 --proxy-user 'USER':'PASS' "TARGET_URL":8000
```

When using GSS-API (Kerberos)

```

```

```
Accepted on 10.0.2.15.1080, matched by client-rule #1, auth: username
host 192.168.99.101.8000, command connect
connect to host 192.168.99.101.8000 is now in progress
pass(1): tcp/connect [: username%vagrant@10.0.2.15.50072 10.0.2.15.1080 -> 192.168.99.103.50072 192.168.99.101.8000

sending response: VER: 5 REP: 0 FLAG: 0 ATYP: 1 address: 192.168.99.103.48486, authmethod 259

```