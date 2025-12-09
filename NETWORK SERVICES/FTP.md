---
tags:
  - "#centos"
---
File Transfer Protocol

Package
**vsftpd** : Very Secure FTP

**Ports** 20, 21
#### Files and Directories

**etc/vsftpd/vsftpd.conf** - main config file
**/var/ftp**: default data directory

* **control connection**: established first, uses port 21
* **data connection**: established every time a listing or transfer is requested
	* active FTP: server connects to the client using source port 20
	* passive FTP: the client opens the data connection

Options

```
anonymous_enable=YES
```

#### Create an Anonymous drop box

Users can write but cannot read files

``` bash
mkdir /var/ftp/uploads
chmod 0730 /var/ftp/uploads
chown :ftp /var/ftp/uploads
```

`/etc/vsftpd/vsftpd.conf`
```
anon_upload_enable=YES
anon_mkdir_write_enable=YES
chown_uploads=YES
chown_username="USERNAME" # change ownership of anon uploaded files to USER
```
#### SELinux

[[LINUX/SELINUX/SELinux Working Modes]]

Change the SELinux context of the ftp shared folder to allow public writes

``` bash
semanage fcontext -a -t public_content_rw_t "/var/ftp/uploads(/.\*)?"
restorecon -R -v /var/ftp/uploads
```

Get the SELinux boolean that allows anon writes:

``` bash
semanage boolean -l | grep ftp
# ftpd_anon_write                (off  ,  off)  Allow ftpd to anon write
```

Turn the **ftpd_anon_write** switch on and write the change to disk

``` bash
setsebool -P ftpd_anon_write on
```

#### Connecting to FTP

man `lftp`

Packages: 
`ftp` or `lftp`

When connecting with the ftp client, the session is binary mode. This allows to download binary files. To download text files, switch to ascii mode by typing ascii and ENTER

To connect to an FTP server

``` bash
(l)ftp "FTP_SERVER_IP"
```

When connecting with ftp anonymously - user: anonymous, pass: BLANK
`lftp` connects as anonymous user by default

ftp commands:
* `ls`: lists folder contents
* `put` FILE: uploads a file to the current ftp folder
* `get` FILE: downloads a file
* `pwd`: print working dir in the ftp space
* `lpwd`: print local working dir
* `cd`: changes directory
##### Connecting as Local User

`local_enable=YES` must be present in the main conf file
`ftp_home_dir -> on` boolean switch

#### Secure FTP

Configure `vsftp` over TLS

Generate the certificate pair: .crt and .key

`/etc/vsftpd/vsftpd.conf`
```
ssl_enable=YES
allow_anon_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
rsa_cert_file=CERTIFICATE.CRT
rsa_private_key_file=PRIVATE_KEY.key
ssl_ciphers=HIGH

pasv_enable=YES
pasv_max_port=10100
pasv_min_port=10090
```
