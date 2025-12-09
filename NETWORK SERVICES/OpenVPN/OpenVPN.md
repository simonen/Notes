---
tags:
  - openvpn
  - certificates
  - openssl
  - centos
---

Open Virtual Private Networks

Opens an SSL/TLS encrypted tunnel between two networks at separate physical locations
Dependent on OpenSSL

**TUN / TAP**

Virtual network interfaces built into the Linux Kernel. Specified in the client and server configuration files

- `TUN`: for routed networks
- `TAP`: for bridged networks

#### Installing Server and Client OpenVPN

Packages:
- `epel-release` (repo)
- `openvpn`: provides client and server functionality
- `NetworkManager-openvpn-gnome`: for GNOME systems

Sample configurations:
* `/usr/share/doc/openvpn/sample`

#### Setting up a Connection Test

OpenVPN works with **UDP** on port **1194** by default

Add the `openvpn` service to the firewall on the **openvpn** server

```bash
firewall-cmd --add-service=openvpn --permanent ; firewall-cmd --reload
```

##### SELinux

List OpenVPN related SELinux context labels

```bash
semanage fcontext -l | grep openvpn
```

The `/var/log/openvpn.log` and `/var/log/openvpn-status.log` must have the `openvpn_var_log_t` SELinux context

`/var/log/audit/audit.log`
```
type=AVC msg=audit(...): avc:  denied  { write } for  pid=11813 comm="openvpn" name="openvpn-status.log" ... scontext=system_u:system_r:openvpn_t:s0 tcontext=unconfined_u:object_r:var_log_t:s0 tclass=file permissive=0
```

Set the appropriate SELinux context

```bash
semanage fcontext -a -t openvpn_var_log_t /var/log/openvpn-status.log
semanage fcontext -a -t openvpn_var_log_t /var/log/openvpn.log
restorecon -v /var/log/openvpn-status.log
restorecon -v /var/log/openvpn.log
```

```
ls -Zla /var/log/openvpn*
1 root root unconfined_u:object_r:openvpn_var_log_t:s0 /var/log/openvpn.log
1 root root unconfined_u:object_r:openvpn_var_log_t:s0 /var/log/openvpn-status.log
```

#### The Simplest (Point-to-Point) Connection

Create a basic VPN tunnel from Host1 to Host2. Use for testing purposes only!

`Host 1`
``` bash
openvpn --dev tun0 --ifconfig 10.0.0.1 10.0.0.2 [--daemon] [--verb 1-10]
```

Create a VPN tunnel from Host2 to Host1

`Host 2`
``` bash
openvpn --remote 'Host1' --dev tun0 --ifconfig 10.0.0.2 10.0.0.1 [--daemon]
```

- `--dev tun`: creates a layer 3 VPN tunnel for IP networks
- `--ifconfig <local VPN endpoint> <expected remote VPN endpoint>`
- `--daemon`: sends the job to the background
- `--verb 1-10`: Verbosity level
- `--cipher CIPHER`: Specify cipher. Use `AES-256-CBC` for newer versions.

This will set up a tunnel between the hosts. Their current sessions will be taken by the tunneling, so open new sessions to use the hosts.

`Host2`
```
net_addr_ptp_v4_add: 10.8.0.2 peer 10.8.0.1 dev tun0
TCP/UDP: Preserving recently used remote address: [AF_INET]10.0.2.155:1194
UDP link local (bound): [AF_INET][undef]:1194
UDP link remote: [AF_INET]10.0.2.155:1194
Peer Connection Initiated with [AF_INET]10.0.2.155:1194
Initialization Sequence Completed
```

`Host1`
```
TUN/TAP device tun0 opened
/sbin/ip link set dev tun0 up mtu 1500
/sbin/ip addr add dev tun0 local 10.8.0.1 peer 10.8.0.2
UDPv4 link local (bound): [AF_INET][undef]:1194
UDPv4 link remote: [AF_UNSPEC]
Peer Connection Initiated with [AF_INET]10.0.2.156:1194
Initialization Sequence Completed
```

##### Plaintext Tunnel

> [!NOTE]
> For debugging purposes only!

Disable the HMAC authentication

`server`
```bash
openvpn --ifconfig 10.200.0.1 10.200.0.2 --dev tun0 --auth none
```

```
 ******* WARNING *******: All encryption and authentication features disabled -- All data will be tunnelled as clear text and will not be protected against man-in-the-middle changes. PLEASE DO RECONSIDER THIS CONFIGURATION!
```

#### Using Configuration Files

OpenVPN treats configuration file entries and command-line parameters identically. Command line parameters just start with `--` and they can be used to override config file entries.

Basic OpenVPN config file

Start the server with a non-default port

`server`
```bash
openvpn --ifconfig 10.200.0.1 10.200.0.2 --dev tun --port 11000
```

`client.conf`
```
dev tun
port 1194
ifconfig 10.200.0.2 10.200.0.1
remote 192.168.137.10
verb 3
```

Start the client connection using the config file and override the default 1194 port

```bash
openvpn --config 'client.conf' --port 11000
```

This is considered a "connection block" and cannot be overridden with `--port N`

```
remote 'openvpnserver' 1194
```

#### Setting Up Encryption with Pre-shared Static Keys

> [!NOTE]
> Using Pre-shared static keys is an outdated method which does not provide forward secrecy. Use TLS mode with certificates instead. 

Create the secret key in `/etc/openvpn/keys` and share it with the other host

```bash
openvpn --genkey secret 'SECRET.key'
```

`/etc/openvpn/server.conf`
```
dev tun
ifconfig 10.0.0.1 10.0.0.2
secret /etc/openvpn/keys/SECRET.key
local LOCAL_IP_ADDR
cipher AES-256-CBC # for stronger encryption
```

`/etc/openvpn/client.conf`
```
dev tun
ifconfig 10.0.0.2 10.0.0.1
secret /etc/openvpn/keys/SECRET.key
remote SERVER_IP_ADDR
cipher AES-256-CBC
```

Start the connection on both, server and client

- `openvpn /etc/openvpn/server.conf`
- `openvpn /etc/openvpn/client.conf`

This type of connection is limited to two hosts only

Or do it using the command line

```bash
openvpn --ifconifg 'HOST1_VPN_ENDPOINT' 'HOST2_VPN-ENDPOINT' --dev tun \
--secret 'SECRET.KEY'
```
#### Controlling OpenVPN with `systemctl`

To run OpenVPN as a daemon in the background, managed by `systemd`, the service unit file must be created first. On both, server and clients

OpenVPN-related unit files in `/usr/lib/systemd/system`. 

``` bash
systemctl enable --now openvpn-server@'SERVER_CONFIG_NAME'
systemctl enable --now openvpn-client@'CLIENT_CONFIG_NAME'
```

The @ indicates a parameterized unit file. Multiple unit files can be created for the same service, calling different configuration files. This allows for multiple configurations to be created without writing multiple unit files.

How a parametrized unit file looks

```bash
systemctl cat openvpn-server@.service
```

```
[Unit]
Description=OpenVPN service for %I
...
Documentation=https://community.openvpn.net/openvpn/wiki/HOWTO

[Service]
Type=notify
PrivateTmp=true
WorkingDirectory=/etc/openvpn/server
ExecStart=/usr/sbin/openvpn --status %t/openvpn-server/status-%i.log --status-version 2 --suppress-timestamps --cipher AES-256-GCM --data-ciphers AES-256-GCM:AES-128-GCM:AES-256-CBC:AES-128-CBC >
...
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

The `.service` file shows what ciphers are passed as daemon options. 

Configuration files are read from the `WorkingDirectory` directive. 

#### Server-Client Connections

This setup allows multiple client connections to a single server.

> [!NOTE]
> No openvpn daemons should be running

The most Basic Server Config

`server.conf`
```
dev tun
proto udp
port 1194

local LISTEN_ADDRESS
server VPN_NETWORK MASK
topology subnet

push "route LOCAL_NETWORK MASK"

keepalive 10 120

ca /etc/openvpn/server/FULL-CHAIN.CRT
cert /etc/openvpn/server/SERVER.CRT
key /etc/openvpn/server/PRIVATE.KEY

# TLS Auth
tls-groups secp384r1:prime256v1:brainpoolP256r1
dh none
cipher AES-256-GCM
```

`topology subnet`: Use this as the default `topology net30` is obsolete

`client.conf`
```
dev tun
proto udp
nobind

client
remote VPN_SERVER_IP 1194

ca /etc/openvpn/client/certs/full-chain.crt
cert /etc/openvpn/client/certs/branch1.crt
key /etc/openvpn/client/certs/branch1.key

# TLS Auth
cipher AES-256-GCM
```


- `server NETWORK MASK`: Server will take the first address of the network pool and assign the rest to incoming VPN connections
- `tls-auth <ta.key>`: 0 for the server, 1 for the client. Use `ec` instead.
- `keepalive`: (ping every N seconds) (wait for response seconds)
- `verb [0-11]` : verbosity level, 0 = no logging
- `user/group [USER]`: user/group to run OpenVPN as. 
- `persist [tun/key]`: Allow OpenVPN to retain sufficient privileges to work with the network and SSL after the privilege drop.
- `explicit-exit-notify 1`
- `log-append /var/log/openvpn.log`
- `status /var/log/openvpn-status.log`
- `verb 4`
- `mute 20`
- `tls-groups`
- `dh none`: Disable DH key exchange. Not needed with ECDHE and TLS1.3
#### Routing

Add route to a network behind the VPN server on the clients

`server`
```
push "route INTERNAL_NETWORK SUBNET_MASK"
```

- `push "route network subnet"`: Send route to connecting clients. The quotes are mandatory
- `route NETWORK SUB"`: Add route to the server's routing table.

Restart the server and the clients. The clients will now have a new route in the routing table

`ip route`
```
default via 192.168.137.1 dev enp0s3 proto static metric 100
10.0.6.0/24 via 192.168.0.1 dev tun0 <-- internal network routed through the VPN tun
192.168.0.0/24 dev tun0 proto kernel scope link src 192.168.0.2
192.168.137.0/24 dev enp0s3 proto kernel scope link src 192.168.137.16 metric 100
```

VPN traffic needs to be forwarded from the gateway to the internal network and back. 

Add the VPN tunnel interface to the firewall public zone

``` bash
firewall-cmd --zone=public --add-interface=tun0 --permanent
```

Forward incoming connections on 22 from the tunnel to the internal network

``` bash
iptables -A FORWARD -i tun0 -s 10.0.2.0/24 -d 192.168.0.0/24 -p tcp --dport 22 -j ACCEPT
```

##### Masquerading

To rewrite the source address of packets coming from the VPN client with that of the VPN server

`server`
```bash
iptables -t nat -A POSTROUTING -s VPN_NET/24 -d DEST_NET/24 -o DEV -j MASQUERADE
```

- `-t nat`: Specifies the `nat` table, used for changing source/destination IP addresses.
- `-A POSTROUTING`: Appends a rule to the POSTROUTING chain, part of `nat` table , which is used after the routing decision - just before the packet leaves the system.
- `-s NETWORK`: Matches packets **originating from the VPN subnet**
- `-d NETWORK`: Matches packets **going to** the destination network
- `-o DEV`: Applies the rule if only the packets is going out via the DEV interface
- `-j MASQUERADE`: Masks (NATs) the source IP address of the packet with the IP of the outgoing interface, so that return traffic can find its way back
#### Distributing Client Configurations with .ovpn Files

Linux clients must have the **NetworkManager-openvpn-gnome** package installed to be able to import the .**ovpn** (pro)files in the GUI

The .**ovpn** file packs the information from the client conf file and certificates. Connection profile. Requires sudo privileges when executed in shell

`.ovpn structure`
```
OPENVPN_SETTINGS

<ca>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</ca>

# client.crt
<cert>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</cert>

# client.key
<key>
-----BEGIN ENCRYPTED PRIVATE KEY-----
...
-----END ENCRYPTED PRIVATE KEY-----
</key>
```

#### Hardening OpenVPN Servers

All SSL and TLS protocols under TLS 1.2 are deprecated and should not be used. Set min TLS version to 1.2

`server and client`
```
tls-version-min 1.2
```

Use stronger data channel cipher and enforce it by disabling cipher negotiation

`server and client`
```
cipher AES-256-GCM
ncp-disable
```

```
Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
```

TLS 1.2. Set preferred cipher suites. 

`server and client`
```
# TLS 1.2
tls-cipher TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-RSA-
WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256:
TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256
```

Elliptic Curve Diffie-Hellman Ephemeral (ECDHE) can be used instead of the Diffie-Hellman static keys (ta.key). Simple, faster, better security. Use `tls-groups <CURVES>` for better flexibility

`server`
```
dh none
# Explicitly set a single elliptic curve
ecdh-curve secp384r1 
# Provide a list of preferred curves for TLS exchange. This method is better
tls-group CURVE1:CURVE2:...
```

List available elliptic curves

```bash
openvpn --show-curves
openssl ecparam -list_curves
```

- `opt-verify`: ( server ) checks for compatibility between server and client settings, and disconnects clients that do not match. Checks:
	`dev-type, link-mtu, tun-mtu, proto, ifconfig, comp-lzo, fragment, keydir, cipher, auth, keysize, secret, no-replay, no-iv, tls-auth, key-method, tls-server, tls-client`
- `tls-server`: on the server, works with `tls-client` on the client
- `remote-cert-tls server` : ( client ) Ensures the server's certificate is properly validated. Adds an extra layer of security by ensuring the OpenVPN client connects only to legitimate server. During handshake. Requires critical key usage extensions in the certificate. NOT TESTED!
- `remote-cert-tls client`: used in the OpenVPN server to ensure that the client certificates have the correct key usage during the handshake.  Requires critical key usage extensions in the certificate.
- `ecdh-curve secp384r1`: Explicitly set an elliptic curve. Use `tls-group` instead.
- `dh none`: Explicitly disable DH for handshake negotiation. Use ECDHE instead.
- `verify-client-cert require`
- `auth sha[128|256|512]`:   Bit message hash 'SHA256' for HMAC authentication


[Man-in-the-middle attacks and client-server verifications](https://openvpn.net/community-resources/how-to/#mitm)

Check if the certificate has the necessary extensions

```
X509v3 Extended Key Usage:
                TLS Web Client/Server Authentication
            X509v3 Key Usage: critical
                Digital Signature
```

`remote-cert-tls server`: The client successfully verifies the server certificate
```
Validating certificate extended key usage
++ Certificate has EKU (str) TLS Web Client Authentication, expects TLS Web Server Authentication
++ Certificate has EKU (oid) 1.3.6.1.5.5.7.3.2, expects TLS Web Server Authentication
++ Certificate has EKU (str) TLS Web Server Authentication, expects TLS Web Server Authentication
us=681809 VERIFY EKU OK
us=682124 VERIFY OK: depth=0, C=BG, ST=Sofia, L=Sofia, O=Ohio, OU=IT, CN=headoffice.ohio.cc
```

The server successfully verifies the client certificate

```
Validating certificate extended key usage
192.168.137.195:1194 ++ Certificate has EKU (str) TLS Web Client Authentication, expects TLS Web Client Authentication
192.168.137.195:1194 VERIFY EKU OK
192.168.137.195:1194 VERIFY OK: depth=0, C=BG, ST=Sofia, L=Sofia, O=Ohio, OU=IT, CN=kavarna.ohio.cc
```

Further:
Users can be disallowed from saving passwords, SELinux, chroot

#### Other Configuration Directives

- `client-to-client`: allows VPN endpoints to "see" each other.
- `resolv-retry infinite`
- `ifconfig-pool-persist ipp.txt [refresh interval]` : used to persistently assign IP addresses to VPN clients.
	When a client connects, the VPN server assigns it an IP address from the predefined pool in the `server` directive. The `ipp.txt` OpenVPN manages IP-to-client mapping automatically in this file. NOT WORKING!
- `float`:  ( in server conf only ) allows clients to roam on different networks without losing connection, as long as they pass authentication tests
- `mute 20`: limits the number of repetitive log messages. Suppress the first N occurrences of the same msg
- `explicit-exit-notify 1`: Ensures proper connection cleanup, helping the server recognize the client has intentionally disconnected, allowing it to manage resources more efficiently
- `nobind`: Prevents the client from binding to a specific local port, allowing the system to assign a random, available port for outgoing traffic. Good for avoiding port conflicts, when running multiple VPN connections or when network conf is dynamic
- `comp-lzo`: deprecated. Compression is discouraged

#### Authentication

Users will authenticate with their Linux credentials to the gateway using the pam authentication plugin

`/etc/openvpn/server/.conf`
```
plugin /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so login
```

`/etc/openvpn/client/.conf`
```
auth-user-pass
```

The client

```
[root@localhost client]# systemctl restart openvpn-client@mobile-client
🔐 Enter Auth Username: kimchen
🔐 Enter Auth Password: ******
```

`server log`
```
AUTH-PAM: BACKGROUND: received command code: 0
AUTH-PAM: BACKGROUND: USER: kimchen
AUTH-PAM: BACKGROUND: my_conv[0] query='Password: ' style=1
10.0.2.6:59463 PLUGIN_CALL: POST /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so/PLUGIN_AUTH_USER_PASS_VERIFY status=0
10.0.2.6:59463 TLS: Username/Password authentication succeeded for username 'kimchen'
10.0.2.6:59463 [branch1.ohio.cc] Peer Connection Initiated with [AF_INET]10.0.2.6:59463
```

#### Client-Specific Options using client-config-dir Files

```
/etc/openvpn/server/
├── ccd
│   ├── branch1.ohio.cc <- matching user
│   └── DEFAULT <- everyone else
```

Using client-specific options provides fine-grained access.

`server.conf`
```
server 10.200.0.0 255.255.255.0
topology subnet
client-config-dir /etc/openvpn/server/ccd
```

Extract the CN of the client certificate

```bash
openssl x509 -in CLIENT.CRT -noout -subject
```

```
subject=C=BG, ST=Sofia, O=Ohio, OU=IT, CN=branch1.ohio.cc
```

In the `clients` directory, create a file named after the CN of the client certificate

`/etc/openvpn/clients/branch1.ohio.cc`
```
ifconfig-push 10.200.0.7 255.255.255.0 (for topology subnet)
ifconfig-push 10.200.0.7 10.200.0.7 (for legacy topology net 30)
```

`client`
```
TUN/TAP device tun0 opened
net_iface_mtu_set: mtu 1500 for tun0
net_iface_up: set tun0 up
net_addr_v4_add: 10.200.0.7/24 dev tun0
```

If there is no matching client file in the directory, default configurations can be placed in a file called `DEFAULT` 

- Specify full path to the ccd file in the server configuration
- The directory must be accessible to `nobody` or  whatever user is running `openvpn`
- Filenames are strict and without extensions

Some other options

- `push OPTION`: Used for pushing DNS, routes...
	- `push "dhcp-option DNS 10.0.8.1"`
	- `push "route NETWORK SUBNET`
- `push-reset`: Overrides global `push` option
- `iroute CLIENT_NET SUB`: Routes client subnets to the server through the correct tunnel. Uses OpenVPN's internal routing table, not the OS'. Done on the server.
- `disable`: Disable a user. Just the word `disable` in the file.
- `config`: Include another configuration file

#### OpenVPN Management Console

The OpenVPN management console provides useful info on connections and other functionality.

Start OpenVPN with 

`server`
```bash
openvpn --config SERVER.CONF --management 'OPENVPN_SERVER' 'PORT'
```

Connect to the console

`server`
```
nc or telnet 'OPENVPN_SERVER' 'PORT'
```

```
Management Interface for OpenVPN 2.... 
Commands:
auth-retry t           : Auth failure retry mode (none,interact,nointeract).
<...>
status
OpenVPN CLIENT LIST
Updated,2025-06-03 17:09:01
Common Name,Real Address,Bytes Received,Bytes Sent,Connected Since
dhcp.ohio.cc,10.0.8.2:52754,6363,6387,2025-06-03 17:05:24
ROUTING TABLE
Virtual Address,Common Name,Real Address,Last Ref
10.200.0.2,dhcp.ohio.cc,10.0.8.2:52754,2025-06-03 17:05:24
10.0.6.0/24,dhcp.ohio.cc,10.0.8.2:52754,2025-06-03 17:05:24
```

#### Pool Ranges

Using pool ranges allows to separate VPN administrative users from regular VPN users by maintaining a persistent record of client-to-IP mappings.

This is achieved by using `client-config-dir` with matching clients who are given specific IP address, while everyone else is assigned IPs dynamically from the pool. For more control of the pool, if using the `server` directive, a `DEFAULT` file can be included with the specified pool.


`/etc/openvpn/server/ccd/branch1.ohio.cc`
```
ifconfig-push 10.200.0.6 255.255.255.0
```

`/etc/openvpn/server/ccd/DEFAULT`
```
ifconfig-push 10.200.0.100 255.255.255.0
```

The `server 192.168.200.0 255.255.255.0` directive does in effect:

`server.conf`
```
mode server 
tls-server 
ifconfig 192.168.200.1 255.255.255.0  
ifconfig-pool 192.168.200.10 192.168.200.20 
push "route 192.168.200.0 255.255.255.0"
push "route-gateway 192.168.200.1"
push "topology subnet"
```

By specifying `ifconfig-pool`, instead of `server ...`, this behavior can be overridden. `server` directive uses `ifconfig-pool` implicitly and cannot be used both at the same time.

`server`
```
dev tun
proto udp
port 1194

mode server
tls-server
topology subnet
ifconfig 10.200.0.1 255.255.255.0
ifconfig-pool 10.200.0.100 10.200.0.120

push "topology subnet"
push "route 10.200.0.0 255.255.255.0"
push "route-gateway 10.200.0.1"

<...certs...and...tls...>

client-config-dir /etc/openvpn/server/ccd
```

