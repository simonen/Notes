Resources: 
`Wiley - DNS Security Management 2017`
`Apress – Pro Linux System Administration 2nd 2017`
www.cloudflare.com/dnssec/root-signing-ceremony/
www.cloudflare.com/dnssec/how-dnssec-works/
 
[[gitea/LINUX 1/NETWORK SERVICES/DNS/BIND#DNSSEC and TSIG]]

**DNS** requests are connectionless. ie. **UDP**. TCP used only when a UDP packet is too small to contain the response. 

**cache poisoning**: common attack which fills DNS caches with faulty data. Clients can be directed to fake servers.

**DNSSEC**: DNS Security Extensions. Provides a means of detecting unauthorized zone file changes by signing them. validates data before storing it in cache. Uses `Chain of Trust` model

**TSIG**: Transaction signature. Provides a means of authenticating updates to a DNS database

Zone Signing Key, ZSK: 
Key Signing Key, KSK:
#### DNSSEC Record Types

- `DNSKEY`: the public key of the zone signing key pair. Used to validate the RRSIG signature to verify the authenticity of the RRset
- `NSEC` - Next
- `DS` - Delegation Signer Record. Contains the hash of a DNSKEY record. Used by the resolver to validate the authenticity of the public key signing key. Performs a hash on the pk sk and compares that to the DS record.
- `RRSIG`: Resource Record Signature

##### The DS Records

DS records are used by the parent zone (e.g., "local") to authenticate the DNSKEY records in the child zone (e.g., "olympus.local"). To establish DNSSEC validation for "olympus.local" on the client side, you would need to configure the DNS resolver with these DS records as trust anchors. This ensures that the DNS resolver can verify the authenticity of the DNSKEY records in the "olympus.local" zone and validate DNSSEC signatures.

#### Cryptography

> dsset-olympus.local.

```
[root@delphos ~]# cat dsset-olympus.local.
olympus.local.          IN DS 38713 5 1 05217DA4F7CA070C3EABDDF5540CAFD0D5FB8322
olympus.local.          IN DS 38713 5 2 CF80937341D04BD76004430CE4508C58A7E62DAC444463EFFC3B406F 7F7BBACC
```

```
olympus.local. IN DS <key_tag> <algorithm> <digest_type> <digest>
```

- `<key_tag>`: A numerical value that uniquely identifies the DNSKEY record to which this DS record corresponds.
- `<algorithm>`: The cryptographic algorithm used to create the digest.
- `<digest_type>`: The type of digest used. This can be either 1 (SHA-1) or 2 (SHA-256).
- `<digest>`: The cryptographic digest of the corresponding DNSKEY record.

Kolympus.local.+005+38713.key
Kolympus.local.+005+38713.private
Kolympus.local.+005+53451.key
Kolympus.local.+005+53451.private

#### BIND-CHROOT

Packages: 
`bind-chroot`

The **bind-chroot** will replicate the normal named environment under a new root directory **/var/named/chroot**, transferring existing zone and conf files and preserving their paths.

``` bash
ls /var/named/chroot
```

```
drwxr-x---. 2 root named  44 Apr  8 22:20 dev
drwxr-x---. 4 root named 187 Apr  8 22:21 etc
drwxr-x---. 3 root named  19 Apr  8 22:11 run
drwxr-xr-x. 3 root root   19 Apr  8 22:11 usr
drwxr-x---. 5 root named  52 Apr  8 22:11 var
```

Stop the named.service if running, and start the named-chroot service

```
systemctl enable --now named-chroot
```

```
named     2164  Ssl  /usr/sbin/named -u named -c /etc/named.conf
```

The chrooted name process

```
named    11702  Ssl  /usr/sbin/named -u named -c /etc/named.conf -t /var/named/chroot
```
