
Replace the first occurrence of a word with another
$ sed "s/OLD_WORD/NEW_WORD/" 

Replace all occurrences 
$ sed "s/OLD_WORD/NEW_WORD/g" 

To print lines starting with a character
$ ls -l | sed -n '/^d/ p' 
```
prints only dirs
[root@dbserver19 log]# ls -l | sed -n '/^d/ p'
drwxr-xr-x. 2 root  root     204 Jan 19 18:56 anaconda
drwx------. 2 root  root      23 Jan 19 19:02 audit
drwxr-x---. 2 mysql mysql     25 Apr  2 11:27 mariadb
```

