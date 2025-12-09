---
tags:
  - dns
  - centos
---
[[gitea/LINUX 1/NETWORK SERVICES/DNS/DNS]]
`Apress – Learn Centos Linux Network Services - Antonio Vazquez`

Berkley Internet Name Domain

Package
- `bind` (CentOS)
- `bind9` (Ubuntu)

Service
- `named` (CentOS)
- `bind` (Ubuntu)

man **named**
#### Files and Directories

- `/etc/named.conf` : main conf file
- `/var/named/` : zone files (CentOS)
- `/var/cache/bind`: zone files (Ubuntu)
- `/usr/share/doc/bind-9.11.4/sample/` : sample config files

##### Related System Files

- `/etc/hosts`
- `/etc/nsswitch.conf`
- `/etc/resolv.conf`

The `named.service` runs as the `named` user. Should NOT run as root

``` 
[root@minimal init.d]# ps aux | grep named
named    19575    Ssl   /usr/sbin/named -u named -c /etc/named.conf
```

#### Configuring BIND

`/etc/named.conf`
```
options {
         listen-on port 53 { 127.0.0.1; };
         listen-on-v6 port 53 { ::1; };
         directory       "/var/named";
         dump-file       "/var/named/data/cache_dump.db";
         recursing-file  "/var/named/data/named.recursing";
         secroots-file   "/var/named/data/named.secroots";
         allow-query     { localhost; };
     }
```

1. `listen-on port` 53: { 127.0.0.1; }** : Assign an IP of a network interface, otherwise the service will be unreachable by network devices. Or 'any'.
2. `directory`: default directory for zone files
3. `allow-query`: Remote networks or computers allowed to query the server.
	**any**; *NETWORK/MASK*; *IP_ADDRESS*; Avoid **any** if possible.

`recursive`
* `yes`: the DNS server will perform recursive queries for clients to other DNS servers
* `no`: the DNS server will not perform recursive queries and will only answer queries for which it has authoritative information 

#### Lookup Zones

The default zone "." is configured in `/var/named/named.ca`. Contains IP addresses of internet root servers. 

`/etc/named.conf`
```
zone "." IN {
        type hint;
        file "named.ca";
	};
```

##### Forward Lookup Zones

To define a forward lookup zone, i.e translating hostnames to ip addresses Location relative to the default directory specified in the options.

``/etc/named.conf``
```
zone "olympus.local" IN {
        type master;
        file "olympus.local.zone";
        };
```

To check configuration syntax

``` bash
named-checkconf
```

##### The Zone file

[[gitea/LINUX 1/NETWORK SERVICES/DNS/DNS#Zones]]

Minimal zone file configuration, containing the zone file header, the SOA record and the nameserver record.

```
$ORIGIN olympus.local.
$TTL 10800      ; 3 hours
@                    IN SOA     ns1.olympus.local. dns-admin.olympus.local. (
                                30         ; serial
                                86400      ; refresh (1 day)
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                10800      ; minimum(NegativeTTL)(3 hours)
                                )
                    IN  NS      ns1.olympus.local.
```

When the "@" symbol appears at the beginning of a resource record, it's implicitly referring to the current origin or base domain name. This usage is commonly seen in zone file metadata, such as SOA (Start of Authority) records, NS (Name Server) records, and TXT (Text) records that provide domain-related information. The DNS server will append the domain to hostnames in the same column
The trailing dot at the end of the FQDN tells named to treat the entry as FQDN. Leaving off the dot will treat it as a hostname, appending the $ORIGIN again

admin.olympus.local. : the contact field. @ is .

```
                    IN  MX      prometheus.olympus.local.
delphos             IN  A       10.0.2.67
prometheus          IN  A       10.0.3.90
$TTL 300        ; 5 minutes
mail                IN  CNAME   prometheus.pl
ns1                 IN  CNAME   delphos
                        TXT     "313ca80f6a3edb16931ded49de08ba24be"
```
; comments
- `serial`: every zone must have an associated serial number. Used when replicating information between DNS servers to determine if there is a newer version of a zone file
olympus.local. 

Check the syntax of a zone file

``` bash
named-checkzone "ZONE" "ZONE_FILE"
zone olympus.local/IN: loaded serial 0
OK
```

```
[root@prometheus ~]# host ns1
ns1.olympus.local is an alias for delphos.olympus.local.
delphos.olympus.local has address 10.0.2.67
```
##### Reverse DNS Lookup Zones. .ARPA Zone Management

Resolves IP addresses to hostnames

Reverse lookup zones do not have domains, but an unique address range. The special 
`in-addr.arpa` domain is used - the root zone for reverse mappings. 

The **.arpa** domain is the “Address and Routing Parameter Area” domain and is designated to be used exclusively for Internet-infrastructure purposes. A list of second level .arpa domains can be found here https://www.iana.org/domains/arpa.

Define the zone

`/etc/named.conf`
```
zone "2.0.10.in-addr.arpa" IN {
	type master;
    file "10.0.2.zone";
 };
```

2.0.10: the network portion in reverse.
`in-addr`: Second level `.arpa` domain used for mapping IPv4 addresses to Internet domain names

Create the reverse zone file `/var/named/NETWORK.zone`

Minimal reverse look configuration

`/var/named/10.0.2.zone header`
```
$ORIGIN 2.0.10.in-addr.arpa.
$TTL 2D
@        IN SOA delphos.olympus.local. admin.olympus.local. (
               0               ; serial
               259200          ; refresh (3 days)
               14400           ; retry (4 hours)
               18140           ; expire (3 weeks)
               604800)         ; minimum (1 week)

         NS delphos.olympus.local.
67       PTR delphos.olympus.local.
```


```
23       PTR prometheus.olympus.local
```

23, 67 is the host part of the IP addresses of the related hosts.

To run a reverse lookup for an address

```
[root@prometheus ~]# dig -x 10.0.2.67 @10.0.2.67

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> -x 10.0.2.67 @10.0.2.67
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55365
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;67.2.0.10.in-addr.arpa.                IN      PTR

;; ANSWER SECTION:
67.2.0.10.in-addr.arpa. 172800  IN      PTR     ns1.olympus.local.
67.2.0.10.in-addr.arpa. 172800  IN      PTR     delphos.olympus.local.

;; AUTHORITY SECTION:
2.0.10.in-addr.arpa.    172800  IN      NS      delphos.olympus.local.

;; ADDITIONAL SECTION:
delphos.olympus.local.  10800   IN      A       10.0.2.67

;; Query time: 1 msec
;; SERVER: 10.0.2.67#53(10.0.2.67)
```

To output the ANSWER SECTION only

```
[root@prometheus ~] dig -x 10.0.2.67 +short
ns1.olympus.local.
delphos.olympus.local.
```
#### Forwarders 

If a DNS server does not have info about a record, it can forward the query to an upstream DNS server. Insert the code block in the general options block. 

`/etc/named.conf`
```
options {
	recursion yes;
	forward only;
	forwarders { FORWARDER_IP; FORWARD_IP; }
}
```
#### Slave Servers and Zone Transfers

Slave servers hold a read-only copy of the DNS records of the master server. It serves to share the workload and act as a fail-over in case the main service becomes unavailable.

As the slaves do not change data in the zone files, they only need to define slave status and masters within the zone in the `/etc/named.conf`

Define the zone and the master(s) it should be transferred from.

`/etc/named.conf` -> slave
```
zone "olympus.local" IN {
	type slave;
	file "/slaves/olympus.local.zone";
	masters { 10.0.2.7; };
}
```

Same applies to the reverse zones. 

Add a NS record of the slave server to the zone file on the master

`/var/named/olympus.local and also in /10.0.2.zone on the master`
```
IN      NS      slave-server.olympus.local.
```

The slaves should be notified whenever a zone has been changed. 

`/etc/named.conf` -> master
```
zone "olympus.local" IN {
        type master;
        file "olympus.local.zone";
        notify yes;
};
```

The `/var/named/slaves` directory on the slaves must be writable by the named user

```
drwxrwx---. 2 named named 4096 Apr  7 21:30 slaves/
```

Restart the named services on the servers
The master should now transfer the shared zone file to the `/var/slaves/` dir on the slave.

```
named[18338]: client @0x7fd5dc0d4f30 10.0.2.9#58196 (olympus.local): transfer of 'olympus.local/IN': AXFR started
delphos.olympus.local named[18338]: client @0x7fd5dc0d4f30 10.0.2.9#58196 (olympus.local): transfer of 'olympus.local/IN': AXFR ended
delphos.olympus.local named[18338]: client @0x7fd5dc0c6790 10.0.2.9#52508: received notify for zone 'olympus.local'
```

```
-rw-r--r--. named named object_r:named_cache_t:s0 olympus.local.zone
```

It is a non readable cache file. Test the slaves by stopping the master service temporarily.

A zone serial number is a version number for the SOA record. When the serial number changes in a zone file, this alerts secondary nameservers that they should update their copies of the zone file via a zone transfer. The serial number should be changed every time a zone is updated.

To reload the configuration and zone files on the master

``` bash
rndc reload ['ZONE']
```

To manually re-transfer the zone files to the slave. Executed from the slave

``` bash
rndc retransfer 'ZONE'
```
#### Security

The **named** user must have read and write permissions over the files and dirs related to the service.

Port **53 TCP**: used for zone transfers
Port **53 UDP**: used for DNS queries

By default BIND transfers zones to any computer. To dump the contents of a zone

``` bash
dig axfr @'DNS_SERVER' 'ZONE'
```

[[gitea/LINUX 1/NETWORK SERVICES/DNS/files/dig%axfr#AXFR]]

```
[root@prometheus ~]# dig axfr @10.0.2.7 olympus.local

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> axfr @10.0.2.7 olympus.local
; (1 server found)
;; global options: +cmd
olympus.local.          10800   IN      SOA     olympus.local. root.olympus.local. 0 86400 3600 604800 10800
olympus.local.          10800   IN      NS      delphos.olympus.local.
olympus.local.          10800   IN      NS      ns1.olympus.local.
olympus.local.          10800   IN      MX      10 mail.olympus.local.
debian12.olympus.local. 10800   IN      A       10.0.2.10
delphos.olympus.local.  10800   IN      AAAA    fe80::20c:29ff:fe78:4cb1
delphos.olympus.local.  10800   IN      A       10.0.2.7
dns.olympus.local.      10800   IN      CNAME   delphos.olympus.local.
ns1.olympus.local.      10800   IN      A       10.0.2.9
prometheus.olympus.local. 10800 IN      A       10.0.2.4
olympus.local.          10800   IN      SOA     olympus.local. root.olympus.local. 0 86400 3600 604800 10800
;; SERVER: 10.0.2.7#53(10.0.2.7)
;; XFR size: 13 records (messages 1, bytes 350)
```

To protect the DNS db, restrict zone transfers to specific computers or networks. Add allow-transfer { SLAVE_IP; }; to a zone

`/etc/named.conf` -> master
```
zone "olympus.local" IN {
        type master;
        file "olympus.local.zone";
        allow-transfer { 10.0.2.9; };
        notify yes;
};
```

Now addresses outside of the scope will be denied transfers

`/etc/named.conf`
```
[root@prometheus ~]# dig axfr @10.0.2.7 olympus.local

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> axfr @10.0.2.7 olympus.local
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

##### SELinux

[[LINUX/SELINUX/SELinux Working Modes]]

SELinux booleans related to the service

```
[root@ns1 slaves]# getsebool -a | grep named
named_tcp_bind_http_port --> off
named_write_master_zones --> on
```

`named_write_master_zones` should be on for the named user to be able to write zone files

##### DNSSEC and TSIG

Generate the **ZSK**, Zone Signing Key

``` bash
dnssec-keygen -a RSASHA1 -b 512 -n ZONE 'ZONE'
```

```
Generating key pair.......++++++++++++ ....++++++++++++
Kolympus.local.+005+53451
```

Generate the **KSK**, Key Signing Key

``` bash
dnssec-keygen -f KSK -a RSASHA1 -b 4096 -n ZONE 'ZONE'
```

The two key pairs

```
-rw-r--r--. 1 root root dsset-olympus.local.
-rw-r--r--. 1 root root Kolympus.local.+005+38713.key
-rw-------. 1 root root Kolympus.local.+005+38713.private
-rw-r--r--. 1 root root Kolympus.local.+005+53451.key
-rw-------. 1 root root Kolympus.local.+005+53451.private
```

The .key files contain the public portion of the DNSKEY record

`Kolympus.local.+005+53451.key`
```
olympus.local. IN DNSKEY 256 3 5 AwEAAd1RzPufq4TFiNYrRD/deenr8YdPnQXRnOwYajJW T6YiOIF/MJvI7jMG6LwcDyI7G8=

# anatomy
ZONE. IN DNSKEY ALGO FLAGS PROTOCOL PUBLIC_KEY
# 256 - ECDSA, 257 - RSA
```

Add the public keys to the zone file

``` bash
cat Kolympus.local.*.key >> /var/named/olympus.local
```

Sign the zone files

``` bash
dnssec-signzone -N increment -o olympus.local /var/named/olympus.local.zone
```

This will generate a new signed zone file - ZONE.signed. Edit the zone definition to point to the new signed zone file and restart the service. Zone files must be owned by **named**
Every record in the signed zone file will have an RRSIG now.

``` bash
chown named /var/named/*.zone.signed
```

To query for the DNSKEY record

``` bash
dig DNSKEY @MASTER_DNS 'ZONE'. + multiline
```

```
[root@delphos named]# dig @10.0.2.7 DNSKEY olympus.local. +multiline

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> @10.0.2.7 DNSKEY olympus.local. +multiline
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62024
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;olympus.local.         IN DNSKEY

;; ANSWER SECTION:
olympus.local.          10800 IN DNSKEY 257 3 5 (
                                AwEAAcY+Mbt+0cKzEaRpowIPpbi
                                4c4llHZUw/wK13tuyO3j+D4VDWG
                                ZkwH6tk7EMUR4B1BcBgYk6NuDun
                                Emz/vx3penA0MefPTxl7yjOHtRX
                                pTMe8MfCqwStqaXtbaF4MfEeWqy
                                mEylFR8mUXXbfSVljpB08uo4YcW
                                ) ; KSK; alg = RSASHA1 ; key id = 38713
olympus.local.          10800 IN DNSKEY 256 3 5 (
                                AwEAAd1RzPufNYrRD/deenr8YdP
                                nQXRnOwYajJWTldeI7jMG6LwcDyI
                                7G8=
                                ) ; ZSK; alg = RSASHA1 ; key id = 53451
;; Query time: 0 msec
;; SERVER: 10.0.2.7#53(10.0.2.7)
;; WHEN: Mon Apr 08 09:13:56 EEST 2024
;; MSG SIZE  rcvd: 658
```

The keys should be put in `/etc/named.conf` in the trusted-keys section. Omit 'IN DNSKEY'

`/etc/named.conf` -> master
```
options {
...
};
...
trusted-keys {
ZONE ALGO FLAGS PROTOCOL "PUB_KEY";
};
```

To verify the authenticity of a record on the client side

``` bash
dig +sigchase "RECORD" @'SERVER'
```

```
;; WE HAVE MATERIAL, WE NOW DO VALIDATION
;; VERIFYING A RRset for delphos.olympus.local. with DNSKEY:53451: success
;; OK We found DNSKEY (or more) to validate the RRset
;; Now, we are going to validate this DNSKEY by the DS
;; the DNSKEY isn't trusted-key and there isn't DS to validate the DNSKEY: FAILED
```

The dig +sigchase looks for the DNSKEY in **/etc/trusted-key.key** file
They can be obtain either from the .key files, generated on the server or by dig-ing them

``` bash
dig +noall +answer @'SERVER' -t dnskey 'ZONE' >> /etc/trusted-key.key
```

```
olympus.local.          10800   IN      DNSKEY  257 3 5 AwEAAcY+Mbt+VPxpXmDLK/F/9Iwgc0cKzEaRpowIPpbi4c4llHZUw/wKR
olympus.local.          10800   IN      DNSKEY  256 3 5 AwEAAd1RzPufruhmT/nykn4cq4TFiNYrRD/deenr8YdPnQXRnOwYajJW 6YiOIF/MJvYLOl=
```

Now the record signature can be properly verified against the DNSKEY

```
;; WE HAVE MATERIAL, WE NOW DO VALIDATION
;; VERIFYING A RRset for delphos.olympus.local. with DNSKEY:53451: success
;; OK We found DNSKEY (or more) to validate the RRset
;; Ok, find a Trusted Key in the DNSKEY RRset: 53451
;; VERIFYING DNSKEY RRset for olympus.local. with DNSKEY:53451: success

;; Ok this DNSKEY is a Trusted Key, DNSSEC validation is ok: SUCCESS
```
#### References

```
Apress – Learn CentOS Linux Network Services by Antonio Vazquez 
```
