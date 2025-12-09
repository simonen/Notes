
1. After installing the mariadb packages, which command do you need to run to
	set up basic security settings
	**mysql_secure_installation**
2. How do you configure MariaDB to be accessible through the network?
	in my.cnf bind-address=::, skip-networking=0
3. Which command enables you to get information about the settings that can be
	used in my.cnf?
	**/usr/libexec/mysqld --help --verbose**
4. After logging in to the MariaDB shell environment as root, which command
	enables you to get an overview of databases that are available?
	SHOW DATABASES;
5. Which command enables you to make addressbook your active database;
	**use addressbook**;
6. To find out which tables are available in the addressbook database, which
	command would you use?
	SHOW TABLES;
7. To find out which fields are available in the addressbook table, which com-
	mand would you use?
	describe addressbook;
8. To find out which records are available in the addressbook table, which com-
	mand would you use?
	select \* from addressbook;
9. . What command enables you to create a snapshot?
	lvcreate -s 
10. How can you temporarily prevent the database from writing modifications to
	the database files?
	FLUSH TABLES WITH READ LOCK;