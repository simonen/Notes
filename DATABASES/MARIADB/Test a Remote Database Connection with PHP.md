---
tags:
  - databases
  - php
  - mariadb
---
[[gitea/LINUX 1/DATABASES/MARIADB/MariaDB]]
[Basic MariaDB Administration](http://127.0.0.1:8088/books/linux-administration/page/basic-mariadb-administration)

1. Install Apache web server
2. Install php
3. Install mariadb on a separate server
4. Create a database or use the test db
5. Create the db_connection.php file
6. Create the index.php file
7. Configure the firewall and SELinux
8. Configure the hosts file / DNS

###### Install the Apache server

```sh
yum install -y httpd
```

###### Install the repositories needed for **PHP** (CentOS 7)

```sh
yum install epel-release yum utils
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum-config-manager enable remi-php81
yum update
```

###### Install **php** and extentions

```sh
yum install -y php
yum install -y php-fpm php-curl php-cli php-json php-mysql php-opcache php-dom php-exif php-fileinfo php-zip php-mbstring php-hash php-imagick php-openssl php-pcre php-xml php-bcmath php-filter php-pear php-gd php-mcrypt php-intl php-iconv php-zlib php-xmlreader
```

###### Install **mariadb** and mariadb-server

```sh
yum install -y mariadb mariadb-server
systemctl enable --now mariadb
mysql_secure_installation
```

###### Create the **db** user, accessible only from the web server, or the db consumer

```sql
Mariadb > CREATE USER USER@WEBSERVER IDENTIFIED BY 'PASSWORD';
Mariadb > GRANT ALL PRIVILEGES ON DATABASE*.* TO USER@WEBSERVER;
```

##### Create the `db_connection.php` file in `/var/www/hmtl` for convenience

`/var/www/hmtl/db_connection.php`
```php
> <?php
function OpenCon()
{
$dbhost = "DB_SERVER";
$dbuser = "DB_USER";
$dbpass = "DB_USER_PASS";
$dbname = "DB_NAME";
$conn = new mysqli($dbhost, $dbuser, $dbpass,$dbname) or die("Connect failed: %s\n". $conn -> error);
return $conn;
}
function CloseCon($conn)
{
$conn -> close();
}
?>
```

###### Create the `index.php` file

`/var/www/hmtl/index.php`
```php
<?php
include 'db_connection.php';
$conn = OpenCon();
echo "Connected Successfully";
CloseCon($conn);
?>
```

##### Allow apache and mariadb through their firewalls

```bash
firewall-cmd --add-service=http --permanent ; firewall-cmd --reload
firewall-cmd --add-service=mysql --permanent ; firewall-cmd --reload
```

##### Configure SELinux to allow remote db connections

```sh
setsebool httpd_can_network_connect_db 1
```

Test the connection.
Use PHP to make sure it can properly connect to the db. Curl does not test for db conn.

```
[root@vpnserver html]# php /var/www/index.php
Connected Successfully
```
