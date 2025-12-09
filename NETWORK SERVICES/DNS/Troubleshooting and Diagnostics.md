---
tags:
  - dns
---

The `named` user must own and have read and write permissions over the named-related files and directories. 

```
/etc/named.conf
Access: (0640/-rw-r-----)  Uid: (   25/   named)   Gid: (   25/   named)
/var/named
Access: (1770/drwxrwx--T)  Uid: (    25/    named)   Gid: (   25/   named)
```

#### rndc 

`rndc` - Name Server control utility. Front end to named

Files required for the `rndc` command: 
- `/etc/rndc.conf`
- `/etc/rndc.key`

Port 953

To generate a sample `rndc.conf` and a key for `rndc.key` conf

``` bash
rndc-confgen
```

Create the `/etc/rndc.conf` file with the output from the rndc-confgen

```
key "rndc-key" {
        algorithm hmac-md5;
        secret "0TcJn3W+jtAIy7A5vFEEgg==";
};

options {
        default-key "rndc-key";
        default-server 127.0.0.1;
        default-port 953;
};
```

To notify slaves about zone changes

``` bash
rndc notify "ZONE"
```

To retransfer a zone from a master

``` bash
rndc retransfer "ZONE"
```

#### host - DNS lookup utility

The host utility is a replacement for the old `nslookup` utility

Packages: 
`bind-utils`

To do a nameserver lookup for a domain

``` bash
host "FQDN"
```

```
[root@prometheus ~]# host smtp.google.com
smtp.google.com has address 142.250.153.26
...
smtp.google.com has address 74.125.128.26
smtp.google.com has IPv6 address 2a00:1450:4013:c16::1a
```

To do a reverse lookup. Works if there is a PTR record in the reverse lookup zone on the NS

``` bash
host "IP_ADDRESS"
```

```
36.141.251.142.in-addr.arpa domain name pointer sof04s07-in-f4.1e100.net
```

dnssec : influences forwarding. If forwarding does not work, temporarily set 

```
dnssec-validation no;
```

#### Logging

To filter out named-related messages only

``` bash
journalctl _COMM=named
```

#### dig and delv for queries

`dig` - Domain Information Groper

dig searches `/etc/resolv.conf` by default for nameservers

```
[root@prometheus ~]# dig -t MX google.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> -t MX google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63846
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 10

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.                    IN      MX

;; ANSWER SECTION:
google.com.             300     IN      MX      10 smtp.google.com.

;; ADDITIONAL SECTION:
smtp.google.com.        300     IN      A       108.177.96.27
smtp.google.com.        300     IN      A       108.177.96.26
smtp.google.com.        300     IN      A       108.177.119.27
smtp.google.com.        300     IN      A       108.177.119.26
smtp.google.com.        300     IN      A       108.177.127.26
smtp.google.com.        300     IN      AAAA    2a00:1450:4013:c06::1a
smtp.google.com.        300     IN      AAAA    2a00:1450:4013:c06::1b
smtp.google.com.        300     IN      AAAA    2a00:1450:4013:c00::1b
smtp.google.com.        300     IN      AAAA    2a00:1450:4013:c00::1a

;; Query time: 48 msec
;; SERVER: 192.168.137.1#53(192.168.137.1)
;; WHEN: Sat May 04 14:14:18 EEST 2024
;; MSG SIZE  rcvd: 252
```

Sections:
AUTHORITY : The nameserver(s) responsible for the searched domains.
ADDITIONAL:  IP address the server(s) of the authority section resolve to
ANSWER: 
SERVER: The DNS server providing the response

To find the delegation path to nameservers

``` bash
dig +trace "DOMAIN"
```

