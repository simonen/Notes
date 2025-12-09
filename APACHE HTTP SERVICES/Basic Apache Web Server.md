---
tags:
  - apache
  - centos
---

Apress – Pro Linux System Administration 2nd 2017

Packages:
- `httpd` - CentOS
- `apache2` - Debian
- `mod_ssl` - SSL module for Apache

Ports: 80, 443
#### Getting Started

Install Apache on RHEL

``` bash
sudo group install 'Basic Web Server'
```

##### Files and Directories

- `/etc/httpd` - ServerRoot
- `/etc/httpd/conf/httpd.conf` - Main config file
- `/etc/httpd/conf.d` - Drop-in conf files, virtual hosts
- `/etc/http/conf.modules.d` - Module specific conf files
- `/var/www/html` - Default document root
- `/usr/share/doc/httpd-2.4.6/` - Sample config and virtual hosts files

- `DocumentRoot` - specifies the location where Apache looks for content.
- `ServerRoot` - specifies the location of Apache conf files

```
[kimchen@rhel9 ~]$ ll /etc/httpd
total 4
drwxr-xr-x. 2 root root   59 Dec 24 20:27 conf
drwxr-xr-x. 2 root root  135 Jan  7 01:18 conf.d
drwxr-xr-x. 2 root root 4096 Jan  7 01:18 conf.modules.d
lrwxrwxrwx. 1 root root   19 Jul 20 11:48 logs -> ../../var/log/httpd
lrwxrwxrwx. 1 root root   29 Jul 20 11:48 modules -> ../../usr/lib64/httpd/modules
lrwxrwxrwx. 1 root root   10 Jul 20 11:48 run -> /run/httpd
lrwxrwxrwx. 1 root root   19 Jul 20 11:48 state -> ../../var/lib/httpd

```

The symbolic links are created to allow Apache to be run in a chroot environment.
Processes that are running in chroot env can read files only from that environment
#### Apache Virtual Hosts

**virtual host**: a distinct Apache .conf file or section that is created for a unique hostname (site)

Procedure to access a virtual host:

* Client starts a session to a host, typically by a web browser
* DNS name resolve
* The Apache process receives request for all virtual hosts it is running
* The Apache process reads the HTTP header and decides which vh to forward to
* Apache reads the specific VH conf file to find which **DocumentRoot** it is using
* The request is forwarded to the appropriate contents file in that specific document root

* **name-based virtual hosting**: virtual hosting causes Apache to serve a web page for a specific directory based on the name of the site a remote user connected to. Any number of name-based virtual hosts can share a single IP address. The name of the site is determined by a special header that is sent in the request to the web server.

* **ip-based virtual hosting**: IP-based virtual hosting causes Apache to serve a web page from a specific directory, based on the IP address the request was received on. For each IP-based virtual host, the Linux host needs to have an IP address assigned to a network interface.  Apache uses TLS for encryption

#### Creating Virtual Hosts

1. Create DNS records for the Virtual Hosts in `/etc/hosts`
2. Create a file in `/etc/http/conf.d/'HOSTNAME'.conf`
3. Make sure the `ServerName` is resolvable. 

`/etc/http/conf.d/_HOSTNAME_.conf`
```
<VirtualHost *:80>
	ServerAdmin webmaster@account.example.com
	DocumentRoot /www/docs/account.example.com
	ServerName account.example.com
	ServerAlias www.account.example.com
	ErrorLog logs/account.example.com-error_log
	CustomLog logs/account.example.com-access_log common
</VirtualHost>
```

`DirectoryIndex index.html` - This tells apache to look for a file named index.html as the webpage home page

`curl www.account.example.com`
`curl –H 'Host: www.example.com' http://localhost`

-H 'Host: www.example.com' : sends the special header
##### Host Based Access

To control access separately to virtual host the `\<Directory>` block must be inserted in the `VirtualHost` block

`/etc/httpd/conf.d/virtualhost.conf`
```
<VirtualHost *:PORT>
	<Directory "/web">
		<DIRECTIVES>
	</Directory>
</VirtualHost>
```

Directives:
* `Allow from` \[all | HOST | IP]
* `Deny from` \[all | HOST | IP]
* `Order` \[allow,deny | deny,allow]

Order
- **Allow,Deny**: Apache first applies the `Allow` directives and then the `Deny` directives. It means that if an IP address matches any `Allow` directive, it's allowed access unless it also matches a `Deny` directive. If there's no match with any `Allow` directive but matches a `Deny` directive, access is denied.

- **Deny,Allow**: Apache first applies the `Deny` directives and then the `Allow` directives. It means that if an IP address matches any `Deny` directive, access is denied unless it also matches an `Allow` directive. If there's no match with any `Deny` directive but matches an `Allow` directive, access is allowed.

To deny a specific host access to a virtual host

```
Order deny,allow
Deny from <HOST1> <HOSTN> 

or

Order allow,deny
Allow from all
Deny from <HOST1> <HOSTN>
```

To allow only specific hosts access to a virtual host

```
Order allow,deny
Allow from <HOST1> <HOSTN>

or

Order deny,allow
Allow from <HOST1> <HOSTN>
Deny from all
```

##### Options

`AllowOverride None | All` 
* None
* All

Options
* All
* FollowSymLinks
* Indexes
* MultiViews
* IncludesNOEXEC
* Includes
* SymLinksIfOwnerMatch
* ExecCGI

##### User-Based Access

[[gitea/LINUX 1/APACHE HTTP SERVICES/Advanced Apache Web Server#Configuring Private Directories]]

User: 
`apache`
##### Firewall

Ports: 80, 443
##### SElinux 

Booleans
Contexts
#### Apache Modules

Modules add extra functionality. Enabled by the `LoadModule` directive

CentOS
`/etc/httpd/conf.modules.d/*.conf`

Ubuntu

 http://httpd.apache.org/docs/2.4/mod/.
 
`/etc/apache2/mods-available`

To enable modules in Ubuntu, symlinks are created in 
`/etc/apache2/modules-enabled`

Modules can be managed manually by the 
`a2enmod`, `a2dismod` commands

##### Apache MPMs

Multi Processing Modules

* `prefork` (default) - connections are handled by a separate process. Suitable for nonthreadsafe web applications. A control process creates a child process that listens for connections
* `worker` - mixture of process-based and thread-based processing. Suitable for threadsafe apps. The parent process creates a child process that launches several threads; one thread is assigned to each incoming connection. Incoming connections are passed off to waiting server threads
* `event` - newer, based on the worker module. Optimized for handling `keepalives`, which handle active connections to waiting threads.

The default MPM module can be changed by commenting or uncommenting the appropriate modile in `/etc/httpd/conf.modules.d/00-mpm.conf `

#### Logging

`LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\""
vhost_combined`

- `%v`: virtual host
- `%p`: port
- `%h`: remote hostname
- `%l`: remote username
- `%u`: authenticated user
