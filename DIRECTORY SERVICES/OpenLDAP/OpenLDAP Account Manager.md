---
tags:
  - centos
  - apache
  - php
  - ldap
---

https://www.ldap-account-manager.org/lamcms/releases

Packages:
`httpd php php-gd php-zip php-gmp
`
Configuration files:
- `/usr/share/ldap-account-manager/config`
- `/etc/httpd/conf.d/lam.apache.conf`

Object classes:
`person, posixAccount, posixGroup`, `inetOrgPerson`, `shadowAccount`

SELinux  booleans needed

```
httpd_can_connect_ldap
```

