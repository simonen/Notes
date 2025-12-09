Port 636 (ldaps://) enforces TLS

Port 389 (ldap://) STARTTLS. 

> `SELinux` context of the cert files must be right, otherwise it will be blocking access to certs, resulting in `implementation error`

```
drwxr-xr-x. 2 system_u:object_r:slapd_cert_t:s0 root root  /etc/openldap/certs/
```

```
denied  { read } for  pid=1593 comm="slapd" name="cacert.pem" dev="dm-0" scontext=system_u:system_r:slapd_t:s0 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file permissive=0
```

The `ldap` user must have access to the keys

Restore the context of the certs 

``` bash
restorecon 'FILE'
```

To see the currently configured TLS attributes

``` bash
slapcat -b "cn=config" | grep -i tls
```

To list all available TLS attributes

``` bash
ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config | grep -i tls
```

Put the certs in `/etc/openldap/certs`
To change the certificate paths, create a new `.ldif` file

`/etc/openldap/slap.d/tls.ldif`
```
dn: cn=config
changeType: modify
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/certs/ldap.ohio.cc.cert
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/ldap.ohio.cc.key
-
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/cacert.pem
-
```

Apply the changes

``` bash
ldapmodify -Y EXTERNAL -H ldapi:// -f tls.ldif
```

`/etc/openldap/ldap.conf`
```
URI ldap[s]://ldap.ohio.cc/
TLS_CACERT /etc/openldap/cacerts/ca.cert.pem
TLS_REQCERT [allow, demand, try, never]
```

To enforce `ldaps` only (CentOS 7)

`/etc/sysconfig/slapd`
```
SLAPD_URLS="ldapi:/// ldaps:///"
```

Test the TLS connection 

(STARTTLS)

``` bash
 openssl s_client -connect ldap.ohio.cc:389 -starttls ldap -CAfile /etc/openldap/certs/cacert.pem
```

Forced TLS

``` bash
 openssl s_client -connect ldap.ohio.cc:636 -CAfile /etc/openldap/certs/cacert.pem
```

This should not work. ldaps:// only should.

``` bash
ldapwhoami -H ldap://ldap.ohio.cc -xv -D uid=fitka,ou=people,dc=ohio,dc=cc -w 123123
```

```
slapd[31151]: conn=1022 fd=11 ACCEPT from IP=192.168.137.10:44136 (IP=0.0.0.0:636)
Oct 11 20:28:37 ldap.ohio.cc 
slapd[31151]: conn=1022 fd=11 TLS established tls_ssf=256 ssf=256
```

To enforce TLS on database operations and binds

`database`
```
olcSecurity: ssf=0 update_ssf=128 simple_bind=128 tls=256
```

Configure server-client mutual TLS verification

`cn=config`
```
olcVerifyClient: [never, allow, try, demand]
```

- `never`: The server never asks for client's certificate
- `allow`: The server asks for client's certificate but allows connection regardless
- `try`: The server asks for client's certificate. If none is given, connection proceeds. If the client certificate is invalid, the connection is terminated immediately.
- `demand`: The server allows connection only if client's certificate is valid. Most restrictive and probably used for server-to-server communication, i.e., replication.

Use the client certificates as arguments. This is mandatory if `olcVerifyClient` is set to `demand`

```
ldapsearch -H ldap://dc.ohio.cc -x -b dc=ohio,dc=cc -ZZ \
  -D "cn=admin,dc=ohio,dc=cc" \
  -w 123123 \
  -ZZ \
  -o tls_cert=/etc/openldap/certs/headoffice.ohio.cc.cert \
  -o tls_key=/etc/openldap/certs/headoffice.ohio.cc.key \
  -o tls_cacert=/etc/openldap/certs/cacert.pem
```

Private key, server cert and CA can be combined into a single .pem file

``` bash
cat server.key server.crt ca.crt > combined.pem
```