
- `/etc/passwd` - contains account information
- `/etc/shadow` - contains authentication information

**LDAP**: Lightweight Directory Access Protocol: started as a protocol to get information from hierarchical directory servers
**Kerberos**: A service that can be used on top of an LDAP directory for authorization
**Identity Management**: Red Hat Identity Management (IdM) is based on the **FreeIPA** project
**IdM** provides easy solution for LDAP/Kerberos server setup. Includes **DNS** and **NTP** servers

#### Kerberos Basics

Kerberos authentication is based on credentials called **tickets**, secured with encryption
Instead of passwords, tickets are sent across the network, encrypted with the user's password.

Tickets are issued by a central server **KDC** (Key Distribution Center)

**Realm**: a group of hosts that use the same **KDC** to get tickets from.
Can be compared to a **Windows Domain** or an **LDAP Suffix**
The DNS domain of the Kerberos realm is written in all caps
i.e the example.com domain is EXAMPLE.COM realm

Kerberos is used for secure access to servers running applications - app servers

1. Users enters his password 
2. login program converts the username to **kerberos principal name** 
3. login program sends the login request to the **KDC** authentication service
5. the **KDC** generates a secret session key used as a ticket granting ticket **TGT**
6. The KDC keeps one copy and encrypts another with the user's password which is sent back to the login program
7. The login program attempts to decrypt the TGT with the user's password. If successful, the user now has a TGT which grants him access to other Kerberos-enabled services. The user does not have to enter the password again, which means Kerberos is **SSO** (Single Sign-On)
8. Time stamps play an important role. All servers in a Kerberos environment must be time synced, otherwise authentication problems can occur.

#### Kerberos Principals

Principals can be users or network services
principal: **primary/instance@REALM**

**instance** part is optional

EXAMPLE:
**nfs/server1.example.com\@EXAMPLE.COM**

* **nfs**: primary
* **server1.example.com**: the name of the host the principal belongs to
* **EXAMPLE.COM**: the realm the principal belongs to

**service names** usually include all three parts
**usernames** usually exclude the **instance** part: lisa\@EXAMPLE.COM

Services store their passwords in a **keytab** file which allows the server to log into Kerberos without human intervention

/etc/krb5.keytab: contains the names of all service principals on the server + the password

$ **strings** **KEYTAB_FILE**: displays the contents of a keytab file
$ **klist**: does the same

#### LDAP Authentication with Kerberos Authorization

Domains in LDAP are called **containers**. Users need to specify which container they will be using as base environment, a.k.a 
* **container**: **DNS** domains in **LDAP**
* **base context**: the container users are using as base environment
* **distinguished name**: the name of the base context including the name and full path

The **authconfig** utility

* **authconfig**: command-line utility that enables authentication setup
* **authconfig-tui**: menu-driven interactive interface for auth setup
* **authconfig-gtk**: graphical interface for auth setup.

CONF FILES:

* **/etc/ldap.conf**: **LDAP** client conf. Defines LDAP server to be used
* **/etc/krb5.conf**: Kerberos specific info
* **/etc/sssd/sssd.conf**: info used by the **system security services daemon** used for retrieving and caching user and authentication info
* **/etc/nslcd.conf**: alternative to the **sssd**. Not default conf
* **/etc/nsswitch.conf**: indicates which server should be contacted for retrieving authentication information. Contains lines like **passwd: files sss** which indicates that local conf files should be used first, after which **sssd** should be used
* **/etc/pam.d**/\*: contains files that define how authentication should be handled for various services
* **/etc/openldap/cacerts**: stores the root certificate authorities used for validating **SSL** certificates and identifying LDAP services
* **/etc/sysconfig/authconfig**: contains variables that specify how authconfig utilities should work

#### Using nslcd and sssd as Authentication Back-end Services
