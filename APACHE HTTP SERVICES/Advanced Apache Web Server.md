---
tags:
  - apache
  - centos
  - certificates
  - ldap
  - php
---

Make sure the **httpd-manual** module is installed. Opened as a web page \https://apache_up/manual

Essential Apache parameters file

`/etc/httpd/ httpd.conf`
```
root@server1 ~]# cat /tmp/httpd.conf
ServerRoot "/etc/httpd"
Listen 80
Include conf.modules.d/*.conf

User apache
Group apache

ServerAdmin root@localhost

<Directory />
	AllowOverride none
	Require all denied
</Directory>

DocumentRoot "/var/www/html"

<Directory "/var/www">
	AllowOverride None
	# Allow open access:
	Require all granted
</Directory>

# Further relax access to the default document root:
<Directory "/var/www/html">
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>

<IfModule dir_module>
	DirectoryIndex index.html
</IfModule>

<Files ".ht*">
	Require all denied # denies access to all files 
</Files>

ErrorLog "logs/error_log"
LogLevel warn

<IfModule alias_module>
	ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
</IfModule>

<Directory "/var/www/cgi-bin">
	AllowOverride None
	Options None
	Require all granted
</Directory>

IncludeOptional conf.d/*.conf
```

* `ServerRoot`: The directory that contains `conf` files. All `conf` files are relative to this dir
* `Listen`: The interface and port `httpd` listens on. `Listen 80` listens on all interfaces on port 80. `Listen 10.0.2.2:80` listens on the specified address. Ubuntu gets this information from a `ports.conf` file
* `Include`: Directories that contain additional conf files
* `ServerAdmin`: The server admin email
* `Directory`: Used as a block of parameters specific to one dir. Used to determine what contents are allowed. Contains: AllowOverride, Require, Options
* `AllowOverride`: If set to None, httpd will ignore the .**htaccess** file for per-dir settings
* `Options`: Specify several options: **Indexes**, shows a dir listing if index.html cannot be accessed. 
* `Require all`: If set to **Granted**, allows the contents of a dir to be accessed. **Denied**, denies.
* `DirectoryIndex`: Specifies the name of file that should be opened when accessing a dir. Default: **index.html**
* `ErrorLog`: Where errors are logged to
* `LogLevel`: which types of messages should be logged
* `ScriptAlias`: Where Apache looks for scripts that are allowed for execution
* `IncludeOptional`: optional conf files that can be included


1. **AllowOverride None:**
    
    - Prevents `.htaccess` files in this directory from overriding the settings specified here. This can enhance security and performance by centralizing configuration and reducing the overhead of parsing `.htaccess` files.

#### SELinux Related Settings

* `httpd_sys_content_t`: context type: Set on directories that Apache is allowed to access
* `httpd_sys_content_rw_t`: Set on dirs that Apache is allowed to read/write to
* `httpd_sys_script_exec_t`: for dirs that contain executable scripts
* `httpd_unified`: Boolean. Defines Apache restricted policy. Off by default
* `httpd_enable_cgi`: Boolean. On by default. Allows Apache to run scripts
* `httpd_tty_comm`: Boolean. Determines if Apache is allowed to access a TTY. Must be ON if TLS private keys that prompt for password are used
#### Configuring Write Access to The DocumentRoot

By default only the `root` user has write access to the `DocumentRoot`

Create a group `webdev`

``` bash
groupadd webdev
```

Set access permissions to the `DocumentRoot` folder

```
setfacl -R -m g:webdev:rwX /var/www/docs
setfacl -R -m -d:g:webdev:rwx /var/www/docs
```

#### Configuring TLS security

Package: `crypto-utils`
Apache mod: `mod_ssl`

[[LINUX/PKI/OpenSSL]] For SSL certificates and keys generation

Minimal requirements for SSL enabled sites

```
# TLS is enabled by default on port 443
Listen 192.168.137.19:443
# But explicit configuration is required on all other ports
Listen 192.168.137.19:8443 https
```

`/etc/httpd/conf.d/VIRTUALHOST.conf`
```
<VirtualHost *:443>
ServerName salex.example.com
 	DocumentRoot </DIR>

	SSLEngine on
	SSLCertificateFile /PATH_TO_CRT/<SERVERNAME>.crt
	SSLCertificateKeyFile /PATH_TO_KEY/<SERVERNAME>.key
	
	ErrorLog logs/ssl_error_log
	TransferLog logs/ssl_access_log
	LogLevel warn
</VirtualHost>
```

`SSLHonorCipherOrder on`: The server selects the cipher suite, not the client 

SELinux context type of the folder containing the cert and keys must be `cert_t`
Make sure port 443 is open

To test if SSL works with apache:

``` bash
openssl s_client -connect 'APACHE_HOST':443 -state -debug
```

To test with curl:

``` bash
curl https://'WEB_SERVER'
```

curl will return an error in this case.
To download the certificate from the web server:

``` bash
openssl s_client -showcerts -servername 'WEB_SERVER'  'WEB_SERVER':443 > cacert.pem
```

Now, using curl, include the downloaded certificate:

``` bash
curl 'https://WEB_SERVER' --cacert 'CACERT.PEM'
```

`-k`: to skip certificate verification

To redirect **http** to **https**

`/etc/http/conf/httpd.conf`
```
Listen 80
Listen 443
```

`/etc/httpd/conf.d/virtual_host.conf`
```
<VirtualHost *:80>
        ServerName salex.example.com
        Redirect / https://salex.example.com/
</VirtualHost>

<VirtualHost _default_:443>
        ServerAdmin webmaster@salex.example.com
        DocumentRoot /var/www/html/
        ServerName salex.example.com
        
        SSLEngine on
        SSLCertificateFile /certs/salex.example.com.crt
        SSLCertificateKeyFile /certs/salex.example.com.key
        
</VirtualHost>
```

Enforcing SNI. Can be configured per virtual host.

```
<VirtualHost 192.168.0.1:443> 
	ServerName does-not-exist.example.com
	# Do not serve any content to the clients that # do not support virtual
	secure hosting (via SNI). 
	SSLStrictSNIVHostCheck On
... </VirtualHost
```

#### Reserving Default Sites for Error Messages

It is not a good idea to serve web content to an incorrectly specified request

```
# We're using this default web site to explain # host mismatch and SNI issues to our users. 
<VirtualHost 192.168.0.1:443>

	# The hostname used here should never match. 
	ServerName does-not-exist.example.com 
	DocumentRoot /var/www/does-not-exist

	# Require SNI support for all sites on this IP address and port.
	SSLStrictSNIVHostCheck on

	# Force all requests to this site to fail with a 404 status code.
	RewriteEngine On
	RewriteRule ^ - [L,R=404]

	# Error message for the clients that request # a hostname that is not
	configured on this server.
	ErrorDocument 404 "<h1>No such site</h1><p>The site you requested does not ↩
	exist.</p>"
</VirtualHost>
```

#### OCSP Stapling

[[LINUX/PKI/OpenSSL Cookbook#Creating a Certificate for OCSP Signing]]

Two basic directives needed to enable OCSP stapling. Needs a valid certificate chain.

* Configure a cache of 128 KB for OCSP responses

```
SSLStaplingCache shmcb:/opt/httpd/logs/stapling_cache(128000)
```

* Enable OCSP stapling by default for all sites on this server. Can be configured per virtual host.

```
SSLUseStapling on
```

OCSP responses are cached for 3600 seconds by default

```
SSLStaplingStandardCacheTimeout <sec>
```

Apache caches both successful and failed responses for 600 seconds by default

```
SSLStaplingErrorCacheTimeout <seconds>
```

To turn off responder errors

```
SSLStaplingReturnResponderErrors off
```

Apache by default generates fake OCSP `tryLater` responses in the cases in which the real OCSP responder is unresponsive. Disabling fake responses will still allow clients to communicate with the responders directly. 

```
SSLStaplingFakeTryLater off
```

By default, OCSP responder is listed in the certificate. To override the certificate OCSP information and hardcode a custom responder

```
SSLStaplingForceURL http://ocsp.example.com
```

#### TLS Session Management

##### Standalone Session Cache

**apache mod**: `mod_socache_shmcb`

For caching using shared memory. Roughly 4000 sessions can be stored in 1 MB of cache

```
# Specify session cache type, path, and size (1 MB). 
SSLSessionCache shmcb:/path/to/logs/ssl_scache(1024000)

# Specify maximum session cache duration of one day. Defaults to 5 minutes
SSLSessionCacheTimeout 86400
```

Restarting the server clears the cache.

To configure a mutex 

```
# Configure the mutex for TLS session cache access synchronization. 
SSLMutex file:/var/run/apache2/ssl_mutex
```

> Apache uses the same TLS session cache for the entire server. Sharing it among unrelated applications is dangerous

##### Standalone Session Tickets

By default, session ticket is provided by OpenSSL

##### Distributed Session Caching

**apache mod**: `mod_socache_memcache`
**Packages**: `memcached`

Apache uses the network caching program `memcached`. Deploy it and connect the web servers to it.

```
LoadModule socache_memcache_module modules/mod_socache_memcache.so
```

Distributed TLS session caching is a mechanism to exchange session information among cluster nodes in case of not using TLS termination or sticky sessions.

```
# Use memcached for the TLS session cache. 
SSLSessionCache memcache:memcache.example.com:11211

# Specify maximum session cache duration of one hour.
SSLSessionCacheTimeout 3600
```

Memcached options

- `-m 10`: Allocate 10 MB of RAM to ensure the session data is cached for the entire duration
- `-k`: Lock the cache memory to improve performance and prevent TLS session data from being written to swap
- `-c #`: Ensure that the maximum number of connections allowed is sufficient to cover the max number of concurrent connections supported by the entire cluster
- `-d`: Run as a daemon
- `-u memcache`: Rus as user `memcache`
- `-p 11211`: Run on port 

**Availability**:
	`Memcached` is now a single point of failure for the cluster
**Performance**:
	Compare the performance of a single server against that of the entire cluster. Disable session tickets in the client to avoid measuring the wrong resumption mechanism.
**Security**
	Communication with `memcached` is not encrypted. Communicating with `memcached` can be done over a special encrypted network segment. Run separate `memcached` sections, one for each application.

##### Distributed Session Tickets

If each node in a web cluster is expected to terminate TLS, it must share the same key, thus per-server keys generated by OpenSSL cannot be used. 

Apache 2.4.x supports manually configured session ticket keys

Generate a ticket key. 

```bash
openssl rand -out ticket.key 48

```

```
SSLSessionTicketKeyFile /path/to/ticket.key
```

Session keys must be rotated regularly - for example, once a day.

##### Disabling Session Tickets

Apache 2.4.11+

```
SSLSessionTickets off
```

#### Client Authentication

* Enable it
* Provide all the necessary CA certificates to form a full chain
* Provide revocation information

```
# Require client authentication. 
SSLVerifyClient require

# Specify the maximum depth of the certification path,
# from the client certificate to a trusted root.

SSLVerifyDepth 2

# Allowed CAs that issue client certificates. The # distinguished names of these certificates will be sent # to each user to assist with client certificate selection. 

SSLCACertificateFile conf/trusted-certificates.pem
```

Other options for `SSLVerifyClient`
`optional`: Requests a client certificate but does not require it. The status of the validation is stored in the SSL_CLIENT_VERIFY variable: NONE, SUCCESS, FAILED. Useful if you want to provide a custom response for failed client validations/
`optional_no_ca`: External service is expected to validate the certificate.


Checking client certificates for revocation is done through a local CRL list. A script periodically retrieves fresh CRLs and reloads the web server

```
# Enable client certificate revocation checking. 
SSLCARevocationCheck chain

# The list of revoked certificates. A reload is required # every time this list is changed.

SSLCARevocationFile conf/revoked-certificates.crl

# 2.4.x Use OCSP to check client certificates for revocation. (slower)
SSLOCSPEnable On
```

Disable compression

```
SSLCompression off
```

#### HTTP Strict Transport Security

```
# Enable HTTP Strict Transport Security.
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
```

HSTS policy can be set only on HTTP responses delivered over an encrypted channel. Also, make ensure that all site visitors reach HTTPS as soon as possible:

```
VirtualHost *:80> 
	ServerName www.example.com 
	ServerAlias example.com ...

	# Redirect all visitors to the encrypted portion of the site. 
	RedirectPermanent / https://www.example.com/
</VirtualHost>

VirtualHost *:443> 
	ServerName www.example.com 
	ServerAlias example.com ...
	SSL_Directives...
```

To test HSTS

```bash
curl -I https://gitlab.ohio.cc
```

```
HTTP/1.1 200 OK
Date: Fri, 31 Jan 2025 23:38:14 GMT
Server: Apache/2.4.62 (CentOS Stream) OpenSSL/3.2.2
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Test against http

```bash
curl -I http://gitlab.ohio.cc
HTTP/1.1 301 Moved Permanently
```
#### LDAP Integration

[Apache LDAP Documentation]([mod_authnz_ldap - Apache HTTP Server Version 2.4](https://httpd.apache.org/docs/2.4/mod/mod_authnz_ldap.html#exposed))

Packages
`mod_ldap`

SELinux:

```
type=AVC msg=audit(1728856095.509:325): avc:  denied  { name_connect } for  pid=2953 comm="httpd" dest=636 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:ldap_port_t:s0 tclass=tcp_socket permissive=0
```

``` bash
setsebool httpd_can_connect_ldap 1
```

To allow access to the web resource to authenticated users only using LDAP. Users trying to access a specific location will be asked to authenticate. 

```
<VirtualHost *:443>
	virtual host stuff
 <Location /RESOURCE >
         AuthType Basic
         AuthName "LDAP example.com"
         AuthBasicProvider ldap
         AuthLDAPURL ldaps://LDAP_SERVER/BASE_DN?uid?sub
         Require valid-user
 </Location>
</VirtualHost>
```

Other directives
- `Require ldap-group <AUTHORIZED GROUP>`: Give access to authorized group users only.
- `AuthLDAPBindAuthoritative on`
- `AuthLDAPBindDN cn=webadmin,ou=meta,dc=example,dc=com`
- `AuthLDAPBindPassword <thewebadminpasswordincleartext>`

#### Connecting to Databases

`SELinux` considerations

- `httpd_can_network_connect_db` (Boolean) : Enabled when the db server is on a remote machine
- `httpd_can_network_connect` (Boolean): Enabled when it is not obvious that the target is a db server 

#### Deploying CGI Applications

**Common Gateway Interface**: Method of serving dynamic content. Apache executes an app and returns the output of it.

Requirements:

* The **ScriptAlias /cgi/bin/** "*DIR*" must exist
* The **ScriptAlias** directory, where script are stored, must be of **httpd_sys_script_exec_t** context type

##### Serving PHP Content

* **PHP** scripts can be included using **CGI**.
* **mod_php** apache module that includes an internal PHP interpreter. 

**mod_php** adds a **/etc/httpd/conf.d/php.conf** file that allows .php extensions to be executed by the **httpd** server

##### Adding Dynamic Python Content

* Using **CGI**
* **mod_wsgi** module. **WSGIScriptAlias** is added to the virtual host file.

Sample:
**WSGIScriptAlias** /webapp/ /opt/webapp/app.py

The **WSGI** app must be executable by the apache user and group. Context type set to **httpd_sys_content_t**


#### Configuring Private Directories

`/etc/httpd/htpasswd`: apache users are stored in this file

Create apache user for the first time (if the httpasswd file does not exist)

``` bash
htpasswd -c /etc/httpd/htpasswd "USERNAME"
```

Add more apache users:

``` bash
htpasswd /etc/httpd/htpasswd "USERNAME"
```

`-c` :option overrides the `htpasswd` file!!!** Same as echo "stuff" > file

```
[root@salex httpd]# cat htpasswd
babuna:$apr1$QXCsof4m$zeq6m2zWq65xqBvdRHjwg0
gligana:$apr1$NPsGkFsm$kakl7rHzIdJN2LpeO0wLI/
```

In the virtual host config add the following block:

```
<Directory /var/www/html/secret>
        AuthType Basic
        AuthName "secret files"
        AuthUserFile /etc/httpd/htpasswd
        Require user <USER>
</Directory>
```

To configure group access:

Create a file `/etc/httpd/htgroup`
Add the following in the `htgroup` file

```
<GROUP_NAME>: <USER1> <USER2> <USERN> (from the htpasswd file)
```

Add the following to the virtual host conf file:

```
AuthGroupFile /etc/httpd/htgroup
Require group <GROUP_NAME> (as per the htgroup file)
```

Commonly used options for access restriction:

* **AuthType**: Type of authentication. Set to Basic in most cases
* AuthName: Defines a name for the directory. Optional
* AuthUserFile: Specify the htpasswd file where web users are stored
* AuthGroupFile: Specifies the htgroup file where web user groups are stored
* `Require`: specifies which users or groups have access:
	* Require valid-user: all users from the htpasswd file
	* Require user *USER*: specific user from the htpasswd file
	* Require group *GROUP*: specific group from the htgroup file
	* Require ip *IP_ADDRESS*
	* Require expr "EXPRESSION": access granted if expression is true
	* Require method "METHOD": access granted to certain HTTP methods
	* Require env "ENV_VAR": access granted if env variable set
	* Require all granted: allow access unconditionally
	* Require all denied: deny access unconditionally

#### PHP Support

[[LINUX/SOFTWARE MANAGEMENT/Fedora Package Management]]

PHP is not threadsafe. FastCGI process manager can be used to link PHP to Apache

MariaDB support in PHP

CentOS packages
`php-fpm php-mysql php-gd php-imap php-mbstring`

`php-fpm`: A daemon that translates requests from the HTTP server and responds with the response from PHP code. 
`php-fmp.conf`: config file
PHP-FPM can separate apps into pools. Pools are created in `/etc/php-fpm.d/`

```
mv /etc/httpd/conf.d/php.conf /etc/httpd/conf.d/php.conf.bk
```

Make sure the following modules are loaded

``` bash
grep -E '(proxy.so|fcgi)' /etc/httpd/conf.modules.d/00-proxy.conf
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
```

Add the following

``` bash
echo "listen = 127.0.0.1:9000" >> /etc/php-fpm.d/www.conf
echo "listen= /run/php-fcgi.sock" >> /etc/php-fpm.d/www.conf
```

Restart the `php-fpm.service`

A socket is now created

```
[root@server9 php-fpm.d]# stat /run/php-fcgi.sock
  File: /run/php-fcgi.sock
  Size: 0               Blocks: 0          IO Block: 4096   socket
```

The `ProxyPassMatch` directive must be added to match on any .php files and pass them on to PHP-FPM module

