
2 **$ hostnamectl** NAME
3 1. Create physical volumes,
2. Create volume groups
3. Set the extent size to 8MiB
4. Create the logical volume
5. **$ lvcreate -L 500M** *VGROUP*
4 Bad mount in the fstab. Non-existent device
5 1. Create a repo file in /etc/yum.repos.d
2. add **baseurl**=ftp://repourl
3. $ dnf repolist to check the new repo
6 As root user, create a crontab 0 0 * * * COMMAND for the user
7 Make the default login shell for the user **/sbin/nologin**
8 systemctl status httpd, add the port to the firewall, service and port to SELinux
9 Add kernel option in the GRUB startup menu **init=/bin/bash**, mount -o remount,rw / because the fs is in ro mode, /.autorelabel file should be present. replace the PID 1 init process with exec /usr/systemd/system to be able to use the reboot command
10 $ tuned-adm list, tined-adm profile PROFILE
11 $ dnf search sealert
12
13 update the mandb $ mandb
14 $ usermod -aG sales USER
15
16 **server add server10.example.com iburst** to the **/etc/chrony.conf** file
17 
1. Create a dir /ldapusers/ldapuser1, 2 3, etc
2. Add /ldapusers \*(rw,no_root_squash) to the /etc/exports file to create the export
3. Check out if the dir has been exported properly $ showmount -e NFS_SERVER
4. Configure the firewall and SELinux to allow the nfs export through. Add the nfs service to the firewall. Port 2049 in SELinux
5. On the client install autofs
6. In /etc/auto.master add /ldapusers /etc/auto.ldapusers
7. Configure wildcard mount in /etc/auto.ldapusers add \* -rw nfsserver:/ldapusers/&


