
##### Unbound Caching Server

packages to configure DNS cache servers:
* **bound**: recommended solution for **RHEL**. More secure than other packages
* bind
* dnsmasq

``` bash
yum install -y unbound ; systemctl enable --now unbound
```

Main config file: 
`/etc/unbound/unbound.conf`

Unbound listens on localhost by default.
Unbound does not accept client connections by default

To accept incoming connections on any interface (in `unbound.conf`):
`interface: 0.0.0.0` 

To accept incoming client connections:

```
access-control: NETWORK/MASK allow
```


To forward DNS requests to another DNS server, forward zone for the root (.) domain is configured

```
forward-zone:
	name: "."
	forward-addr: IP_ADDRESS
```

To check the `unbound.conf` for errors:

```bash
unbound-checkconf
```

To bypass validation for a domain:

```
domain-insecure <DOMAIN>
```

To create keys that do not exist

**$ unbound-control-setup**

##### Trust Anchors

To get the trust anchor for a domain
**$ dig +dnssec DNSKEY** *DOMAIN*

#### Troubleshooting DNS Issues

##### Dumping and Changing the Unbound Cache

To dump the contents of the **unbound** cache:
**\# unbound-control dump_cache >** *CACHE_FILE*

To load the dumped file contents back into the unbound cache
**\# unbounc-control load_cache** < *CACHE_FILE*

To purge an outdated record:
\# **unbound-control flush** *RECORD* 