
#### Using find to Search for Files

**man find**

**find** is traversing dir trees recursively by default

``` bash
find /"DIR_TO_TRAVERSE" -type "TYPE_OF_OBJECT" -name "NAME"
```

`-iname`: makes the search pattern case-insensitive

Object types:
- `b`: block
- `c`: character special
- `p`: named pipe (FIFO)
- `l`: symbolic links
- `f`: regular files
- `d`: dirs
- `s`: socket

To search for files by owner

``` bash
find / -user "USERNAME_OR_UID"
```

`-group`: Search by group name

To search for files owned by nobody (files not belonging to a valid user)

``` bash
find / -nouser -o -nogroup
```

To search for files by exact permissions 

``` bash
find / -perm "PERMISSIONS"
```

- `-perm` -PERMISSIONS: The '-' means any. `-644` disregards the special bit.
- `-perm /PERMISSIONS, /PERMISSIONS`: either or

Search for files which are writable by either their owner or their group but not by anyone

``` bash
find -perm /220, -perm /u+w,g+w, /u=w,g=w
```

To pipe the results to a command 

```bash
find / [OPTIONS] -exec "COMMAND" {} \;
```

To find and delete files owned by specific user

``` bash
find / -user "USER" -exec rm -rf {} \;
```

Similar effect can be achieved with **xargs**

``` bash
find / -name "NAME" | xargs rm -rf
```
#### Using locate

Package
`mlocate`

man locate

locate is faster than find because it uses indexed entries, which are updated on boot, thus could return correct matches. To avoid that, the db must be kept up to date

``` bash
updatedb
```

To locate the binary, source, and manual page files for a command

``` bash
whereis 'COMMAND'
```

To locate the path to a file. which also shows what a command is aliased to

``` bash
which cp
alias cp='cp -i'
        /bin/cp
```

