---
tags:
  - centos
  - databases
  - postgresql
---

Package: 
`postgresql`: client
`postgresql-server`: server

Port: 
`5432/tcp`

Main configuration file
- `/var/lib/pgsql/data/postgresql.conf`

#### Install and Start up PostgreSQL

``` bash
dnf install -y postgresql postgresql-server
```

Initialize the database cluster

``` bash
postgresql-setup initdb
```

Start the service

``` bash
systemctl enable --now postgresql
```

#### Configure Authentication and Connections

By default, PostgreSQL is configured to be accessed on the local unix socket by the `postgres` user only. Configure database access and authentication methods: 

Configure PostgreSQL to listen on all ipv4 interfaces. 

`/var/lib/pgsql/data/postgresql.conf`
```
listen_addresses = '0.0.0.0'
```

`listen_addresses = "*"`:  ipv4 and ipv6. 

`/var/lib/pgsql/data/pg_hba.conf`
``` bash
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
```

Methods:
- `peer`: The authenticating Unix user is mapped directly to the corresponding PostgreSQL user. No password prompt.
- `md5`: The authenticating user will be prompted for password.
- `gss`: Kerberos
- `cert`: Very high level of sec
- `ldap`

Set password for the `postgres` user. Change the prompt first

``` bash
sudo -i -u postgres
```

Enter PostgreSQL

``` bash
-bash-4.2$ psql
```

Set password for the `postgres` user. Password in `''` quotes.

``` postgresql
postgres=# ALTER USER postgres PASSWORD 'your_pass';
postgres=# \q # exit the postgres prompt
```

If method is set to `md5` in `pg_hba.conf`, authenticate as the `postgres` user with its password

```bash
psql -U postgres -W
```

To allow remote access

`/var/lib/pgsql/data/postgresql.conf`
``` bash
# - Connection Settings -

listen_addresses = '*' #  comma-separated list of IP addresses
```

With pass-based authentication

`/var/lib/pgsql/data/pg_hba.conf`
``` yaml
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             NETWORK/MASK            md5
```

Connect to a remote PostgreSQL server

```
psql -h SERVER -U username -d database
```
#### Databases

Create a database

``` postgresql
CREATE DATABASE 'dbname';
```

Connect to the db

``` postgresql
/c
```

Create a table

``` postgresql
CREATE TABLE table_name1 (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```

Verify the table

``` postgresql
\d table_name
```

Insert items into the table

``` postgresql
INSERT INTO table_name1 (table_column) VALUES ('Alice');
```

Do a query to verify

``` postgresql
SELECT * FROM table_name1;
```

Drop a database

``` postgresql
DROP DATABASE 'dbname'
```

#### Users

List PostgreSQL users

```
\du
```

Create a user

``` postgresql
CREATE USER postgres_user WITH PASSWORD 'your_secure_password';
```

Change user password

``` postgresql
ALTER USER postgres PASSWORD 'your_pass';
```

**Grant Database Access**: This allows the user to connect to the `ohio` database.

``` postgresql
GRANT CONNECT ON DATABASE ohio TO ohio_user;
```

Authenticate to PostgreSQL with a new user

``` bash
psql -U `USER` -d 'database' -W
```

Execute PostgreSQL commands from the Linux prompt

``` bash
psql -U ohio_user -d ohio -c "SELECT * FROM users;"
```

Commands can be executed from a `.sql` file

`commands.sql`
```
INSERT INTO users (name) VALUES ('Bob');
SELECT * FROM users;
```

Execute

``` bash
psql -U 'USER' -d 'DATABASE' -f commands.sql
```

Common command options
- `-U`: Specifies the user to connect as.
- `-d`: Specifies the database to connect to.
- `-c`: Executes a command and exits.
- `-f`: Executes commands from a file.
- `-A`: Outputs unaligned (no extra formatting).
- `-t`: Outputs tuples only (removes headers and footers).
- `-W`: Prompt for password

Store username and passwords in a `~/.pgpass` file to avoid typing passwords at login

`~/.pgpass`
```
hostname:port:database:user:plainpass
```

Check user's hashed password

```
SELECT rolname AS username, rolpassword AS hashed_password
FROM pg_authid
WHERE rolname = 'postgres'
```

#### Extensions

Requires `pgcrypto` extension

Package: 
`postgresql-contrib`

Create the extension in the database

```
CREATE EXTENSION pgcrypto;
```

Confirm

```
postgres=# \dx
```

```
                  List of installed extensions
   Name   | Version |   Schema   |         Description
----------+---------+------------+------------------------------
 pgcrypto | 1.0     | public     | cryptographic functions
 plpgsql  | 1.0     | pg_catalog | PL/pgSQL procedural language
```

**Grant Usage Privileges**:

```
-- Grant all privileges on the database
GRANT ALL PRIVILEGES ON DATABASE ohio TO ohio_user;

-- Grant all privileges on all existing tables
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO ohio_user;

-- Grant all privileges on all existing sequences
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO ohio_user;

-- Grant all privileges on all existing functions
GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO ohio_user;

-- Set default privileges for future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON TABLES TO ohio_user;

-- Set default privileges for future sequences
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON SEQUENCES TO ohio_user;

-- Set default privileges for future functions
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON FUNCTIONS TO ohio_user;
```

`\l`: This lists all databases and their owners.
`\dx`: List currently installed extensions
`\dp`: Check table permissions
`\dp TABLE`: Verify privileges on a specific table
`\d TABLE`: Display the structure of a table
`\q`: Quit the `postgresql` prompt

Confirm All Privileges Are Granted

```
SELECT grantee, privilege_type 
FROM information_schema.role_table_grants 
WHERE table_name='TABLE_NAME';
```

```
  grantee  | privilege_type
-----------+----------------
 ohio_user | INSERT
 ohio_user | SELECT
 ohio_user | UPDATE
 ohio_user | DELETE
```

Verify the Current User and Database Connection

``` postgresql
SELECT current_user, current_database();
```

```
 current_user | current_database
--------------+------------------
 postgres     | ohio
```

Install the python postgresql connector

``` bash
yum install python-psycopg2
```

The bareos DB stuff

``` yaml
- hosts: web1.ohio.cc
  become: true
  tasks:
    - name: Check PostgreSQL connection
      community.postgresql.postgresql_ping:
        db: ohio
        login_host: web1.ohio.cc
        login_user: ohio_user
        login_password: your_password  # Use a variable for the password
      register: result
    - name: Output connection result
      debug:
        var: result
    - name: Create BareOS DB
      community.postgresql.postgresql_db:
        db: bareos
        login_host: web1.ohio.cc
        login_user: postgres
        login_password: '123123'
    - name: Restore BareOS DB
      community.postgresql.postgresql_db:
        db: bareos
        state: restore
        target: '/usr/lib/bareos/scripts/ddl/creates/postgresql.sql'
        login_host: web1.ohio.cc
        login_user: postgres
        login_password: '123123'
    - name: Create BareOS User
      community.postgresql.postgresql_user:
        db: bareos
        login_host: web1.ohio.cc
        login_user: postgres
        login_password: '123123'
        user: bareos_user
        password: "123123"
    - name: Grant DB permissions to penka
      community.postgresql.postgresql_privs:
        db: bareos
        privs: ALL
        role: bareos_user
        type: database
        state: present
        login_host: web1.ohio.cc
        login_user: postgres
        login_password: '123123'  # Use a variable for the password
```

Test a PostgreSQL connection

```
---
- name: Install and test PostgreSQL connection
  hosts: web1.ohio.cc
  become: true
  vars:
    postgres_user: "postgres"
    test_database: "postgres"

  tasks:
    - name: Install PostgreSQL server
      yum:
        name: postgresql-server
        state: present

    - name: Initialize PostgreSQL database
      command: /usr/bin/postgresql-setup --initdb
      args:
        creates: /var/lib/pgsql/data/pg_hba.conf  # Ensures this only runs if the DB hasn't been initialized

    - name: Start and enable PostgreSQL service
      service:
        name: postgresql
        state: started
        enabled: true

    - name: Test PostgreSQL connection
      become_user: "{{ postgres_user }}"
      command: psql -d "{{ test_database }}" -c "SELECT 1;"
      register: connection_test
      failed_when: "'1' not in connection_test.stdout"
      changed_when: false

    - name: Display test result
      debug:
        msg: "PostgreSQL connection test successful: {{ connection_test.stdout }}"
```