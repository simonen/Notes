
man 5 hosts_access

TCP wrappers control access to service daemons

```
Access will be granted when a (daemon,client) pair matches an entry in the /etc/hosts.allow file.
Otherwise, access will be denied when a (daemon,client) pair matches an entry in the /etc/hosts.deny file.
Otherwise, access will be granted.
```

/etc/hosts.allow: take precedence over hosts.deny
/etc/hosts.deny

Set ALL: ALL in /etc/hosts.deny

Set ALL: localhost 