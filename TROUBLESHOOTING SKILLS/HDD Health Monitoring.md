
Check used and free disk space

```bash
df -h
```

Maxing out on `inodes` could cause running out of disk space even if disk usage is low.
Kernel upgrades can clog up a disk space pretty easily as kernel headers accumulate, taking lots of inodes. 

Show used and free `inodes`

```
df -ih
```

```
[root@dbserver19 rsync]# df -hi
Filesystem              Inodes IUsed IFree IUse% Mounted on
devtmpfs                  227K   367  227K    1% /dev
tmpfs                     230K     1  230K    1% /dev/shm
tmpfs                     230K   486  230K    1% /run
tmpfs                     230K    16  230K    1% /sys/fs/cgroup
/dev/mapper/centos-root   8.5M   36K  8.5M    1% /
```

10-20% used `inodes` could indicate a problem

Sort folders in the parent directory by number of files

```
find . -xdev -type f | cut -d "/" -f 2 | sort | uniq -c | sort -n
```

```
      3 tmp
     13 root
     19 home
   1690 etc
   4787 var
  27103 usr
```
#### Monitoring with smartmontools

Package: 
`smartmontools`

**S.M.A.R.T** : Self-Monitoring Analysis and Reporting Technology

To check if a disk supports SMART

```
smartctl -i /dev/sda
```

```
[root@dnsserver remote]# smartctl -i /dev/sda
smartctl 7.2 2020-12-30 r5155 [x86_64-linux-5.14.0-362.8.1.el9_3.x86_64] (local build)
Copyright (C) 2002-20, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     VBOX HARDDISK
Serial Number:    VB0f2b4859-1c3cccab
Firmware Version: 1.0
User Capacity:    21,474,836,480 bytes [21.4 GB]
Sector Size:      512 bytes logical/physical
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ATA/ATAPI-6 published, ANSI INCITS 361-2002
Local Time is:    Mon Mar 25 22:57:46 2024 EET
SMART support is: Unavailable - device lacks SMART capability.
```

To get a complete data dump:

\# smartctl -x /dev/sda

To enable / disable smartcl for each disk that is monitored
\# smartctl -s on /dev/sda
\# smartctl -s off /dev/sda

To run a short health check
\# smartctl -H /dev/sda
\# smartctl -l /dev/sda -> long check

For complete report
\# smartctl -Hc /dev/sda
#### Data Recovery Utilities

**TestDisk** & PhotoRec are free and open-source data recovery utilities. 
https://www.cgsecurity.org/testdisk.pdf
