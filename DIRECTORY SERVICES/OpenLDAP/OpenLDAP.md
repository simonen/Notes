---
tags:
  - centos
  - ldap
---
OpenLDAP was forked from the original project designed by the University of Michigan.
[The OpenLDAP Project](http://www.openldap.org/project)

`overlays`: give OpenLDAP advanced functionality to alter or extend normal LDAP behavior.

Packages: 
`openldap` `openldap-servers` `openldap-clients`

Service: 
`slapd` (the LDAP server)

#### Setting up an LDAP Directory

1. Generate the database admin password hash
2. Set up the `Suffix`, `RootDN`(database admin) and `RootPW`(database admin hashed password) attributes in the database
3. Monitor database `olcAccess`
4. Set up the `BaseDN` (root) of the DIT
5. Load up additional schemas for user objects
6. Security: Firewall, `SELinux`, TLS, ACLs
7. Logging
8. Password policy overlays

#### Files and Directories

- `/etc/openldap/:` LDAP server root dir
- `/etc/openldap/schema/`: Available schemata. Contains `.ldif` files with schema definition to extend the core schema`
- `/etc/openldap/certs/`:  certs and passwords are stored here
- `/etc/openldap/slapd.d/`: files comprising the `slapd` DIT (Directory Information Tree)
- `/usr/share/openldap-servers/slapd.ldif`: 
#### Databases

- `{-1}frontend`: applies global configurations to other databases
- `{0}config`: it represents the **dynamic configuration (cn=config)** itself. The `cn=config` database allows for runtime configuration changes without the need to stop or restart the OpenLDAP server. LDAP server-specific configurations go here. Accessed via the `EXTERNAL` mech, user mapped to Unix root
- `{1}monitor`: **monitoring database**. This database provides real-time statistical information about the running OpenLDAP server, such as connection data, operation counts, backend statistics, and more.
- `{2}hbd`: **HDB (Hierarchical Database, DEPRECATED for mdb)** backend. Used for storing LDAP entries, and it is a variant of the BDB (Berkeley DB) backend. 

#### LDAP Querying

``` bash
ldapsearch 'AUTH MECH' 'LDAP URI' 'BASE DN' ['SEARCH FILTER']
```

Omitting the search filter will return everything from the base dn

Show all databases

``` bash
ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase=\*
```

Show the current configuration of the `{2}hdb` database

``` bash
ldapsearch -Y EXTERNAL -H ldapi:/// -b "olcDatabase={2}hdb,cn=config"
```

- `-Y EXTERNAL`: Specifies the SASL auth mechanism. In this case `EXTERNAL` mechanism, the client authenticates to the server based on its operating system credentials, usually using certificates.
- `-H ldapi:///` : The ldap URI to connect to. In this case, it is the local LDAP UNIX domain socket (ldapi://) connecting to the ldap server, running on the local machine
- `-b cn=config`: The base DN for the search operation. The `cn=config` subtree contains server configuration
- `-s <scope>`: The scope of the search query. Can be `base`, `one`, `sub` or `children`.
- `-Z`: Tries using TLS to make the connection. `-ZZ`: TLS must be successful before continuing
- `-LLL`: print responses in LDIF format without comments
- `-W`: Prompt for password
- `-w <password>`: Include password in the cli
- `olcAccess=*`:  The filter used for the search operation. Specifies that the search should include all entries in the `olcAccess` subtree. `olcAccess` entries represent the access control rules configured for the LDAP sever. Determines who can have what access to which resources.

To query the LDAP directory externally. This will list all user objects

``` bash
ldapsearch -H ldap://'LDAP_IP':389 -x -b "dc=olympus,dc=local" -D "cn=admin,dc=olympus,dc=local" -w power "(ObjectClass=inetOrgPerson)"
```

``` bash
ldapwhoami -x -D "uid=hypathia,ou=users,dc=example,dc=com" -W
```

To look for an attribute in the schema

``` bash
ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config "(olcAttributeTypes=*)" | grep -C 3 -i olcLogLevel
```

##### Search Filtering

Filtering is used to speed up searches

```
(LOGICAL_OPERATOR(CONDITION_1)(CONDITION_2)(CONDITION_N))
```

If this expressions represents a match, additional filters can be applied to it

```
(MATCH)(LOGICAL_OPERATOR(CONDITION_1))
```

`&` (AND) logical operator. Here it applies to all conditions and they must be true for the search to return results. Like python `and`

```
(&(objectclass=person)(uid=jsmith))
```

`|` (OR) Logical operator. At least one of the conditions must be true

```
(|(objectclass=person)(uid=jsmith))
```

`!` (NOT). The condition must not be true.

```
(!(CONDITION))
```

Return all entries belonging to the `person` class with /bin/bash shells, but exclude `uid=fikretka`

```
(&(objectclass=person)(loginShell=/bin/bash)(!(uid=fikretka)))
```
#### Setting Suffix, RootDN, and RootPW

After installation, start the service and create password for the admin. Assuming root is ldap admin too.

``` bash
slappasswd
# {SSHA}J8sXrejwjt7rWhE6vMM8dT8PWuoJOQ5c
```

Add the attribute `olcRootPW`, which stores the admin password

`.ldif`
```
dn: olcDatabase={2}hdb,cn=config
changeType: modify
add: olcRootPW
olcRootPW: {SSHA}J8sXrejwjt7rWhE6vMM8dT8PWuoJOQ5c
```

`.ldif`
``` bash
dn: olcDatabase={2}hdb,cn=config # backend to be modified
changeType: modify # type of change
replace: olcSuffix # type of modification
olcSuffix: dc=olympus,dc=local # updated attribute
-
replace: olcRootDN
olcRootDN: cn=admin,dc=olympus,dc=local
```

- `olcSuffix`: Directs queries for `dc=olympus,dc=local` (base DN for the directory) to this database instance
- `RootDN`: The DN of the root user for the database with full privileges
- `olcRootPW`: hashed database root password, generated with `slappasswd`

#### Access Control Lists

Access controls are attached to the database configuration. Defined by the `olcAccess` attribute. Can be stacked. Evaluated from top to bottom.

To see the current access control list

``` bash
ldapsearch -H ldapi:/// -Y EXTERNAL -b "olcDatabase='{N}DATABASE',cn=config" -LLL
```

`olcAccess` directives control access to LDAP entries and attributes using a combination of rules, permissions and conditions. Can be stacked, requires indexing `olcAccess{0}`, `olcAccess{1}`, etc.

The general syntax for the `olcAccess` directive:

```
olcAccess: [{ N }]to <what> by <who> <access-level> [ control ]
```

`what`: Specifies what entries or attributes the rule applies to. Scopes:
* `dn.base`: Applies only to a specific, single entry with exact DN (e.g., `cn=admin,dc=example,dc=com`)
* `dn.one`: Applies to all entries one level below the specified DN, but not the DN, or deeper levels
* `dn.subtree`: Applies to the base DN and everything below it.
* `dn.children`: Applies to everything below the base DN, but not the DN
* `attrs=userPassword`: (filter)
`who`: (Client requesting information). Defines who the rule applies to. Multiple `who` declarations are allowed. Top to bottom.
* `anonymous`: Unauthenticated users. Any user trying to bind(log in) without creds
* `self`: The entry owner
* `users`: Authenticated users
* `peername.ip=`: Client's IP address
* `sockname.ip`: Server's address
* `dn.exact='DN'`: Single specific entry
* `group.exact='GROUP_DN'`: Restricts permissions to strict group membership
* `ssf`: Security strength factor of the connection
`access`: Specifies the level of access granted
`control`: (optional) Defines how the list processed after entry.

Access levels

| Access   | Privileges                                           |
| -------- | ---------------------------------------------------- |
| none     | Allows no access                                     |
| disclose | Allows no access but returns an error                |
| auth     | Enables bind operations (authenticate)               |
| compare  | Allows you to compare the entry                      |
| search   | Allows to search in that part of the DIT             |
| read     | Allows read access                                   |
| write    | Allows write access                                  |
| manage   | Allows all access and the ability to delete entities |

Example: Restrict access to `userPassword` attribute of the `ou=people` subtree:

```
olcAccess: to dn.subtree='ou=people,dc=example,dc=com' attrs=userPassword
	by self write
	by anonymous auth
	by * none
```

The rule applies to the `userPassword` attribute
* `self write`: Allows a user to write to their own `userPassword` attribute (change their password)
* `by anonymous auth`: Allows anonymous users to just authenticate.
* `by * none`: Stops further processing.

> Put the most sensitive access rules to the top. Terminate access rules with `by * none` to restrict further access.

To allow authentication only if it has TLS security strength factor (`tls_ssf`) of 128

```
by [<who>] tls_ssf=128 auth
```

##### Modifying ACLs 

`acl_mod.ldif`
```
dn: olcDatabase={N}DATABASE,cn=config
changetype: modify
replace: olcAccess
olcAccess: {N}<access list>
-
add: olcAccess
olcAccess: {N+1}<access list>
```

Load with `ldapmodify`
#### Schema

OpenLDAP schemas define the rules for directory entries, including how the attributes are structured, which attributes and object classes can be used  and constraints (e.g., syntax matching rules)

A schema is made up of the following components:
* Attributes: Individual pieces of data, such as `cn`, `sn`, `uid`, etc.
* Object Classes: Define a collection of attributes that an entry can have. For example, the `inetOrgPerson` object class include attributes like `cn`, `sn`, `mail`
* Syntaxes: Specify the data type of attribute values, e.g., string, integer or binary
* Matching Rules: Define how attribute values are compared against searches (e.g., whether the comparison should be exact (equality), partial (substring), or approximate)

Schemas are defined in `.schema` files. Corresponding files in the `.ldif` format are derived from the `.schema` files.

Object class definition

```
objectClass ( <OID> 
NAME <name> 
DESC <description> 
SUP <parent class> 
<class type> 
<MUST|MAY> ( attritubutes ) )
```

- `OID`: Private Enterprise Number or Globally unique identifier. Registered at the IANA
- `NAME`: The name of the object class
- `SUP`: (superior) statement specifies the parent object class from which this object class inherits.`top` means no parent. The highest in the object class hierarchy. 
- `class types`:
	`AUXILIARY`: An auxiliary object can be added to an existing entry to provide additional attributes. Cannot be the primary object class of an entry.
	`STRUCTURAL`: Defines the main identity or role of the entry
	`ABSTRACT`: A base class that cannot be directly instantiated, only inherited. Like `top`
- `MUST`: defines attributes that are required when this object class is used. `MUST dc` means that the object must have the `dc` attribute.
Spacing inside of the ( ) is strict

Example: Object class

```
objectClasses: ( 2.16.840.1.113730.3.2.2
  NAME 'inetOrgPerson'
  DESC 'InetOrgPerson'
  SUP organizationalPerson
  STRUCTURAL
  MUST ( cn $ sn )
  MAY ( uid $ mail $ telephoneNumber $ mobile ) )
```

Example: Attribute Definition

```
attributeTypes: ( 2.5.4.3 
  NAME 'cn'
  DESC 'Common Name'
  EQUALITY caseIgnoreMatch
  SUBSTR caseIgnoreSubstringsMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 
  SINGLE-VALUE )
```

- `EQUALITY`: matching rule for equality searches
- `SUBSTR`: matching rule for substring searches
- `SYNTAX`: Specifies the data type or format for the attribute
- `SINGLE-VALUE`: Indicates that this attribute can only have one value (e.g., passwords)

Attributes must be declared in the schema, and one attribute can be included in one or more object classes. By default are MULTI-VALUE (e.g., email addresses). Attributes can be hierarchical
* They are not terminated with a top
* The absence of the SUP definition indicates the end of the hierarchy. 

Example of attribute inheritance:
- `name`: parent of the `cn`, `gn`, and `sn`
##### Common Schemas 

OpenLDAP comes with several default schema that provide basic object classes and attributes. Most common are
1. `core.schema`
	* defines basic attributes and object classes like `organization`, `organizationalUnit (ou)`, and `person`, `dcObject (dc)`
2. `cosine.schema`: Provides `dnsDomain` object class and `host` attribute
3. `inetorgperson.schema`
4. `nis.schema`: provides user account objects and attributes, such as `PosixAccount` and shadow password settings
5. `ppolicy.schema`: Adds password policy features
6. `ldapns.schema`: Contains attributes and object classes used by LDAP servers for naming services.

`/etc/openldap/schema/`: contains available schemata 

View the currently loaded schemata

``` bash
ldapsearch -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config dn
```

```
dn: cn={0}core,cn=schema,cn=config
```

Loading new schema is done dynamically with `SCHEME.ldif`. Load the `ppolicy` schema

```
ldapadd -Y EXTERNAL -H ldapi:/// -f ppolicy.ldif
```

The schema is loaded 

```
dn: cn=schema,cn=config
dn: cn={0}core,cn=schema,cn=config
dn: cn={1}ppolicy,cn=schema,cn=config
```

##### Creating Custom Schema

Define the object class and attribute in a `custom.schema` file

```
# Id$

attributetype ( 1.1.3.10
  NAME 'exampleActive'
  DESC 'Example User Active'
  SINGLE-VALUE
  EQUALITY booleanMatch # True of False values
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 ) # boolean data type

objectclass ( 1.1.1.2
  NAME 'exampleClient'
  SUP top
  AUXILIARY
  DESC 'Example.com User objectclass'
  MAY ( exampleActive ) )
```

Create a custom_schema.conf file

`custom_schema.conf`
```
include '/PATH TO CUSTOM.schema FILE'
```

Convert the `.schema` to `.ldif` using `slaptest`. The `.ldif` will be created in a temporary dir

``` bash
slaptest -f 'CUSTOM.schema' -F 'TEMP LDIF DIR'
# config file testing succeeded
```

This creates the ldif-formatted file `/etc/openldap/schema/ldif_converted/cn=config/cn=schema/cn={0}exampleactive.ldif`

Edit the file to leave only this

```
dn: cn={0}exampleactive
objectClass: olcSchemaConfig
cn: {0}exampleactive
olcAttributeTypes: {0}( 1.1.3.10 NAME 'exampleActive' DESC 'Example User Act
 ive' EQUALITY booleanMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE
  )
olcObjectClasses: {0}( 1.1.1.2 NAME 'exampleClient' DESC 'Example.com User o
 bjectclass' SUP top AUXILIARY MAY exampleActive )
```

Check the indexing of currently loaded schemata. The custom schema index must not conflict with existing schemes. Change the index to the next available number

`dn: cn={0}exampleactive` => `dn: cn={2}exampleactive,cn=schema,cn=config`

Save the file in the `/etc/openldap/schema/` dir as `.ldif`. Name should match the `.schema` file. Load the new schema.

``` bash
ldapadd -Y EXTERNAL -H ldapi:/// -f exampleActive.ldif
```

```
adding new entry "cn={2}exampleactive"
ldap_add: Server is unwilling to perform (53)
        additional info: no global superior knowledge
```

The new schema is loaded.

```
[root@ldap schema]# ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config dn

dn: cn=schema,cn=config

dn: cn={0}core,cn=schema,cn=config
dn: cn={1}ppolicy,cn=schema,cn=config
dn: cn={2}exampleactive,cn=schema,cn=config
```
#### Indexes

Indexes are used to speed up searches on the database. As a rule, the most searched objects, like `cn` and `uid` used for user authentication, must be indexed.

`olcDbIndex`: attribute specifies which attributes in the database should be indexed and which types of indexing should be applied.
* `eq`(Equality index): indexes attribute values for exact matches (e.g., `uid=john.doe`)
* `sub` (Substring index): indexes attribute values for substring searches (e.g., `cn=*doe`)
* `pres` (Presence index): indexes attributes for presence checks (e.g., `uid=*` or `cn=*`, to check if the attribute exists)
* `approx` (Approximate index): indexes attributes for approximate matching, used for 'sounds-like' searches or typos

```
dn: olcDatabase={2}hdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcHdbConfig
olcDatabase: {2}hdb
olcDbDirectory: /var/lib/ldap
olcSuffix: dc=ohio,dc=cc
olcRootDN: cn=admin,dc=ohio,dc=cc
olcRootPW: {SSHA}GuUBhARds4oP52Gecstl7OO4AQ7Dsd7l
olcDbIndex: objectClass eq,pres
olcDbIndex: ou,cn,mail,surname,givenname eq,pres,sub
```

`objectClass eq`: Creates an equality index on the `objectClass` attribute. This allows for fast searches when filtering by object class

Re-index the db after modifying the db

``` bash
slapindex -b 'DATABASE'
```

#### LDAP Objects

**OpenLDAP** stores its information in backends or databases. The most commonly used database is the **Berkley DB** backend - bdb. As of more recently - hdb.

- `LDIF` - LDAP Data Interchange Format: text format designed to retrieve or update information from and on the LDAP server.
- `olc`: OpenLDAP configuration
- `dn`: (Distinguished Name) attribute: used to uniquely identify objects

#### The LDIF Format

The LDIF is a specification on how to add, modify and remove entries in a LDAP database

```
dn: <the distinguished name to change>
changetype: <add|replace|delete>
<attribute or objectclass>: value
```

```
dn: dc=example,dc=com
objectclass: dcObject
objectclass: organizationalUnit
dc: example
ou: example
```

#### Modifying Objects

Simply editing configuration files will not work. When working with files, .ldif files with specifically formatted desired operations are created first and the **ldapmodify** command is executed against them to modify the server configuration.

``` bash
dn: DN of the entry to modify
changeType: TYPE OF CHANGE [replace | delete]
replace: ATTRIBUTE
ATTRIBUTE: NEW VALUE
```


The file starts with the database backend that is gonna be receiving the modifications
Followed by changeType, which implies modifications, and their exact kind - replace

Execute the changes

``` bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f my_config.ldif
```

Modification on the CLI directly

``` bash
ldapmodify -Y EXTERNAL -H ldapi:/// <<EOF 
dn: cn=config 
changetype: modify 
replace: olcLogLevel 
olcLogLevel: stats config 
EOF
```

And again, execute the **ldapmodify** command

The following command is querying the LDAP server's configuration (specifically, the `cn=config` subtree) using the EXTERNAL SASL mechanism for authentication. It retrieves information about all database backends (`olcDatabase`) configured on the LDAP server. This command is useful to inspect the LDAP server's configuration, including its database backend settings.

``` bash
ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase=\*
```

#### Adding Objects

Adding objects file structure. Multiple object classes are allowed, providing the necessary attributes.

`object.ldif`
```
dn: 'DN OF THE OBJECT'
objectclass: <object class>
attribute: <value>
```

##### Setting Up the BaseDN (Root) of the DIT

The base entry (or suffix) for the **Directory Information Tree** (DIT) defines the root of the LDAP directory hierarchy and is the first object to add. It provides the starting point for all further entries in the directory.

The base entry is often a domain component (dc) object or an organization. It is based on the `dcObject` object class, which is declared in the `core.schema`

```
objectclass ( 1.3.6.1.4.1.1466.344 NAME 'dcObject' DESC 'RFC2247: domain component object' SUP top AUXILIARY MUST dc )
```

Because the `dcObject` is an AUXILIARY object class, it cannot be used to create an entry.
The `organization` is a STRUCTURAL object class (allows to create entries) and must be included. It requires the `o` attribute.

To check what attributes must be defined when adding an object of particular class, we must consult the `schema` located in `/etc/openldap/slapd.d/cn=config/cn=schema`

`/etc/openldap/slapd.d/cn=config/cn=schema/cn={0}core.ldif`
```
olcObjectClasses: {2}( 2.5.6.4 NAME 'organization' 
	DESC 'RFC2256: an organization' 
	SUP top STRUCTURAL 
	MUST o 
	MAY ( userPassword $ searchGuide $ seeAlso $ businessCategory 
		$ physicalDeliveryOfficeName $ st $ l $ description ) )
```

`base_dn.ldif`
```
dn: dc=ohio,dc=cc
objectClass: dcObject
objectClass: organization
dc: ohio
o: ohio
```

- `dn: dc=ohio,dc=cc`: The distinguished name (DN) of the base entry, which becomes the root of the directory tree
- `dc`: domain component. This attribute is mandatory for the `dcObject` object class.
- `o`: organization. This attribute is mandatory for the `organization` object class
- `objectClass: dcObject` : Provides the domain component `dc` attribute
- `objectClass: organization` : Provides the STRUCTURAL organization attribute `o`

Create the basedn entry. For domain-related tasks we authenticate as LDAP database admin, not external user.

``` bash
ldapadd -f 'CONFIG.ldif'-D cn=admin,dc=ohio,dc=cc -w 'PASS'
```

Verify if the entry has been created

``` bash
ldapsearch -x -b dc=olympus,dc=local ["(object)"]
```

##### Adding an Organizational Unit

Create an OU called `users` to store all LDAP users. Create a users.ldif file with

```
dn: ou=users,dc=olympus,dc=local
objectClass: organizationalUnit
ou: users
```

Add the new OU with the ldapadd command

``` bash
ldapadd -f users.ldif -D cn=admin,dc=olympus,dc=local -w 'PASS'
```

Verify

``` bash
ldapsearch -x -b dc=olympus,dc=local ou
```

```
# extended LDIF
#
# LDAPv3
# base <dc=olympus,dc=local> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#
# users, olympus.local
dn: ou=users,dc=olympus,dc=local
objectClass: organizationalUnit
ou: users
```

##### Adding Users

To create a POSIX user account, meaning to be recognized by a Unix-like systems, the key attributes of a POSIX user must be provided. The `posixAccount` object class, defined in the `nis.schema`, provides those attributes.

`nis.schema`
* `posixAccount`: 
	* MUST: `cn` $ `uid` $ `uidNumber` $ `gidNumber` $ `homeDirectory`
	* MAY:  `userPassword` $ `loginShell` $ `gecos` $ `description` )
* `posixGroup`:
	* MUST: `cn` $ `gidNumber`
	* MAY: `userPassword $ memberUid $ description`
* `shadowAccount`: For additional password options
	* MUST: `uid`
	* MAY ( `userPassword` $ `shadowLastChange` $ `shadowMin` $
              `shadowMax` $ `shadowWarning` $ `shadowInactive` $
              `shadowExpire` $ `shadowFlag` $ `description` ) 

Optional user attributes 
* `objectClass: person`:  defined in the `core.schema`
	* MUST: `cn`, `sn`
* `objectClass: inetOrgPerson`: defined in the `inetOrgPerson.schema`
	* MAY: `initials`, `mail`, `givenName`, etc.

To define a POSIX user

```
dn: uid=fikretka,ou=users,dc=olympus,dc=local
objectClass: person
objectClass: posixAccount
objectClass: shadowAccount
cn: Fikretka
sn: Fikretska
uidNumber: 10006
gidNumber: 10006
[gecos: "Description"]
userPassword: {SSHA}ccTh4Hm+IVlvji/PS/+SyBQFkrx5YaN8
homeDirectory: /home/fikretka
loginShell: /bin/bash

dn: cn=fikretka,ou=groups,dc=olympus,dc=local
objectClass: posixGroup
cn: fikretka
gidNumber: 10006
memberUid: fikretka
```

Load the schemata

``` bash
ldapadd -Y EXTERNAL -H ldapi:// -f /etc/openldap/schema/nis.ldif
ldapadd -Y EXTERNAL -H ldapi:// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:// -f /etc/openldap/schema/inetOrgPerson.ldif
```

Now, add the user.

``` bash
ldapadd -x -D cn=admin,dc=ohio,dc=cc -f 'USER.ldif' -w 'PASS'
```

Test if a user exists

```bash
ldapwhoami -x -D "cn=webadmin,ou=meta,dc=ohio,dc=cc" -w 'pASS' [-nv]
```
##### Adding Groups

Create the .ldif specifying the group

```
dn: cn=scientists,ou=people,dc=ohio,dc=cc
objectClass: groupOfNames
cn: scientists
member: cn=fikretka,ou=people,dc=ohio,dc=cc
```

Add the group

``` bash
ldapadd -x -D cn=admin,dc=ohio,dc=cc -f scientists.ldif -W
```

##### Deleting Objects

Delete individual objects from the CLI

``` bash
ldapdelete "cn=Archimedes of Syracuse,ou=users,dc=olympus,dc=local" -D cn=admin,dc=olympus,dc=local -w 'PASS' -v
```

Delete multiple entries from an `.ldif` file. Provide a list of DNs

`del_users.ldif`
```
cn=seven,ou=people,dc=ohio,dc=cc
cn=eight,ou=people,dc=ohio,dc=cc
cn=nine,ou=people,dc=ohio,dc=cc
```

Execute the `ldapdelete` command

``` bash
ldapdelete -x -D cn=admin,dc=ohio,dc=cc -f del_users.ldif -w 'P' -v
#ldap_initialize( <DEFAULT> )
#deleting entry "cn=seven,ou=people,dc=ohio,dc=cc"
#deleting entry "cn=eight,ou=people,dc=ohio,dc=cc"
#deleting entry "cn=nine,ou=people,dc=ohio,dc=cc"
```

#### Adding Modules

A list of available modules can be found in the `slapd.ldif` file. Make sure they exist in the system too. The module section is commented out, meaning its not in the `cn=config` tree yet.

`slapd.ldif`
```
#dn: cn=module,cn=config
#objectClass: olcModuleList
#cn: module
#olcModulepath: /usr/lib/openldap
#olcModulepath: /usr/lib64/openldap
#olcModuleload: accesslog.la
#olcModuleload: auditlog.la
#olcModuleload: back_dnssrv.la
```

Add the `cn=module,cn=config` entry in the `cn=config` tree

`add-module.ldif.ldif`
```
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulepath: /usr/lib64/openldap
olcModuleLoad: <MODULE>.la
```

Execute the .ldif

``` bash
ldapadd -Y EXTERNAL -H ldapi:/// -f add-module.ldif
```

Display loaded modules

``` bash
ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config "(objectClass=olcModuleList)" -LLL
slapcat -n 0 | grep olcModuleLoad
```

```
dn: cn=module{0},cn=config
objectClass: olcModuleList
cn: module{0}
olcModulePath: /usr/lib64/openldap
```

#### Overlays WIP

Add `ppolicy` overlay configuration for fine-grained password policy

Load the `ppolicy` module

`add_ppolicy.ldif`
```
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: ppolicy.la
```

```
ldmodify -Y EXTERNAL -H ldapi:/// -f add_ppolicy.ldif
```

Result

```
dn: cn=module{0},cn=config
objectClass: olcModuleList
cn: module{0}
olcModulePath: /usr/lib64/openldap
olcModuleLoad: {0}ppolicy.la
```

Create a default policy object

```
cn=default,ou=policies,dc=ohio,dc=cc
```

#### Securing SLAPD with TLS

For a list of loaded and available TLS-related attributes

```
ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config | grep TLS
```

Main TLS attributes

```
olcTLSCACertificatePath: /etc/openldap/certs
olcTLSCertificateFile: "OpenLDAP Server"
olcTLSCertificateKeyFile: /etc/openldap/certs/password
```

Additional TLS

```
olcTLSVerifyClient: [ never|allow|try|demand ]
olcTLSCRLCheck: [ none|peer|all ] # TLS Revocation list
olcTLSProtocolMin: [ 0.0, 3.1-3.4 ] # 3.3 = TLS 1.2 recommended
olcTLSCipherSuite: <cipher list>
```

#### Working with SSF

Security Strength Factor defines the minimum security strength allowed for connections to the `slapd` server. Controlled by the `olcSecurity` attribute. TLS, for example, uses ssf based on the key size used for encryption.

The `olcSecurity` attribute is used to control whether SASL, TLS or other sec mechanism is place before certain operations are allowed.

View the currently configured security policy

``` bash
ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcSecurity
```

```
olcSecurity: <security_flags>
```

Common security flags:

* `ssf=<value>`: The effective security strength of the connection
* `update_ssf=<value>`: The required encryption for writes or other modifications to the directory
* `simple_bind=<value>`: The required minimum encryption used for binding
* `tls=<value>`

`ssf` values:
* 0: no protection (default)
* 1: TLS protection
* 56: DES or other weak ciphers
* 112: Triple DES or other strong ciphers
* 128: strong ciphers
* 256: AES, SHA ciphers

`ssf` can be set globally on `cn=config` or per database. 

```
dn: cn=config
changetype: modify
add: olcSecurity
olcSecurity: <sec flags>
```

> Do not use `olcVerifyClient: demand` + `olcSecurity ssf=128 update_ssf=256 simple_bind=128 tls=256` at the same time on the `cn=config`, because it will lock you out!

When TLS is not being when making queries

```
ldap_bind: Confidentiality required (13)
        additional info: TLS confidentiality required
```
#### LDAP Authentication

Check supported SASL auth mechanisms

``` bash
ldapsearch -x -LLL -H ldapi:/// -s base -b "" supportedSASLMechanisms
```

Instead of authenticating using the local passwd directory, users will be able to authenticate against an external LDAP server, where the credentials are stored, and thus gain access to the particular machine. In order to do so, the client machine must be configured to pass the credentials over to the LDAP server.

##### Authentication stack

* Auth (LDAP) Client libraries
* PAM hook
	* Auth against LDAP directory
	* Auto-creation of $HOME /password-auth-ac:29:session     optional      `pam_mkhomedir.so umask=0077`
* nsswitch

`SSSD` and `NSLCD` provide mechanisms for remote authentication against an external directory.

List supported authentication mechanisms

``` bash
ldapsearch -H ldapi:/// -Y EXTERNAL -b "" -s base -LLL supportedSASLMechanisms
```

##### PAM Pluggable Authentication Modules

[PAM](https://www.redhat.com/sysadmin/pluggable-authentication-modules-pam)

Linux Pluggable Authentication Modules is a suite of libraries that allow a Linux system administrator to configure methods to authenticate users. It provides a flexible and centralized way to switch authentication methods for secured applications by using configuration files instead of changing application code

Modules location: 
`/etc/pam.d/`

The `sssd-ldap` packages provides the **pam_sss.so** PAM shared object, necessary for remote authentications

##### SSSD

System Security Service Daemon

https://sssd.io/
[SSSD](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_authentication_and_authorization_in_rhel/understanding-sssd-and-its-benefits_configuring-authentication-and-authorization-in-rhel)

Provides a set of daemons to manage access to remote directories and authentication mechanism, an NSS and PAM interface toward to the system and a pluggable back end to connect to multiple different account sources

**Packages**: 
`sssd`
`sssd-client `
`sssd-ldap`
`openldap-clients` (optional)
`oddjob-mkhomedir` (optional). Auto create user homedirs on login

**Files and Directories** 
- `/etc/sssd/sssd.conf `
- `/usr/lib64/sssd/conf/sssd.conf `- sample config file

> The `sssd.conf` must have `0600` permissions for the daemon to start

``` bash
chmod 0600 /etc/sssd/sssd.conf
```

`/etc/sssd/sssd.conf`
```
[sssd]
config_file_version = 2
services = nss, pam
domains = ohio.cc
debug_level = INT

[nss]

[domain/ohio.cc]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://ldap.ohio.cc
ldap_search_base = ou=users,dc=olympus,dc=local
# TLS
ldap_id_use_start_tls = true
ldap_tls_reqcert = demand
ldap_tls_cacert = /etc/openldap/certs/cacert.pem
ldap_tls_ciphers = HIGH:!MEDIUM:!LOW:!aNULL:!NULL:!SHA:!MD5:!RC4
ldap_tls_protocol_min = 1.2

[pam]
ldap_schema = rfc2307
```

SSSD installs the PAM modules needed for authentication

`/etc/pam.d/system-auth -> /etc/authselect/system-auth`
```
auth required pam_sss.so
session optional pam_sss.so
account sufficient pam_sss.so
password sufficient pam_sss.so
```

Now, the ldap client should be able to query the ldap user database

```bash
[root@localhost sssd] id fikretka
uid=10006(fikretka) gid=10006(fikretka) groups=10006(fikretka)
```

To clear the SSSD cache

```
sss_cache -E
```
##### Configure authentication

Use `authselect` to configure the pam modules, `nsswitch` automatically to work with `sssd`

``` bash
authselect select sssd --force 
```

Or use authconfig or authconfig-tui

``` bash
authconfig --enableldap --enableldapauth --ldapserver='LDAP_SERVER_IP' --ldapbasedn="dc=olympus,dc=local" --enableldaptls --update
```

##### Home Dir on Login

Install and enable the `oddjob-mkhomedir` service

```
dnf install oddjob-mkhomedir -y
systemctl enable --now oddjobd.service
```

```
authselect select sssd with-mkhomedir --force
```

Or manually put the following line in the appropriate pam files

```
session     optional       pam_mkhomedir.so umask=0077
```

- `/etc/pam.d/login`
- `/etc/pam.d/password-auth`
- `/etc/pam.d/su`

Logging in will now automatically create a `homedir` for the user

#### Troubleshooting LDAP

[OpenLDAP Issue Tracking System](https://bugs.openldap.org/)


Show debug information

``` bash
ldapsearch -x -d1 -b dc=olympus,dc=local -H ldapi:/// [-ZZ]
```

Test the current configuration for errors

``` bash
slaptest -u
```

To increase the debug level of the **sssd**, add **debug_level = 7** to **/etc/sssd/sssd.conf**

Show issues with `slaptest`

``` bash
slaptest [-d LEVEL]
```

Dump selected db configuration

``` bash
slapcat -n 'DB_NUMBER'
```

``` bash
slapcat -b 'BASE_DN'
```

#### Reference

Apress – Learn Centos Linux Network Services.pdf - Antonio Vazquez
Apress - Pro Linux System Administration 2nd 2017 - Denis Matotek

```
├── slapd.d
│   ├── cn=config
│   │   ├── cn=schema
│   │   │   └── cn={0}core.ldif
│   │   ├── cn=schema.ldif
│   │   ├── olcDatabase={0}config.ldif
│   │   ├── olcDatabase={-1}frontend.ldif
│   │   ├── olcDatabase={1}monitor.ldif
│   │   └── olcDatabase={2}hdb.ldif
│   └── cn=config.ldif

```

#### Log Levels

Logging level is additive. Different levels can be combined.

| Level | Keyword / Description                                                                                                                                            |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -1    | (any) Turns on all debugging information. This is useful for finding out where your LDAP server is failing before you make your logging level more fine-grained. |
| 0     | Turns all debugging off. This is recommended for production mode.                                                                                                |
| 1     | (0x1 trace) Traces function calls.                                                                                                                               |
| 2     | (0x2 packets) Debugs packet handling                                                                                                                             |
| 4     | (0x4 args) Provides heavy trace debugging (function args                                                                                                         |
| 8     | (0x8 conns) Provides connection management                                                                                                                       |
| 16    | (0x10 BER) Prints out packets sent and received.                                                                                                                 |
| 32    | (0x20 filter) Provides search filter processing.                                                                                                                 |
| 64    | (0x40 config) Provides configuration file processing.                                                                                                            |
| 128   | (0x80 ACL) Provides access control list processing                                                                                                               |
| 256   | (0x100 stats) Provides connections, LDAP operations, and results (recommended)                                                                                   |
| 512   | (0x200 stats2) Indicates stats log entries sent.                                                                                                                 |
| 1024  | (0x400 shell) Prints communication with shell back ends.                                                                                                         |
| 2048  | (0x800 parse) Parses entries.                                                                                                                                    |
| 16384 | (0x4000 sync) Provides LDAPSync replication                                                                                                                      |
| 32768 | (0x8000 none) Logs only messages at whatever log level is set.                                                                                                   |
loglevel 480 = 32 (search filter) + 64 (configuration processing) + 128 (access control list processing) + 256 (connections and LDAP operations results)

Setting the loglevel

`loglevels.ldif`
```
dn: cn=config
changeType: modify
replace: olcLogLevel
olcLogLevel: 480
```

Execute the changes

``` bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f loglevels.ldif
```

To confirm

```
[root@ldap slapd.d]# ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config -LLL olcLogLevel
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
dn: cn=config
olcLogLevel: 480
```

#### Backing Up the LDAP Database

Dump the database to backup with `slapcat` to an `.ldif` file

``` bash
slapcat -b 'BASE_DN' -l "BACKUP.ldif"
```

Restore the database 

``` bash
slapadd -b 'BASE_DN' -l "BACKUP.ldif" -F /etc/openldap/slapd.d/ -v [-c] [-u]
```

`-c`: Ignore errors. Useful when the backup contains existing entries
`-u`: Dry-run
`-v`: verbose
