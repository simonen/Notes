---
tags:
  - centos
  - apache
  - databases
---
Packages
httpd
mariadb-server
php, php-mysqlnd, php-fpm
wordpress

#### Apache 

Install Apache

``` bash
dnf install -y httpd
```

Create the virtual host

> [!NOTE]+ /etc/http/conf.d/example.com.conf
> ```
> <VirtualHost *:80>
>         ServerAdmin webmaster@account.example.com
>         DocumentRoot /var/www/html/example.com
>         ServerName example.com
>         ServerAlias www.example.com
>         ErrorLog logs/example.com-error_log
>         CustomLog logs/example.com-access_log common
> </VirtualHost>
> ```

Create the directory

``` bash
mkdir -p /var/www/html/example.com
```

Download and unpack wordpress from https://wordpress.org/download/releases/

``` bash
tar -zx --strip-components=1 -C /var/www/html/example.com -f wp.tar.gz
```

Allow http through the firewall

```
firewall-cmd --add-service=http --permanent ; firewall-cmd --reload
```

Start the Apache server

``` bash
systemctl enable --now httpd
```
#### MariaDB

Install MariaDB server

```
dnf install -y mariadb-server
```

Run the secure installation script

`mysql_secure_installation`

Create the database

``` bash
mysql -u root -e "CREATE DATABASE wpdb;" -p
```

Create the wpdb admin user

``` bash
mysql -u root -e "CREATE USER `wpadmin`@`DBHOST` identified by 'PASSWORD';" -p
```

Grant the user privileges on the wpdb

``` bash
mysql -u root -e "GRANT CREATE, SELECT, INSERT, UPDATE, DELETE ON wpdb.*
TO `wpadmin`@`DBHOST` IDENTIFIED BY 'PASSWORD';" -p
```

Allow the mariadb server through the firewall if on a remote host

``` bash
firewall-cmd --add-service=mysql --permanent ; firewall-cmd --reload
```

Enable the SELinux booleans If on a remote host


Start the mysql server

``` bash
systemctl enable --now mariadb
```
#### PHP

Install epel-release repo

```
dnf install -y epel-release
```

Install php

```
dnf install -y php php-fpm php-mysqlnd
```

Restart the Apache server
#### WordPress Installation

Add the www.example.com record in the hosts file or in the DNS server

Go to http://www.example.com/wp-config/index.php and follow instructions
