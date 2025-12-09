
`easy-rsa` is a wrapper around the `openssl` tool
#### Installing EasyRSA to Manage PKI

[[LINUX/PKI/OpenSSL]] For certificate generation using `openssl`

Download the easy-rsa package from here and unzip it
https://github.com/OpenVPN/easy-rsa/releases/

To add the easyrsa directory to the PATH variable, add the following to the .bashrc file

```
export PATH="/EASYRSA_DIR:$PATH"
```

Create an empty structure for the PKI

``` bash
easyrsa init-pki
```

Build the Certificate Authority. the CA creates and signs server and client certificates

``` bash
easyrsa build-ca
```

Pick strong passphrases to protect it. Choose a **Common Name**
This will create the **ca.crt** and **ca.key** in **EASYRSA_DIR/pki**

Generate a **keypair** and a **certificate signing request** for the OpenVPN server. Ignore protecting the keys with a passphrase by using the **nopass** option

``` bash
easyrsa gen-req 'VPNSERVER_CN' nopass
```

Generate a keypair and a certificate signing request for the client. Put a pass on it

``` bash
easyrsa gen-req 'CLIENT_CN' nopass
```

Sign the requests using their **Common Names**. Server 

``` bash
easyrsa sign-req server 'VPNSERVER_CN'
```

Sign the client requests

``` bash
easyrsa sign-req client 'CLIENT_CN'
```

Generate the Diffie-Hellman parameters for the server -> dh.pem

```
easyrsa gen-dh
```

Create a Hash Based Authentication Code (HBAC) key on the OpenVPN server. `tls-auth`

``` bash
openvpn --genkey --secret ta.key
```

Certificate Distribution
* **vpn server**: ca.crt, VPNSERVER_CN.crt, VPNSERVER_CN.key, dh.pem, ta.key
* **vpn client**: ca.crt, CLIENT_CN.crt, CLIENT_CN.key, ta.key

The location of the certificates on the machines is not strict and is defined in the .conf files
The .req files are no longer necessary

**HBAC**: verifies the integrity of messages

`server.conf`
```
# Connection

dev tun
port 1194
proto udp
topology subnet
server <TUNNEL_NETWORK> <SUBNET_MASK>
keepalive 10 120

# Logging

log-append /var/log/openvpn.log
status /var/log/openvpn-status.log
verb 4
mute 20

# Security

user nobody
group nobody
persist-key
persist-tun

# Certificates

ca /etc/pki/CA/certs/cacert.pem
dh /etc/openvpn/dh2048.pem
cert /etc/openvpn/gateway.ohio.cc.cert
key /etc/openvpn/gateway.ohio.cc.key

# TLS Auth
tls-auth /etc/openvpn/ta.key 0
cipher AES-256-GCM
```