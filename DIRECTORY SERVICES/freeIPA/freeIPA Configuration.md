#### FreeIPA Server
#### RHEL 7 

- `yum update nss`: fixes the 'CA did not start in 300s' error
- `yum -y install ipa-server bind-dyndb-ldap ipa-server-dns`

Include the *IP_ADDRESS FQDN* in the `/etc/hosts` file

```bash
echo 'IP_ADDRESS FQDN' >> /etc/hosts
```
**digging** the hostname should resolve to the host's IP address. If digging does not resolve to the correct nameserver, set the DNS temporarily to the server's IP.

```bash
dig +short ipa.example.com A
```

 **Configure the IPA server**
 
```bash
ipa-server-install --setup-dns
```

Add the relevant services to the firewall

```bash
for i in http https ldap ldaps kerberos kpasswd ntp dns; do firewall-cmd --permanent --add-service=$i; done
firewall-cmd --reload
```


To open the FreeIPA in a web browser:
http://ipa.server/ipa

Unattended installation:

```bash
ipa-server-install --help
```

```bash
ipa-server-install --setup-dns --no-forwarders --no-reverse --admin-password="PASSWORD" --ds-password='PASSWORD' --unattended --realm='EXAMPLE.COM' --skip-mem-check
```

```
ipa-server-install --setup-dns --no-forwarders --no-reverse --admin-password="123123123" --ds-password='123123123' --unattended --realm=EXAMPLE.COM --skip-mem-check
```
#### FreeIPA Clients

Primary DNS should be set as the IPA server IP
The client might have to be configured to use a NTP server

Install the IPA-client

```bash
yum install -y ipa-client
```

Join the domain:

```bash
ipa-client-install --mkhomedir --enable-dns-updates --force-ntpd
```

#### RHEL 9

RHEL9 uses `chronyd` instead of `ntpd` as NTP server
The IPA server should be configured as an **NTP** server or use a dedicated NTP server. 
The client must time synced with the NTP server

```bash
dig +short -t SRV \_ntp.\_udp.DOMAIN 
```

```
;; ANSWER SECTION:
_ntp._udp.example.com.  86400   IN      SRV     1 1 123 ipa.example.com.
```

`+short`: returns the answer section only

#### DNS Services

FreeIPA provides a DNS server

https://www.freeipa.org/page/V2/DNS_Interface_Design

In order for postfix to be able to receive and send messages, DNS must be configured properly. An **MX** record must be added to the **DNS** server.

For IPA DNS server

```bash
ipa dnsrecord-add 'DOMAIN' @ --mx-rec='PREFERENCE(INT) MAIL_SERVER_FQDN.'
```
- `@` - Record name
- `MX` - Record type
- `PREFERENCE` - integer, the lower the higher priority
- `MAIL EXCHANGER` - Mail server FQDN