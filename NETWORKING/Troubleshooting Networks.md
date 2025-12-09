
https://adafruit.com
#### Using `ss`

To search for specific connections (established ssh for example)

``` bash
ss -o state 'established' '( dport = :ssh or sport = :ssh )'
```

To display TCP established connections
``` bash
ss -t
```

To show all TCP connections ipv4, resolved hostnames

``` bash
ss -tar4
```

``` 
[kimchen@minimal ~]$ ss -tar4
State       Recv-Q Send-Q    Local Address:Port    Peer Address:Port
LISTEN      0      128        *:rpc.portmapper      *:*
LISTEN      0      5    minimal:domain              *:*
LISTEN      0      128        *:ssh                 *:*
LISTEN      0      1  localhost:ipp                 *:*
LISTEN      0      100localhost:smtp                *:*
ESTAB       0      64   minimal:ssh          10.0.1.2:54594
ESTAB       0      0    minimal:ssh          10.0.2.2:54568
```

#### Test Port Connectivity

Telnet (TCP only)

``` bash
telnet 'HOST' 'TCP PORT'
```

##### Netcat

NC is a one-shot listener. Does not re-bind for next incoming datagram packet. The listener must be restarted before the next test.

Test a TCP connection to port

`client`
```bash
nc -z -v 'REMOTE_HOST' 'TCP_PORT'
```

Set up a listener

`server`
```bash
nc -l "PORT"
```

Test UDP connections with `netcat`. 

`server`
``` bash
nc -u -l -p 'UDP_PORT'
```

`client`
``` bash
echo 'MESSAGE' | nc -u 'SERVER' 'UDP PORT'
```

Test local socket connectivity

``` bash
nc -U '/PATH_TO_SOCKET'
```

Send a datagram packet using Linux internals

```bash
echo "hello from bash" > /dev/udp/'HOST'/'UDP_PORT'
```

Set up a UDP port listener using Python

```bash
python3 -c 'import socket; s=socket.socket(socket.AF_INET, socket.SOCK_DGRAM); s.bind(("", 1194)); print("Listening..."); data, addr = s.recvfrom(2048); print(f"Received from {addr}: {data.decode()}")'
```

Send a datagram packet using Python

```bash
python3 -c 'import socket; sock=socket.socket(socket.AF_INET, socket.SOCK_DGRAM); sock.sendto(b"hello from Pitona", ("10.0.2.7", 1194))'
```

```
Listening...
Received from ('10.0.2.8', 33716): hello from bash
```

UDP Listening Server in Python3. The server is expected to send a reply back to client.

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind(('10.0.7.2', 1194))

print("UDP echo server listening on 1194")
while True:
    data, addr = sock.recvfrom(4096)
    print(f"Received from {addr}: {data}")
    sock.sendto(data, addr)  # Echo back
```

UDP Client in Python

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.settimeout(3)

server_address = ('10.0.2.7', 1194)
message = b"hello from client"

sock.sendto(message, server_address)
try:
    data, server = sock.recvfrom(4096)
    print(f"Received reply: {data}")
except socket.timeout:
    print("No reply received (timeout)")
```

`client`
```
Received reply: b'hello from client'
```

#### socat (SOcket CAT)

Bidirectional data relay between two data channels. Persistent UDP Listener

`server(listener)`
```bash
socat udp-recv:'UDP_PORT' stdout
```

Does the same 

```bash
socat -u UDP-RECV:'UDP_PORT' -
```

`-`: Print to `stdout`

`client`
```bash
echo "hello from socat" | socat - udp-sendto:'SERVER':'U_PORT'
```

`-`: Read from `stdin`. In this case `echo ...`. Could be `cat FILE.txt`

Set up a listener that sends a reply. The reply receiver is specified manually. Can be another host.

```bash
socat -v -d udp-recvfrom:'UDP_PORT',reuseaddr,fork udp-sendto:'HOST':'UDP_PORT'
```

```
< 2025/06/07 19:15:06.079654  length=16 from=0 to=15
hello from bash
```

Send a file with `socat`

`server`
```bash
socat -u TCP-LISTEN:'T_PORT',reuseaddr - > "RECEIVED_FILE.txt"
```

`client`
```bash
socat -u FILE:'FILE.txt' TCP:'SERVER':'T_PORT'
```
#### tcpdump

Listen for ICMPv4 packets on an interface

```bash
tcpdump -i 'INTERFACE' -n [-v] icmp
```

Listen for UDP packets on specific port

```bash
sudo tcpdump -n -i any udp port 1194
```
#### Using lsof

List Open Files

Show all open files and processes using port 1194, regardless of protocol

```bash
lsof -i :1194
```

To list UDP only processes

```bash
lsof -iUDP:1194
```
#### Ping

Start close and work systematically outwards. Ping localhost first to check if there is a problem with the network card

Ping message responses:
* Network is unreachable
* Name or service not known
* Destination Host Unreachable

#### Profiling Network with fping and nmap

`fping` pings an address range in sequence

**man fping**

Package
`fping`

To ping a subnet

``` bash
fping -c1 -gAds 192.168.137.0/24 2>1 | egrep -v "ICMP|xmt"
```

```
DESKTOP-S3S6ELI.mshome.net (192.168.137.1) : [0], 64 bytes, 0.520 ms (0.520 avg, 0% loss)
server15 (192.168.137.12)                  : [0], 64 bytes, 0.039 ms (0.039 avg, 0% loss)
logserver (192.168.137.23)                 : [0], 64 bytes, 1.14 ms (1.14 avg, 0% loss)
192.168.137.2 (192.168.137.2)              : [0], timed out (NaN avg, 100% loss)
192.168.137.3 (192.168.137.3)              : [0], timed out (NaN avg, 100% loss)
192.168.137.4 (192.168.137.4)              : [0], timed out (NaN avg, 100% loss)
192.168.137.5 (192.168.137.5)              : [0], timed out (NaN avg, 100% loss)
```

`-c`: packet count per host
`-g`: generate a list of addresses to ping, subnet 192.168.1.0/24 or IP range 192.168.33.20 192.168.33.30
`-s`: stats

#### Finding Duplicate IP Addresses with arping

**man arping**

Package:

`iptuils`

To scan a network duplicate IP addresses

``` bash
arping -I 'INTERFACE' -c 4 'IP_ADDRESS'
```

#### Testing HTTP Throughput and Latencly with httping

`httping` tells how long it takes for a server to respond to a HEAD request

Package: 
`httping`

man httping

``` bash
httping -c4 -l -rGg 'WEBSITE'
```

`-r`: resolves hostname only once. Minimizes DNS latency

If minimizing DNS latency makes a huge difference, check nameservers

Test an alternate port

```bash
httping -c4 -l -rsGg WEBSITE:PORT
```

`-s`: displays return code, such as 200 OK.

```
root@server15:~# httping -c4 -l -srGg www.oreilly.com
PING www.oreilly.com:443 (/):
connected to 23.209.22.105:443 (1709 bytes), seq=0 time=106.71 ms 200 OK
connected to 23.209.22.105:443 (1710 bytes), seq=1 time= 84.62 ms 200 OK
connected to 23.209.22.105:443 (1709 bytes), seq=2 time= 82.28 ms 200 OK
connected to 23.209.22.105:443 (1709 bytes), seq=3 time= 81.99 ms 200 OK
--- https://www.oreilly.com/ ping statistics ---
4 connects, 4 ok, 0.00% failed, time 4497ms
round-trip min/avg/max = 82.0/88.9/106.7 ms
```

#### Finding Troublesome Routers with `mtr`

mtr: My Traceroute

Package: 
`mtr`

man mtr

``` bash
mtr -wo LSRABW 'TARGET'
```

```
root@server15:~# mtr -wo LSRABW 9circles.cc
Start: 2024-03-26T09:57:30+0200
HOST: server15                             Loss%   Snt   Rcv   Avg  Best  Wrst
  1.|-- DESKTOP-S.mshome.net            0.0%    10    10   0.5   0.4   0.7
  2.|-- ???                                  100.0    10     0   0.0   0.0 
  3.|-- dsldevice.lan                         0.0%    10    10   1.2   1.1 
  4.|-- 10.107.64.2                           0.0%    10    10   2.8   2.2 
  5.|-- ???                                  100.0    10     0   0.0   0.0 
  6.|-- 212-39-69-100.ip.btc-net.bg           0.0%    10    10   3.1   2.7 
  7.|-- -66-10.ip.btc-net.bg            0.0%    10    10   4.8   3.1  12.3
  8.|-- ???                                  100.0    10     0   0.0   0.0   0.0
  9.|-- ???                                  100.0    10     0   0.0   0.0   0.0
 10.|-- cpr-shost-3dc.superhosting.bg  0.0%    10    10   3.3   2.7   3.8
 11.|-- host-18-171.superhosting.bg    0.0%    10    10   2.8   2.3   3.4
```

`-o`: output. LSRABW, fields to display in order

#### Bandwidth Troubleshooting with iftop

**man iftop**

Packages: 
`iftop`

``` bash
iftop -i 'INTERFACE'
```

#### Setting Bandwidth Limit on a Process with tc

https://netbeez.net/blog/how-to-use-the-linux-traffic-control/

Packages: 
`iproute`

**man tc**

To list current rules on an interface

``` bash
tc qdisc show dev 'DEVICE'
```

```
[root@dbserver19 ~]# tc qdisc show dev enp0s3
qdisc pfifo_fast 0: root refcnt 2 bands 3 priomap  1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
```

To set a 100ms delay on an interface

``` bash
tc qdisc add dev 'DEVICE' root netem delay 'DELAY_MS'
```

```
[root@dbserver19 ~]# tc qdisc show dev enp0s3
qdisc netem 8001: root refcnt 2 limit 1000 delay 100.0ms
```

To clear the rules

```bash
tc qdisc del dev 'DEVICE' root
```

In order to limit the egress bandwidth we can use the following command:

```
tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms`
```

* tbf: use the token buffer filter to manipulate traffic rates
* **rate**: sustained maximum rate
* **burst**: maximum allowed burst
* **latency**: packets with higher latency get dropped

Watch every 2s (default) the output of a command. In this case connectivity to a web server is checked in almost real-time. Titles and other stats are suppressed, only output of `curl` is shown

```bash
watch -t -c "curl -4sS http://HOST:PORT"
```

