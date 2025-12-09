
#### Configuring a Mail Delivery Agent

Sending and receiving messages directly with postfix isn't much practical. To be able to use a proper Mail User Agent (MUA), or simply put, a mail client, an MDA must be set up first.
MDAs work with the POP3 / IMAP protocols to read incoming messages from the MTA (postfix). 

#### Dovecot

www.doc.dovecot.org - Official documentation

Dovecot is a secure IMAP and POP3 server

Packages: 
**dovecot**

Ports **110**: POP3, **995**: POPS3, **143**: IMAP, **993**: IMAPS

Main config file
- `/etc/dovecot/dovecot.conf`

Various config files
- `/etc/dovecot/conf.d`

##### Basic Dovecot Configuration

https://doc.dovecot.org/2.3/configuration_manual/authentication/

Authentication mechanisms are controlled by

`/etc/dovecot/conf.d/10-auth.conf`
```
disable_plaintext_auth = no
auth_mechanisms = plain login
!include auth-system.conf.ext
```

PLAIN should be used only when paired with TLS. 
`auth-system.conf.ext` : specify authentication service (store). This is for Unix users authentication using the passwd db.

`/etc/dovecot/conf.d/auth-system.conf.ext`
```
passdb {
	driver = pam
	#args = dovecot
}
userdb {
	driver = passwd
}
```

Dovecot looks for a PAM service definition 'dovecot' in `/etc/pam.d/` dir.

`/etc/pam.d/dovecot`
```
auth       required     pam_nologin.so
auth       include      password-auth
account    include      password-auth
session    include      password-auth
```

`/etc/dovecot/conf.d/10-ssl.conf`
```
ssl = yes # enabled but not required
```

Test the dovecot configuration. Works similarly to the `postconf` command

``` bash
doveconf
```

#### Mail Location and Mailboxes

`/etc/dovecot/conf.d/10-mail.conf`
```
mail_location = mbox:~/mbox:INBOX=/var/mail/%u
mail_access_groups = mail
```

To list the folders of a mail user

``` bash
doveadm mailbox list -u 'USER'
INBOX
```

To create new mail folder

``` bash
doveadm mailbox create -u 'USER' 'FOLDER'
```

Or just create an empty file in /home/user/mail/MAILBOX_NAME

``` bash
touch /home/'USER'/mail/'MAILBOX'
chmod 0600 /home/'USER'/mail/'MAILBOX'
```
#### Namespaces

> The IMAP protocol syncs mailbox folders with the client automatically

Change the mailbox hierarchy separator. Adding mailboxes here will create them by default for each user.

`/etc/dovecot/conf.d/10-mail.conf` 
```
namespace inbox {
	inbox = yes
	separator = /
	prefix = INBOX/

	mailbox 'CUSTOM_MAILBOX' {
		auto = subscribe
	}
}
```

`auto = subscribe` = automatically show the custom mailbox folder under **Inbox** in the mail client

```bash
doveadm mailbox list -u kimchen
```

```
INBOX
INBOX/Spam
INBOX/Junk
INBOX/Ohio
INBOX/Trash
INBOX/Sent
INBOX/Franken
```

As the INBOX/ prefix is specified, new folders must be referred to by their full path now, as in INBOX/Custom_Folder (in sieve scripts for example)
##### POP3

Port 110

Post Office Protocol downloads emails from the server (MTA). It is configured as an incoming connection server.

The Postfix is an outgoing connection server.

Enable POP3

`/etc/dovecot/dovecot.conf`
```
protocols = pop3 imap lmtp
```

Allow only one POP3 session to run simultaneously for the same user.

`/etc/dovecot/conf.d/20-pop3.conf`
```
pop3_lock_session = yes
```

Test the POP3 connection with telnet

``` bash
telnet 'POP3SERVER' 110
```

```
Trying 10.0.3.89...
Connected to 10.0.3.89.
Escape character is '^]'.
+OK Dovecot ready.
```

Authenticate with a local account. 

```
...
USER kimchen
+OK
PASS "PASSWORD"
+OK Logged in.
```

List emails 

```
LIST
+OK 12 messages:
1 620
2 630
```

To read an email 

``` telnet
RETR 'MSG NUMBER'

RETR 1
+OK 620 octets
Return-Path: <root@prometheus.olympus.local>
X-Original-To: kimchen
Delivered-To: kimchen@prometheus.olympus.local
Received: by prometheus.olympus.local (Postfix, from userid 0)
        id C54D6303B6F0; Fri, 12 Apr 2024 21:43:38 +0300 (EEST)
Date: Fri, 12 Apr 2024 21:43:38 +0300
To: kimchen@prometheus.olympus.local
Subject: Meeting
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Message-Id: <20240412184338.C54D6303B6F0@prometheus.olympus.local>
From: root@prometheus.olympus.local (root)

Hi Kimchen!
Dai Raketi, dai pari
```

The POP3 server is working and accepting connections and able to access the inbox of a user. To configure a client, use the POP3 server as incoming server.

##### IMAP

IMAP keeps messages on the server and syncs them with the client. Incoming server.
Port: 143

`No Starch Press - Book of IMAP`

Test IMAP connection with telnet

Make sure IMAP is enabled and port 143 is allowed through the firewall

`/etc/dovecot/dovecot.conf`
```
protocols = imap lmtp
```

```
telnet IMAP_SERVER 143
```

```
[kimchen@pelisterka ~]$ telnet 10.0.3.89 143
Trying 10.0.3.89...
Connected to 10.0.3.89.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE STARTTLS AUTH=PLAIN] Dovecot ready.
```

> Depending on how IMAP is configured to handle logins, 'user' or 'user'@'domain' might be the required format

Authenticate 

``` bash
'command tag' LOGIN 'USER' 'PASS'
```

command tag can by an arbitrary string

```
kim LOGIN kimchen pass
kim OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY SPECIAL-USE] Logged in
```

Basic IMAP commands to fetch mails

``` bash
'tag1' LIST "" "*"
# * LIST (\NoInferiors \UnMarked) "/" "Sent Items"
# * LIST (\HasNoChildren) "/" INBOX
'tag1' EXAMINE INBOX
# * FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
# * OK [PERMANENTFLAGS ()] Read-only mailbox.
# * 47 EXISTS
'tag1' FETCH 'MAIL NUMBER' BODY[]
```

##### IMAPS

Port 993

To encrypt incoming messages, the IMAPS is used

`/etc/dovecot/conf.d/10-ssl.conf`
```
ssl_cert = </etc/pki/dovecot/certs/dovecot.pem
ssl_key = </etc/pki/dovecot/private/dovecot.pem
```

Test IMAPS over TLS

``` bash
openssl s_client -connect 'IMAP_SERVER':993
```

After showing the certificate info, a standard telnet session opens.

```
dovecot: imap-login: Login: user=<kimchen>, method=PLAIN, rip=10.0.3.82, lip=10.0.3.126, TLS, session=<6ETuFBMWxtYKAANS>
```

In case of problems, disable weak ciphers

```bash
openssl s_client -connect smtp.6circles.cc:993 -cipher 'HIGH:!aNULL:!MD5' -tls1_2 -CAfile /path/to/certificate.pem
```
#### Dovecot Administration with doveadm

man doveadm

Test a virtual user authentication

``` bash
doveadm -v auth test user@domain.com
```

```
passdb: postuser@ohio.org auth succeeded
extra fields:
  user=postuser@ohio.org
```

To list supported hashes by `doveadm`

``` bash
doveadm pw -l
```

To hash a string

``` bash
doveadm pw -s 'HASH_TYPE' -p "STRING"
```

Show logged dovecot users

``` bash
doveadm who
```

```
username         # proto (pids) (ips)
gligana@ohio.org 1 imap  (3300) (192.168.137.1)
```

User lookup

``` bash
doveadm user 'USER'@'DOMAIN'
```

```
field   value
uid     5000
gid     5000
home    /home/kibuna
mail    maildir:/var/mail/vhosts/ohio.net/kibuna/Maildir
```

#### Troubleshooting

Authentication debugging
- `/etc/dovecot/conf.d/10-logging.conf`

