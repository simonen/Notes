
>Apress - Deploying OpenLDAP

LDAP (Lightweight Directory Access Protocol) is a protocol that offers directory services. It can be used to centrally hold identity information for users.
LDAP is used to access X.500-based directory services derived from the Directory Access Protocol (DAP). X.500 is a set of protocols that outline how user information should be stored and accessed.

Common types of directory services:
* Microsoft Active Directory
* Red Hat Directory Services (Enterprise)
* Oracle Directory Server Enterprise Edition
* FreeIPA

#### The X.500 DAP OSI model

DNS-based model

`DIT (Directory Information Tree)` : Hierarchical organization of entries.
`directory root`: dc=com
`branch (ou)`: organizational unit. Logical group for storing object information


Main `ou`s in LDAP:
* People. ou=people
* Groups. ou=groups
* Machines

#### Distinguished Names DN

A dn is an unique entry under the root.  
DN = RDN + all ancestor all the way to the root
RDN (Relative Distinguished Name)

DN example

RDN: `cn=Angela Taylor`
DN: `cn=Angela Taylor,ou=people,dc=example,dc=com`

Likewise, 

RDN: `ou=people`
DN: `ou=people,dc=example,dc=com`

A DN entry is made up of `object classes` and `attributes`. 
`object class`: describes what attributes are present or allowed to.
Classes may support other classes to extend attribute use. Attributes are described by the RND value. 

> Classes and attributes must be defined in a schema and must be unique. A good practice is to prefix custom attributes.

#### Schema

[OpenLDAP Software 2.4 Administrator's Guide Schema](https://openldap.org/doc/admin24/schema.html)

`schema`: a set of definitions that describe the data that can be stored in the directory server. Describes the syntax and matching rules for an available class and attribute definitions.

Example of a custom attribute

`dn: uid=user1,ou=people,dc=example,dc=com`
`uid: user1`
`customAttribute: VALUE`

Filters can be then used to search all instances of the `customAttribute = VALUE` in the directory.

