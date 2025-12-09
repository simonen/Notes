
Open the manual page of a particular command

``` bash
man "COMMAND"
```

List man pages containing a keyword in their short descriptions

``` bash
man -k "COMMAND"
```

List man pages containing a keyword

``` bash
man -K "COMMAND"
```

**whatis** searches a summary database of commands for a complete word match

``` bash
whatis "COMMAND"
```

```
useradd(8) – create a new user or update default new user information
```

**apropos** searches the whatis database and returns commands and functions that contain the searched string

``` bash
apropos "COMMAND"
```

```
[kimchen@localhost ~]$ apropos whoam
ldapwhoami (1)       - LDAP who am i? tool
whoami (1)           - print effective userid
```



