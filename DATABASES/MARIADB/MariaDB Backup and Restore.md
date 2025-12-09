[[gitea/LINUX 1/DATABASES/MARIADB/MariaDB]]
#### Physical Backup

* Raw copies of the database directories and folders. 
* Portable to similar hardware and software
* Database service should be offline or database tables should be locked
* Good practice to have the datadir `/var/lib/mysql` mounted on a separate LVM volume

To show where the actual database is stored:

```bash
mysqladmin -u root -p variables | grep datadir
```

```
| datadir                                                | /var/lib/mysql/
```

Create the backup
1. MariaDB > `FLUSH TABLES WITH READ LOCK`;
2. `systemctl stop mariadb`
3. `lvcreate -s -n LVDBNAME -L SIZE /dev/VG/LV`
4. Start the `mariadb` service
5. MariaDB > `UNLOCK TABLES`;
6. Mount the snapshot volume in a directory
7. Make a tar archive of the mounted `/mysql` dir
8. Unmount, remove the snapshot volume
#### Logical Backup

* Database structure is retrieved by querying the database
* Relatively slow
* Can be created on an operational database
* Portable to other database providers
* Log and configuration files are NOT included

To make a logical backup of a database:

``` bash
mysqldump -u root -p 'DATABASE' --databases > 'FILE.dump'
```

To make a logical backup of all databases:

``` bash
mysqldump -u root -p --all-databases > 'FILE.dump'
```

To restore a database, create a db with the same name and execute:

``` bash
mysql -u root -p 'ORIGINAL_DATABASE_NAME' < DB.dump
```
