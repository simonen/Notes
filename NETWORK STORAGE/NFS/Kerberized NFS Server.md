
Add the **sec**=*method* option on an export

Security methods:
* **none** : Anonymous access based on the **nobody** permissions. Used with **nfsd_anon_write** boolean. Insecure
* **sys**: Default security method. Access is based on **UID** and **GUID**. Incoming user UID is mapped to the same UID on the server, even if names do not match.
* **krb5**: NFS clients prove their identity with a kerberos keytab file. The authentication session must be kerberized as well. **kinit** *username*
* **krb5i**: more secure krb5. Data in the request is guaranteed not be tampered with
* **krb5p**: same as krb5i, adds encryption between server and client. Best security, impact on speed.

#### Establish Kerberized session between NFS client and host:

* `/etc/krb5.keytab`: contains the security principals for the NFS server and client.
	* `klist -k`: to verify the contents of the `krb5.keytab` file
* **kerberized user session**: 
* `sec=METHOD`: both in the share definition and mount option
* `nfs-secure-server.service`: must be active on the server
* `nfs-secure.service`: must be active on the client

The IPA domain should be resolved by the IPA server. To confirm that use the dig cmd:

```bash
dig example.com
```

```
;; QUESTION SECTION:
;example.com.                   IN      A

;; AUTHORITY SECTION:
example.com.            3600    IN      SOA     master.example.com. hostmaster.example.com. 1706902304 3600 900 1209600 3600

;; Query time: 0 msec
;; SERVER: 192.168.137.22#53(192.168.137.22)
```

Create the service principal of the **NFS** server **on the IPA server**:

```bash
ipa service-add nfs/nfsserver.example.com --force
```

`--force`: if DNS error about A record

On the NFS server:

```bash
kinit -k
```

```bash
ipa-getkeytab -s 'ipa.example.com' -k /etc/krb5.keytab -p nfs/nfsserver.example.com
```

`/etc/exports`
```
/SHARE HOSTS(sec=krb5, rw)
```

Change exported dir ownership

```
exportfs -rv
```

On the NFS client:

```bash
kinit -k
```

```
ipa-getkeytab -s 'ipa.example.com' -k /etc/krb5.keytab -p host/host.example.com
```

The `rpc-gssd.service` (`nfs-secure.service`) should be running

```bash
systemctl enable --now nfs-secure
```

```bash
mount -o sec=krb5 NFSSERVER_FQDN:/EXPORT /MOUNTPOINT
```
