---
tags:
  - dns
---
[[gitea/LINUX 1/NETWORK SERVICES/DNS/BIND]]

TO BE REWORKED!!!

Domain Name System. Hierarchical system that stores info about network hosts. In Kerberized environment, it stores info about services available in a network.

#### DNS Hierarchy

[[gitea/LINUX 1/NETWORK SERVICES/DNS/DNS Hierarchy.canvas|DNS Hierarchy]]

1. **root domain** (.)
2. **top-level domains**: `.com, .net, .org, .cn, .de, etc`.
3. **second-level** domain: redhat.com, etc.
4. **subdomain**: www.redhat.com

#### Terminology

##### DNS Server Types

* **Recursive DNS Resolver** : If the requested domain is not it the cache, the server performs a recursive query: `Root -> TLD -> Authoritative` 
* **Cache-only DNS Server**: 
	* Runs as a **recursive resolver**.
	* Does not hold records for a zone, therefore is not authoritative for any domain
	* Caches the results locally for future queries.
* **Forwarder**: If requested domain not found in cache, redirects the query to an upstream resolver. 
* **Master DNS Server**: Holds copies of names and IP addresses of computers belonging to a domain within a zone. Can add, change or delete records
* **Slave DNS Servers**: Read-only. Has all the information to answer queries about a domain, cannot change records. 

* **domain**: a resource record that ends in a common name. Can contain subdomains
* **top-level domains**: the highest hierarchy in DNS names. .com, .net, .bg, .us, etc.
* **sub-domain**: a domain that is a branch in another domain
* **nameserver**: the server responsible for the resource records in a zone. For redundancy, more than one ns is used per domain
* **resource record**: database record that contains specific types information, managed by DNS server. Different types of rr exist.
* **zone**: the branch of the DNS tree for which a specific name server is responsible.

#### DNS Servers

- **BIND**: Authoritative, recursive. Mature. Used in ISPs, enterprise, academic networks
- **Unbound**: Recursive. ISPs, privacy DNS providers
- **PowerDNS**: Authoritative, recursive. Hosting providers. API-driven. DB backends.
- **Microsoft DNS**: Authoritative, recursive. Enterprise. AD integration.
- **Knot DNS/NSD**: Authoritative. TLDs, registries. 
#### DNS Managers

`nsswitch.conf`: specifies the order of which DNS resource providers are contacted

`nsswitch.conf`
```
hosts:      files dns myhostname
```

In order of appearance:
1. **`files`:** `/etc/hosts`
2. **`dns`**: available DNS servers (`resolv.conf`)
3. `myhostname`

`myhostname` : this will resolve the ip address to the host's own hostname without the need to be included in the `hosts` file.
#### Zones

Domains are described as zones, that are defined in zone files. Zone files contain a header, aka SOA  record. Zone header fields:
* `$ORIGIN` .: Defines the start of the zone. Ends with a . do define the end of the zone.
* `TTL`: Time To Live in seconds. Default expiration for records that do not have their own 
* `SOA` (Start of Authority): not strictly part of the header, it is the first record in the zone file. Includes essential info about the zone:
	* `Master`: Primary authoritative DNS server for the domain
	* `Contact`: Email address of the admin responsible for the zone (@ is replaced by .)
	* `Serial`: Zone file version. Used by slaves
	* `Refresh`: How often slave servers should update their copy of the zone
	* `Retry`: The interval between attempts to refresh a slave server
	* `Expire`: How long a slave server is allowed to use any version of the zone file
	* `Negative cache TTL`:  How long a failed zone lookup result might be cached

#### DNS Lookups

Utilities: 
`dig`, `host`, `whois`

Each device connected to the Internet is configured with a DNS resolver, containing the IP addresses of up to three DNS name servers

If a DNS server cannot be reach, another one in the list is contacted.
If a DNS server can be reached and it does NOT have the wanted record, it does NOT look further for it within the other name servers

`/etc/resolv.conf`: **DNS** resolvers on a Linux system. If managed by the `NetworkManager`, any direct modifications will be undone after the `NetworkManager` service is reset.

* **Authoritative answer**: comes from a nameserver that is responsible for a zone
* **Local Authoritative Data**:  The answer is provided by a DNS server, responsible for the requested data. The answer is in resource records in a local zone
* **Remote non-authoritative via recursion**: 
* **Local cached non-authoritative data**: Cached records are fast as they do not require recursion. The time a record is allowed to be kept in cache is determined by the TTL. Once a record is expired it becomes invalid and recursion happens over again.

**TTL, Time to Live**: The amount of time a DNS answer can be kept in cache.

**Recursive Queries**: When a client requests resolution for a domain name, the DNS server might not have the information directly available. In such cases, it needs to query other DNS servers on the internet to resolve the domain. Recursive queries involve the DNS server querying other servers on behalf of the client until it finds the requested information.

To verify if a specific server is providing an answer:

``` bash
dig "DOMAIN"
```

```
[kimchen@ipa ~]$ dig example.com

; <<>> DiG 9.16.23-RH <<>> example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 37620
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;example.com.                   IN      A

;; Query time: 1 msec
;; SERVER: 192.168.137.6#53(192.168.137.6)
;; WHEN: Sun Jan 28 21:23:49 EET 2024
;; MSG SIZE  rcvd: 40
```

To query a DNS server, pass it as a parameter

``` bash
dig @"DNS_SERVER" ["RESOURCE"]
```

Dig status information indicators:
* `NOERROR`: DNS resolving successful
* `NXDOMAIN`: (Non-Existent Domain) Requested DNS info not found
* `SERVFAIL`: Error contacting a vital DNS server
* `REFUSED`: The query is coming from a network or address that is not allowed by the server

flags:
* `qr` (Query / Response): 
* `aa` (Authoritative Answer): This flag indicates whether the responding DNS server is **authoritative** for the domain being queried. This is relevant when the DNS server itself holds the zone records for the domain being queried
* `rd` (Recursion Desired): The client requested recursion (`rd` = 1). When set, the client is asking the DNS server to perform the entire lookup process, including contacting other DNS servers, if necessary.
* `ra` (Recursion Available): The server supports recursion (`ra` = 1). This flag is set in the response and indicates whether the DNS server supports **recursion**.
* `ad` (Authenticated Data): Indicates whether the data in the response has been **cryptographically verified**.

To query for a specific resource record in a domain

``` bash
dig "RESOURCE_RECORD" "DOMAIN" ["@NAMESERVER"]
```

Reverse lookup

``` bash
dig -x "IP_ADDRESS"
```

Output can be limited to the answer section only

``` bash
dig +short SOA google.com @ns1.google.com
```

```
ANSWER
ns1.google.com. dns-admin.google.com. 632436250 900 900 1800 60
```

`<ns> <zone admin email> <zone serial#> <refresh> <retry> <expiry> <nxttl>`

`nx (negative cache) ttl`: tells other dns servers to cache negative results (NXDOMAIN)
to prevent authoritative nameservers from performing continuous lookups for a host that does not exist. This is the period for which a DNS server will respond with "No such host" after the initial request before querying the authoritative servers again. Time in seconds. Could be up to a week.

`forwarder`: Upstream DNS server. Configured per zone. 
#### Resource Records

Resource records contain different types of data:
* **Type**: 
	* `A`: maps a hostname to an IPv4 address
	* `AAAA`: maps a hostname to an IPv6 address
	* `CNAME` (Canonical Name): An alias for one name to another name 
	* `PTR` (Pointer): Maps an IP address to a hostname
	* `NS` (Name Server): maps a domain name to a name server that is authoritative for the DNS zone
	* `SOA` (Start of Authority): Contains info about who is responsible for administration of the domain
	* `MX` @(Mail Exchange): indicates which MTA mail servers are used in a domain
	* `TXT` (Text): Maps a name to human readable text. Used to verify the name of the domain the mail message was received from
	* **SRV** (Service): indicates which host to contact for specific services such as LDAP or Kerberos
* **Data**
* **Class**: the class tells the DNS server what type of network the resource is for.
	* IN: Internet
	* CH: CHAOSNet
	* HS: Hesiod
* **TTL**: Time to Live. In seconds.

#### Cache-Only DNS Servers

Does not hold RR, queries upstream DNS servers for client requests.
Forwarding
Recursion

Query the local DNS caching server

``` bash
host "www.google.com" localhost
```

