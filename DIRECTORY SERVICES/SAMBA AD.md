---
tags:
  - centos
  - dns
  - kerberos
---
This note covers the CentOS 7 installation (from source) and configuration of SAMBA AD 

#### Installation

[Download Samba](https://download.samba.org/pub/samba/stable/)
[Package Dependencies](https://wiki.samba.org/index.php/Package_Dependencies_Required_to_Build_Samba)

Ports:

Download and unpack the archive.  The `./bootstrap/generated-dists/'your_os'` contains a `boostrap.sh` script for installing all the required dependencies. Execute the boostrap.sh script

```
[root@dc1 generated-dists]# ls
centos8s  centos9s  debian11  debian11-32bit  debian12  debian12-32bit  fedora40  opensuse155  ubuntu1804  ubuntu1804-32bit  ubuntu2004  ubuntu2204  Vagrantfile
```

Build, make and install

``` bash
./configure --sysconfdir=/etc/samba /
	--with-systemd /
	--systemd-install-services /
	--with-systemddir=/usr/lib/systemd/system /
	--bindir=/usr/local/bin 

make -j"CPU_CORES"
make install
```

Naming conventions

- Hostname = `DC1`
- DC local IP Address = `10.99.0.1`
- Authentication Domain = `SAMDOM.EXAMPLE.COM`
- Top level Domain = `EXAMPLE.COM`

The DC should use itself as a DNS server. 

`/etc/resolv.conf`
```
search samba.ohio.net
nameserver 127.0.0.1
```

`/etc/hosts`
```
127.0.0.1 localhost
192.168.137.10 dc1 dc1.samba.ohio.net
```

Install bind 

``` bash
yum install bind bind-utils
```

Remove the `/etc/samba/smb.conf` file before executing the `samba-tool`

Set up the AD DC. Values in square brackets are defaults. 

``` bash
samba-tool domain provision --user-rfc2307 --interactive
Domain [SAMBA]: 
Server Role (dc, member, standalone) [dc]:
DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]: BIND9_DLZ  
Administrator password:
Retype password:
```

IPC$ (interprocess connection for communicating with Samba).

**DNS** (BIND9_DLZ)

`/usr/local/samba/bind-dns/named.conf`
```
dlz "samba.ohio.net" {
    # For BIND 9.11.x
     database "dlopen /usr/local/samba/lib/bind9/dlz_bind9_11.so";
};
```

Include the file in the main named.conf file

`/etc/named.conf`
```
include `/usr/local/samba/bind-dns/named.conf`
```

If BIND had been installed prior to the domain provisioning, the `samba-tool` will handle ownership and permissions on the `/usr/local/samba/bind-dns` folder and files, required by BIND.

``` bash
chown -R :named /usr/local/samba/bind-dns
```

Because BIND needs to read a file that is outside of `/etc`, SELinux will block access to the `/usr/local/samba/bind-dns/named.conf` file. Turn it off temporarily

Run BIND 

``` bash
systemctl enable --now named
```

Start the samba daemon

``` bash
systemctl enable --now samba
```

Test Samba by listing available shares

``` bash
smbclient -L localhost -N
#Anonymous login successful

#        Sharename       Type      Comment
#        ---------       ----      -------
#        sysvol          Disk
#        netlogon        Disk
#        IPC$            IPC       IPC Service (Samba 4.21.0)
```

``` bash
smbclient //localhost/netlogon -UAdministrator -c 'ls'
#  .                                   D        0  Sun Sep 29 23:36:15 2024
#  ..                                  D        0  Sun Sep 29 23:36:15 2024
```

Verify DNS. For Samba to act as a domain controller, it must respond to the following DNS record requests:

``` bash
dig @localhost -t SRV _ldap._tcp.samba.ohio.cc +noall +answer
dig @localhost -t SRV  _kerberos._udp.samba.ohio.cc +noall +answer
dig @localhost -t A  dc1.samba.ohio.cc +noall +answer
```

```
;; ANSWER SECTION:
_ldap._tcp.samba.ohio.cc. 900   IN      SRV     0 100 389 dc1.samba.ohio.cc.
_kerberos._udp.samba.ohio.cc. 900 IN    SRV     0 100 88 dc1.samba.ohio.cc.
dc1.samba.ohio.cc.      900     IN      A       192.168.137.10
```

Testing Kerberos

The samba-generated `krb5.conf` file looks like this:

`/usr/local/samba/private/krb5.conf`
```
[libdefaults]
        default_realm = SAMBA.OHIO.CC
        dns_lookup_realm = false
        dns_lookup_kdc = true

[realms]
SAMBA.OHIO.CC = {
        default_domain = samba.ohio.cc
}

[domain_realm]
        dc1 = SAMBA.OHIO.CC
```

`dns_lookup`: determines whether the SRV records are obtained via DNS lookups

Symlink it to `/etc/krb5.conf `

```
ln -sf /usr/local/samba/private/krb5.conf /etc/krb5.conf
```

Get a kerberos ticket

``` bash
kinit administrator@SAMBA.OHIO.CC
# Password for administrator@SAMBA.OHIO.CC:
# Warning: Your password will expire in 41 days on Sun 10 Nov 2024 10:36:28 PM EET
```

``` bash 
klist
```

```
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: administrator@SAMBA.OHIO.CC

Valid starting       Expires              Service principal
09/30/2024 00:14:10  09/30/2024 10:14:10  krbtgt/SAMBA.OHIO.CC@SAMBA.OHIO.CC
        renew until 10/01/2024 00:14:05
```

The samba-generated smb.conf

[[LINUX/NETWORK STORAGE/SAMBA/Service Sections and Directives]]

`/etc/samba/smb.conf`
```
# Global parameters
[global]
        netbios name = DC1
        realm = SAMBA.OHIO.NET
        server role = active directory domain controller
        server services = s3fs, rpc, nbt, wrepl, ldap, cldap, kdc, drepl, winbindd, ntp_signd, kcc, dnsupdate
        workgroup = SAMBA
        idmap_ldb:use rfc2307 = yes

[sysvol]
        path = /usr/local/samba/var/locks/sysvol
        read only = No

[netlogon]
        path = /usr/local/samba/var/locks/sysvol/samba.ohio.net/scripts
        read only = No
        
```

**Special Samba Services**

- `[global]`:
	Global directives apply to the entire Samba server and configure general settings that affect all shares and services.
- `[netlogon]`:
	This share is a special service in Samba that provides domain clients (Windows machines) with access to logon scripts and handles authentication requests. 
- `[sysvol]`:
	This share is essential in Samba's role as an **Active Directory Domain Controller**. It stores **Group Policy Objects (GPOs)**, scripts, and other domain-specific data that need to be replicated and distributed across all Domain Controllers in the network.
	
#### Firewall

``` bash
firewall-cmd --add-service=samba --permanent ; firewall-cmd --reload
```
