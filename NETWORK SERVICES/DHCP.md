---
tags:
  - dhcp
  - centos
  - dns
---
Dynamic Host Configuration Protocol

Packages
- `dhcp-server`

Service
- `dhcpd` (CentOS)
- `isc-dhcps-server` (Ubuntu)

Ports **67, 68 UDP** for IPv4

Ports **547 UDPv6** for IPv6

#### Files and Directories

Main configuration files
- `/etc/dhcp/dhcpd.conf`
- `/etc/dhcp/dhcpd6.conf`

Sample configuration files
- `/usr/share/doc/dhcp-4.2.5/`

Leases file
- `/var/lib/dhcpd/dhcpd.leases`

Before starting the `dhcpd.service`, the server must have at least one configured subnet to lease IP addresses to and an active interface in that subnet. The server will automatically start listening on any interface if its IP address fall within the defined subnets in the conf file.

Minimal configuration. The clients will get IP and subnet mask only.

`/etc/dhcp/dhcpd.conf`
```
subnet 10.0.3.64 netmask 255.255.255.192 {
  range 10.0.3.80 10.0.3.92;
```

Ubuntu: INTERFACE=*INTERFACE*. Listening interface must be declared for the service to start
CentOS: Will listen on any interface associated with a subnet declaration in the .conf file

More options. Use double quotes for strings.
* `option domain-name-servers DNSERVER1, DNSERVER2`: ;
* `option routers DEFAULT_GATEWAY_ADDR` ;
* `option domain-name "DOMAIN_NAME"`;
* `option broadcast-address`: 
* `option subnet-mask "SUBNET_MASK"`

#### Static Lease Configurations

> Statically assigned IP addresses must be outside defined IP ranges!

Static configurations could be done by:
1. Binding a static IP address to a specified MAC address of the requesting client
2. If a resolvable FQDN is given, the IP address is taken from its A record 

Groups of hosts can be defined within a subnet

```
subnet ... {
	group "NAME" {
		host "HOSTNAME" {
			hardware ethernet "MAC_ADDRESS" # ":"-delimited
			fixed-address "IP | FQDN"
		}
		host "HOSTNAMEX" {
			...
		}
	}
}
```

Define a single host for static IP allocation

```
host HOSTNAME {
hardware ethernet MAC_ADDRESS;
fixed-address IP_ADDRESS | FQDN;
options ...
}
```

IP ranges can be organized in pools that allow or deny unknown client requests in favor of static assignments. In this case the range directive is no longer necessary

```
subnet ... {
	options ...
	pool {
	IP-RANGE_START IP_RANGE_END;
	allow | deny unknown clients;
	default-lease-time SECONDS;
	max-lease-time SECONDS;
	}
}
```
#### DHCP Clients

man `dhclient`

`/etc/dhcp/dhclient.conf``

To release an a DHCP configuration. No interface applies the command on all interfaces.

``` bash
# Linux
# Release
dhclient -r "INTERFACE"
# Renew
dhclient -v "INTERFACE"

# Windows
ipconfig /release
ipconfig /renew
```

When working with dynamic DNS updates, the DHCP client can be configured to send its hostname to the DHCP server, which in turn will use that hostname to update the client's DNS records accordingly when IP changes occur. 

Sending hostname to DHCP server. This will override the actual hostname of the machine.

``` bash
dhclient -v -H "HOSTNAME" "INTERFACE"
```

Another way

`/etc/dhcp/dhclient.conf`
```
send host-name "HOSTNAME";
```

Another way

``` bash
nmci con-mod "CONN" ipv4.dhcp-hostname "HOSTNAME"
```

```
[root@server9 dhcp]# nmcli con sh loc | grep ipv4.dhcp
ipv4.dhcp-send-hostname:                yes
ipv4.dhcp-hostname:                     kravar4o
ipv4.dhcp-fqdn:                         --
```

```
May 13 21:05:44 delphos.olympus.local named[18937]: client @0x7f5fdee53160 10.0.2.6#49793/key aphrodite: updating zone '2.0.10.in-addr.arpa/IN': adding an RR at '153.2.0.10.in-addr.arpa' PTR kravar4o.olympus.local.
```
#### DHCP and DNS Dynamic Update

Package
- `bind-utils `

man **nsupdate**

**nsupdate** - Dynamic DNS update utility

DHCP servers must be able to update the DNS records when changing IP addresses. 

Requirements
1. `ddns-update-style interim | standard`
2. Generate the keys with `dnssec-keygen`
3. Add the DNS zone in the `dhcpd.conf`
4. Add the key in `dhcpd.conf`
5. Add the key in `named.conf`
6. `allow-update` to the zone block in `named.conf`

`/etc/named.conf`
```
options {
	...
};
zone "ZONE" {
	...
	allow-update { "REMOTE OR LOCALHOST"; };
}
```

`/etc/dhcp/dhcpd.conf`
```
ddns-update-style interim;
ignore client-updates;

# Add the zones that should be updated by dhcpd

zone "ZONE" {
	primary "DNS_SERVER";
}
zone "REVERSE_ZONE" {
	primary "DNS_SERVER";
}
```

`ddns-update-style`
* `interim`: uses TXT records as DHCID associated with DNS records
* `standard`: uses RR as DHCID

`ignore client-updates:` will not update existing records. 

Changing DNS records can be achieved remotely by the `nsupdate` command. 

To delete remotely a DNS record

``` bash
nsupdate > server "DNS_SERVER"
nsupdate > update delete "DNS_RECORD"
nsupdate > send 
nsupdate > CTRL + C
```

To add a DNS record.

``` bash
nsupdate > server "DNS_SERVER"
nsupdate > update add "DNS_RECORD" "TTL" "RECORD_TYPE" "IP"
nsupdate > send 
nsupdate > CTRL + C
```

Before adding CNAMEs, make sure there any existing already. man nsupdate for info

Before changing the zone files, bind stores zone changes in a jnl (journal) file. Correct permissions must be applied to the dir and files. 

``` 
# This error occurs when there are differences between the .zone.jnl  and the .zone file. Removing the .zone.jnl file hels reload the zone

zone olympus.local/IN: journal rollforward failed: not exact
zone olympus.local/IN: not loaded due to errors.
```

```
kravar4o.olympus.local. 3600    IN      TXT     "31eba78f185083b4d5a5e225b12faca0de"
kravar4o.olympus.local. 3600    IN      A       10.0.2.153
```

The TXT record indicates that the RR entry was created by the DHCP server. If not present, the A record would not be updated. Zone changes are written to a journal (.jnl) file, associated with the zone file. The data in the journal file supersedes that of the zone file.

A zone can be temporarily frozen to prevent the DNS server from dynamically updating it.

``` bash
rndc freeze ["ZONE"]
```

To unfreeze

``` bash
rndc thaw ["ZONE"]
```
#### Using TSIG Keys

For security reasons, it is good to use signed DHCP servers that can update DNS records.
So, instead of allowing IP addresses that are allowed to update the DNS database, we use keys.

Generate the key pair

``` bash
dnssec-keygen -a HMAC-MD5 -b 128 -n HOST "DHCP_HOSTNAME"
```

 Extract the shared secret and put it in 
 
`/etc/named.conf <- DNS server`
```
key "DHCP_HOSTNAME" {
	algorithm HMAC-MD5;
	secret "SECRET"
	 };

zone "ZONE" {
	allow-update { key "DHCP_HOSTNAME"; }
	};
zone "REVERSE_ZONE" {
	allow-update { key "DHCP_HOSTNAME"; }
	};
```

`/etc/dhcp/dhcpd.conf`
```
key DHCP_HOSTNAME {
       algorithm HMAC-MD5;
       secret "SECRET";
       }

zone ZONE {
       primary SERVER_IP;
       key "DHCP_HOSTNAME";
       }
       
zone "REVERSE_ZONE" {
	primary "DNS_SERVER";
	key "DHCP_HOSTNAME";
	}
```

If successful

```
client @0x7f140 10.0.3.126#39739/key aphrodite: updating zone 'olympus.local/IN': adding an RR at 'ss1.olympus.local' A 10.0.3.83
```
