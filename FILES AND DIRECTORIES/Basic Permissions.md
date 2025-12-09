
The Linux filesystem does not work with permission inheritance

**Directory permissions**:

To read a file the parent directory must have **read** and **execute** permissions. r-x

**read**: allows the listing of files, but not file details
**execute**: allows access to the directory and its contents. File permissions depend on their own permissions. Allows **cd** on a directory. R+X allow the listing of file names and their details
**write**: allows the modification, creation or deletion of files and subdirs  if **execute** on the directory is allowed

- To **access a directory**, you need **execute (`x`) permission** on it **and all parent directories** leading to it.
- If you are missing execute permission on any parent directory, you will not be able to access the target directory, even if you have full permissions on the target.

To fix permission issues, ensure the user has execute permissions on all parent directories. You can achieve this by adding the user to the appropriate group or adjusting the directory permissions.

To modify permissions on a file, ownership of the file is required.

A file in a directory with just execute permissions can be read and modified if filename is known and if the file has read and write permissions.

Create a directory with set permissions

``` bash
mkdir -m 0700 /"DIR"
```

#### Applying Read Write and Execute Permissions

$ **chmod**

Octal signature
4 - read permissions
2 - write permissions
1 - execute permissions
0 - no permission

4+2+1 = 7 = read, write and execute
4 + 1 = 5 = read and execute permissions

Change file permissions so that only the owner has read and write permissions

``` bash
chmod 600 "FILE/DIR"
```

#### Applying Permissions Relatively with Symbolic Notation

* u: user
* g: group
* o: others
* a: all

Give the user **read** and **write** permissions

``` bash
chmod u+rw "FILE"
```

Remove a group execute permissions

``` bash
chmod g-x "FILE"
```

Add execute permission to the user and remove read permissions to the group owner

``` bash
chmod u+x,g-r "FILE"
```

Set execute permissions recursively to everything below
$ **chmod a+x** /*DIR*

Set execute permissions recursively to subdirs only
**$ chmod a+X** /*DIR*

The = operator overwrites existing permissions. In this case the group permissions are r--

``` bash
chmod g=r "FILE"
```

Use a reference file for applying permissions

``` bash
chmod --reference="REF_FILE" "FILE"
```

#### Setting Permissions in Batches with chmod

``` bash
chmod "PERMS" "FILE1 FILE2 FILE3"
chmod "PERMS" -vR /"DIR"
chmod -v "PERMS" "*.EXT"
```

Change the permissions of files only in the current dir

``` bash
find . -type f -exec chmod -v 660 {} \;
```

Change the permissions of files belonging to specific user

``` bash
find / -user "USER" -exec chmod -v 660 {} \;
```
