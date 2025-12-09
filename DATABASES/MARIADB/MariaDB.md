---
tags:
  - databases
  - centos
  - mysql
---
MariaDB is a simple SQL shell client

##### Packages: 
`mariadb`
`mariadb-server`

Install the official MariaDB repo to get the latest version
https://mariadb.org/download/?t=repo-config&d=CentOS+7&v=11.3&r_m=chroot-network

##### Documentation
https://mariadb.com/kb/en/mariadb-command-line-client/

##### Files and Directories
- `/etc/my.conf`: MariaDB main conf file
- `/etc/mysql/mysql.conf.d`: Main conf files if mySQL 
- `/var/lib/mysql/`:  Data dir. Default database location
* `ibdata1`: contains system and user data
* `ib_logfileN`: redo, transaction logs. Data is kept here before written to table files
* `/<database>/<tablename.frm>`
- `/var/log/mysql/error.log`: Error log (mySQL)

##### Install MariaDB

```sh
yum install -y mariadb mariadb-server mariadb-test
systemctl enable --now mariadb
# To set up secure mariadb
mysql_secure_installation
```

To get a list of available settings

```sh
/usr/sbin/mysqld --help --verbose 
```

#### Remote Connections to Databases

`/etc/my.cnf`
```
[mysqld]
# Allows connections on all IPv4 and IPv6 addresses. Blank for IPv4 only
bind-address=:: 
# Allows remote connections; 1 denies remote connections. If disabled, local communication with the db is done via sockets, not localhost. 
skip-networking=0 
port PORT
```

To show the current location of databases:

```sh
mysqladmin -u root -p variables | grep datadir
```

```
| datadir                                                  | /var/lib/mysql/
```

##### SELinux 

On the web server

```sh
setsebool httpd_can_network_connect_db 1 
```

##### Firewalls

```sh
# CentOS
firewall-cmd --add-service=mysql --permanent ; firewall-cmd --reload
```

```sh
# Debian
ufw allow in on** INTERFACE  from SOURCE_NETWORK/MASK to any port 3306
```

#### MariaDB Storage Engines

| Archive   | Data Archiving                                                                |
| --------- | ----------------------------------------------------------------------------- |
| Aria      | An enhanced MyISAM database                                                   |
| Cassandra | NoSQL storage engine to access data in a Cassandra cluster                    |
| Connect   | Allows access to text files as if they were database tables                   |
| ScaleDB   | Commercial large-scale high availability (HA)/durable database storage engine |
| Spider    | Allows access distributed databases via sharded share-nothing architecture    |
| TokuDB    | High-performance write-heavy database                                         |
| XtraDB    | A fork and drop-in replacement of MySQL InnoDB                                |
To list installed engines

``` bash
mysql -u root -e "show engines;"
```

XtraDB / InnoDB is the Default engine

To list all InnoDB-related variables

``` bash
mysql -u root -e "show variables like '%innodb%';" -p
```

To list individual variables

``` bash
mysql -u root -e "show variables like '%innodb_fast_shutdown';" -p
```

```
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| innodb_fast_shutdown | 1     |
+----------------------+-------+
```

When there is change in tables, InnoDB stores data in redo logs. When the logs fill up, the MariaDB server flushes the data into table files. Performing these operations at once is more efficient. The logs can also be used to replay transactions in case of a system crash. Similar to journaling. 

One simple performance tuning is to modify the redo file size. For older versions of MariaDB, changing the transaction log size must be done after transaction log files have been flushed of live data.

This can be done by forcing the server to process all entries in the transaction logs and write them to the table files when the server shuts down. This is controlled by the `innodb_fast_shutdown` variable

To force a transaction log flush at shutdown

```
mysql -u root -e "SET GLOBAL innodb_fast_shutdown = 1;" -p
```

Shutting down the server will flush all pending changing from the transaction logs to the table files

Some performance variables

``` 
[mysqld]
innodb_log_file_size = 48M # transaction log file size
innodb_log_buffer_size = 16M # in memory log buffer
innodb_log_files_in_group = 2 # number of log files
innodb_buffer_pool_size = 128M # amount of RAM MariaDB to use. Up to 80%
innodb_flush_method = O_DIRECT
```

innodb_flush_method : handles how data is flushed to disk
* **fsync**: This is the default method. It uses the `fsync` system call to flush data to disk. It is a safe and reliable method but might not be the fastest
* **O_DSYNC**: This method uses the `O_DSYNC` flag to open files. It ensures that data is written to disk directly, bypassing some OS-level buffering. It can be faster than `fsync` but is typically used on systems where `O_DIRECT` is not supported.
* **O_DIRECT**: This method uses the `O_DIRECT` flag to open files. It minimizes the caching done by the operating system, reducing the double buffering effect and potentially improving performance. This method is recommended when InnoDB's internal caching is deemed sufficient and the operating system cache is unnecessary.
* **O_DIRECT_NO_FSYNC**: This is a variant of `O_DIRECT` where the `fsync` system call is not used. It can be faster in some scenarios but might risk data integrity if the system crashes before data is fully written to disk.
* **littlesync**: This method is similar to `fsync` but might offer better performance on some systems due to minor differences in the way it handles synchronization.
* **nosync**: This method disables synchronization completely. It is very fast but also very risky because it doesn't guarantee data persistence in the event of a crash.

```
read_buffer_size = 1M # data read chunk size
read_rnd_buffer_size = 1M
max_allowed_packet = 16M
# binary log. Helps recovery in case of a crash
log_bin = /var/log/mariadb/mariadb-bin.log
expire_logs_days = 14
max_binlog_size = 128M
```

##### Binary logs

Binary logs record all changes made to the MySQL database. They include data modification statements (such as `INSERT`, `UPDATE`, `DELETE`) and certain system-related commands (like `CREATE` and `ALTER`). They do not log `SELECT` statements as they don't alter data.

###### Uses of Binary Logs

1. **Replication**: Binary logs are used to replicate changes from a master server to slave servers. The slave servers read the binary logs to execute the same operations, ensuring they remain synchronized with the master.
2. **Point-in-Time Recovery**: In case of a crash or data corruption, binary logs allow for point-in-time recovery, meaning you can restore the database to a specific moment before the incident occurred.
###### Configuration

```
[mysqld]
log-bin=mysql-bin
server-id=1
``` 

`server-id`:  unique identifier for the server in a replication setup

To print a list of binary logs

```
mysql -u root -e "show binary logs;" -p
```

To view binary log contents

```
mysqlbinlog mysql-bin.000001
```

###### Purging Binary Logs

Automatic
```
[mysqld]
expire_logs_days=7
```

Manual
```
MariaDB> PURGE BINARY LOGS TO 'mysql-bin.000010';
```

Purge logs older than a specific date
```
PURGE BINARY LOGS BEFORE '2024-01-01 00:00:00';
```

###### Binary Log Formats

1. **Statement-Based Logging (SBR)**: Logs the actual SQL statements executed.
2. **Row-Based Logging (RBR)**: Logs the actual changes made to each row.
3. **Mixed Logging**: Uses statement-based logging by default but switches to row-based logging when necessary.

```
[mysqld]
binlog_format=MIXED
```

Binary logs are crucial for replication and data recovery in MySQL. Properly configuring and managing them ensures you can replicate databases efficiently and recover from data loss events accurately. Make sure to monitor the size and growth of your binary logs to maintain optimal performance and storage management.

#### Simple Database Administration Tasks

**mysql** commands are not case sensitive.
Database names and table names are case sensitive.

**Database elements:**

* **Database**: overall thing
* **Tables**: Classes of items that are used in a db.
* **Records**: The specific dataset stored in a table
* **Fields**: information placeholder inside a record
* **Values**: Specific value stored in a field

##### **mySQL** basic commands:

To log in to **mariadb** cli:

```sh
mysql -u root -p
```

To display available databases:

```sql
MariaDB > show databases;
```

**CRUD**: CREATE, READ, UPDATE, DELETE

```sql
MariaDB > CREATE DATABASE `DBNAME`;
MariaDB > CREATE TABLE <TNAME>(RECORD1 VARCHAR(X), RECORDN VARCHAR(X));
MariaDB > USE DATABASE | TABLE <DBNAME | TABLE>;
MariaDB > SHOW TABLES;
MariaDB > INSERT INTO <TNAME>(ATTR1,ATTR2...,ATTRN) VALUES('VALUE1','VALUEN');
MariaDB > SELECT * FROM <TNAME>;
MariaDB > SELECT * FROM <TNAME> WHERE <ATTR>='VALUE' /* filters all records by the given value */*
MariaDB > describe <TNAME>;
MariaDB > DELETE FROM <TNAME> WHERE <ATTR>='VALUE'
MariaDB > UPDATE <TNAME> SET <ATTR>='NEW_VALUE' WHERE <ATTR>='VALUE'
MariaDB > DROP DATABASE IF EXISTS 'MY_DATABASE';
```

If a db name contains special chars like a hyphen `my-db`, it should be enclosed in backquotes or ticks, as the server might interpret the name as a subtraction in this case.

Commands can be executed without entering the `mariadb` CLI:

```sh
mysql -u USER -e '<MYSQL COMMAND>;'
```
##### Managing Users

MariaDB users are stored in the **users** table. 
To create a new user, the **CREATE USER** or **INSERT USER** privilege is needed

```sql
CREATE USER USER@HOST IDENTIFIED BY 'PASSWORD'
```

A **mysql** user is defined by *USER*@*HOST*, depending on the log in type: local or remote.
- `user@'localhost'`: for local login only
* `user@'remotehost'`: for remote users
* `user@'%'`: Wildcard. Any remote host in this case.

List all **mysql** users:

```sql
SELECT User, Host FROM mysql.user;
```

To delete a user:

```sql
DROP USER <USER>@<HOST>;
```

Active users are not dropped immediately. New users have no privileges.

Privilege management

```sql
GRANT SELECT,UPDATE,INSERT,DELETE ON <TABLE.ATTRIBUTE> TO <USER@HOST>;
GRANT SELECT ON <DATABASE.TABLE> TO <USER@HOST>;
GRANT SELECT ON <DATABASE>.* TO <USER@HOST>;
GRANT SELECT ON *.* TO <USER@HOST>; /* gives SELECT privileges on all
# dbases and their tables to a user */*
GRANT { CREATE | ALTER | DROP } ON <DATABASE>.* TO <USER@HOST>;
GRANT ALL PRIVILEGES ON *.* TO <USER@HOST> identified by "PASS"; # super user
SHOW GRANTS FOR <USER@HOST>;
-- To reload privileges after changing them:
FLUSH PRIVILEGES;
```

##### Grant Privileges

- `SELECT` Gives the ability to perform SELECT statements
- `INSERT` Gives the ability to perform INSERT statements
- `UPDATE` Gives the ability to perform UPDATE statements
- `DELETE` Gives the ability to perform DELETE statements
- `INDEX` Gives the ability to create indexes on tables
- `CREATE` Gives the ability to create databases tables
- `ALTER` Gives the ability to alter database tables
- `DROP` Gives the ability to drop database tables
- `GRANT` OPTION Gives the ability to grant the same privileges to other users
- `ALL` Gives all privileges except GRANT OPTION

The native password authentication method. Just type mysql and press Enter.

```sql
MariaDB [(none)]> SET PASSWORD = PASSWORD('your-password');
MariaDB [(none)]> update mysql.user set plugin = 'mysql_native_password'
where User='root';
MariaDB [(none)]> FLUSH PRIVILEGES;
```
