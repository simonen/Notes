
```
# Add the cd repo
mkdir /repo
echo "/dev/sr0 /repo iso9660 defaults 0 0" >> /etc/fstab

# in /etc/yum.repos.d/AppStream
[AppStream]
name=AppStream Repository
baseurl=file:///repo/AppStream
enabled=1
gpgcheck=0

# in /etc/yum.repos.d/BaseOS
[BaseOS]
name=BaseOS Repository
baseurl=file:///repo/BaseOS
enabled=1
gpgcheck=0

dnf repolist
# 500 MiB Partition
mkdir /mydata
# Format the partition
mkfs.ext4 /dev/sdb1
echo "/dev/sdb1 /mydata ext4 defaults 0 0" >> /etc/fstab

# users lori and laura
groupadd sales

useradd -u 2000 -G sales lori ; useradd -u 2001 -G sales laura ; useradd alex
passwd lori ; passwd laura

mkdir -p /groups/sales /groups/data
chown alex:sales /groups/sales
chown alex:data /groups/data
chmod 770 /groups/sales ; chmod 770 /groups/data

# after creating the 1 swap partition
mkswap /dev/sdb2 ; swapon /dev/sdb2
echo "UUID=a0222d1c-dcb6-4160-b8eb-b797ef5f29fb none swap defaults 0 0" >> /etc/fstab

# swap volume 512MiB
vgcreate swapvg /dev/sdb3
lvcreate -l 128 --name swaplv swapvg
mkswap /dev/mapper/swapvg-swaplv ; swapon /dev/mapper/swapvg-swaplv
echo "UUID=9furx1-j5Ul-vrRR-onwh-qL76-cLT5-1w9oMY none swap defaults 0 0" >> /etc/fstab

# Stratis vol after creating the sdc1 partition
mkdir /stratis
dnf install -y stratisd stratis-cli
systemctl enable --now stratisd
stratis pool create mypool /dev/sdc1
stratis filesystem create mypool stratisvol
# get the vol UUID
stratis filesystem list
echo "UUID=77937a9caf9d4d6eac1025ebd2c40944 /stratis xfs defaults,x-systemd.requires=stratisd.service 0 0" >> /etc/fstab
mount -a

# web server install, port 8080
dnf group install "Basic Web Server"
systemctl enable --now httpd
#change port 80 to 8080 in /etc/httpd/conf/httpd.conf

firewall-cmd --add-port=8080/tcp --permanent ; firewall-cmd --reload
semanage port -a -t http_port_t -p tcp 8080

# laura sudo
echo "laura ALL=(ALL) ALL" > /etc/sudoers.d/laura

# linda and anna
useradd -d /users/linda linda ; useradd -d /users/anna anna

# NFS exports on server/client
dnf install -y nfs-utils
systemctl enable --now nfs-server
echo "/users *(rw,no_root_squash)" >> /etc/exports
exportfs -a
showmount -e localhost

# AutoFS on the NFS client
dnf install -y autofs
systemctl enable --now autofs

echo "/users /etc/auto.users" >> /etc/auto.master
echo "* -rw localhost:/users/&" > /etc/auto.users
systemctl restart autofs

# allow through firewall [optional]
firewall-cmd --add-service=nfs --permanent ; firewall-cmd --add-service=mountd --permanent ; firewall-cmd --add-service=rpc-bind --permanent ; firewall-cmd --reload
```

