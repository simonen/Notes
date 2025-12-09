
1. Which command provides the easiest solution to apply basic security set-
	tings to MariaDB after installing it?
	**c. Run the mysql_secure_installation command after installation**
2. Which of the following parameters in my.cnf should you modify to enable full
	network access to MariaDB databases?
	a. Set the bind address to bind=::
3. Which command should you use to log in to the database using administrative
	permissions?
	c. mariadb -u root -p
4. You want to add a user account to the current database. To do so, you need to
	find out the current attributes for users, which are created in a table with the
	name users. Which command can you use to show these attributes?
	a. describe users;
5. Which of the following shows correct syntax to add a user into the users table?
	c) INSERT INTO user (Host,User,Password) VALUES (‘localhost’,
	’linda’,’password’);
6. Which command enables you to find all users who have the last name John-
	son, assuming that the last name is stored in the name field?
	a. select * from user where name = johnson;
7. Which command shows correct syntax for adding a user lisa with the password
	password on the current database?
	d. create user lisa@localhost identified by ‘password’
8. Which of the following is not true about physical database backups?
	b. The physical backup contains the database structure that is retrieved by
	using a query on the database
9. Which command shows correct syntax for making a logical database backup of
	the videos database?
	d**. mysqldump -u root -p videos --databases > /root/videos-db.dump**
10. Which command shows how to create a snapshot of the LVM logical volume
	/dev/vgdata/lvmariadb, which is needed to create a physical backup of your
	databases?
	a. lvcreate -s -n lvmariadb-snap -L 2G /dev/vgdata/lvmariadb
