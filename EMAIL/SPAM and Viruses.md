
#### Viruses and Spam

* SpamAssassin
* ClamAV

Bayesian spam filter: predicts the likelihood that the presence of a word, phrase or other characteristic in an e-mail address means that the e-mail is spam

Basic Postfix configuration options that affect spam. Restriction lists are triggered when an event occurs:

* `smtpd_client_restrictions`: Restrictions when the clients connect
* `smtpd_helo_restrictions`: Restrictions when the HELO/EHLO command is issued
* `smtpd_sender_restrictions`: Restrictions when the MAIL FROM command is issued
* `smtpd_relay_restrictions`: Restrictions applied to RCPT TO prior to the smtpd_recipient_
	 restrictions
* `smtpd_recipient_restrictions`: Restrictions when the RCPT TO command is issued for spam blocking
* `smtpd_data_restrictions`: Restrictions when the DATA command is issued

Some `helo` restrictions

```
smtpd_helo_required = yes 
# First rule that matches, wins # Kicks in after the mail from: rcpt to: comms
smtpd_helo_restrictions = permit_mynetworks, 
reject_invalid_helo_hostname, 
reject_non_fqdn_helo_hostname, 
reject_unknown_helo_hostname, 
permit
```

To ensure that values passed by the MAIL FROM: command are in the form of a valid e-mail address and that the domain is legit (has an A record) :

```
smtpd_sender_restrictions = reject_non_fqdn_sender, reject_unknown_sender_domain
```

To prevent the e-mail server from becoming a relay

```
smtpd_relay_restrictions = permit_mynetworks, 
permit_sasl_authenticated,
reject_unauth_destination
```

Reject known blacklisted servers and domains

```
smtpd_recipient_restrictions = reject_rbl_client zen.spamhaus.org,
reject_rhsbl_reverse_client dbl.spamhaus.org,
reject_rhsbl_helo dbl.spamhaus.org, reject_rhsbl_sender dbl.spamhaus.org, reject_unauth_
pipelining
```

Apply restrictions to the DATA phase of the SMTP transaction

```
smtpd_data_restrictions = reject_unauth_pipelining
```

**`reject_unauth_pipelining`**: This restriction ensures that **SMTP pipelining** is not used by unauthenticated clients. Pipelining allows multiple SMTP commands to be sent without waiting for the server's response, but it can be exploited by spammers to send messages faster. It essentially ensures that clients must wait for the server’s response before sending additional SMTP commands, which adds a layer of control and can reduce the efficiency of certain types of spam attacks.

Deny e-mail address harvesting by disabling the VRFY command which provides little benefit but poses a risk.

```
disable_vrfy_command = yes
```
#### Blacklists Databases

* **Spamhaus Project**: organization that tracks spam and other cyber threats/ Provides servers that you can query to help reduce spam.
* **rbl** (real-time blackhole list): allow to query the spamhaus database for known bad to and from addresses
* `reject_rbl_client`: queries the DNS block list (DNSBL) at zen.spamhaus.org and rejects positive entries there.
* `reject_rhsbl_reverse_client`: queries dbl.spamhaus.org based on the client, which is domain block list (DBL). It can help reject spam based on several elements (HELO,IP/DNS)
* `reject_rhsbl_helo`: DBL that work on the HELO

##### Popular RBL and RHSBL Providers

- **Spamhaus** (zen.spamhaus.org): One of the most trusted sources for blocking spam, widely used by mail servers globally.
- **SORBS** (dnsbl.sorbs.net): Another popular blacklist provider.
- **Barracuda** (b.barracudacentral.org): Used by the Barracuda email security gateway.
- **SpamCop** (bl.spamcop.net): A blacklist often used for detecting spam sources.

- **RBL/DNSBL**: Focus on blocking IP addresses associated with spam.
- **DBL/RHSBL**: Focus on blocking domains associated with spam or malicious activity.
- Configuring your email server with these lists can significantly reduce spam by rejecting or flagging emails from known malicious sources.

#### SpamAssassin

**Package:**
spamassassin

**Configuration files**
/etc/sysconfig/spamassassin
/etc/mail/spamassassin/local.cf

Spamassassin runs  a daemon **spamd** originally as root user. Create a new dedicated system user that will run the daemon

``` bash
useradd -r -m -s /sbin/nologin spamd
```

Configure the spamd daemon to run as the new system spamd user

> [!NOTE] /etc/sysconfig/spamassassin
> ```
> SPAMDOPTIONS="-d -c -m5 -H -u spamd -g spamd" 
> ```

Enable and start the spamassassin.service

SpamAssassin checks:

* Header tests
* Body phrases 
* Bayesian filters
* Whitelist/blacklist
* Reputational checks 
* Collaborative checks 
* RBLs and DNSRBLs
##### SpamAssassin and Postfix Integration

Mail is analyzed while still inside the MTA. It is passed from Postfix's mail queue to SpamAssassin, then scanned for spam and sent back to Postfix for delivery.

In `/etc/postfix/master.cf`

```
smtp      inet  n       -       n       -       -       smtpd
	-o content_filter=spamassassin
	-o receive_override_options=no_address_mappings
```

`content_filter`: tells Postfix that all e-mail delivered to the smtp service to be sent to a filter called `spamassassin`
`receive_override_options=no_address_mappings`: this prevents an e-mail loop from happening

Define the `spamassassin` filter by creating a new service of type `unix`, which is a `unix socket`. The pipe daemon delivers e-mail to an external command. 

In `/etc/postfix/master.cf`:

```
spamassassin   unix   -   n   n   -   -   pipe
 user=spamd argv=/bin/spamc -e /sbin/sendmail -oi -f ${sender} ${recipient}
```

`spamc`: SpamAssassin client that filters messages
**`-e`**: Tells `spamc` to execute the `/usr/sbin/sendmail` command after processing the mail
**`sendmail -oi -f ${sender} ${recipient}`**: Resends the filtered email using the `sendmail`  command to Postfix to be delivered to the user, preserving the original sender and recipient.
`–oi`: tells the sendmail command to stop processing e-mail when it finds a line with a single  period(.)

#### Testing SpamAssassin

**Package**:
swaks

Repo
epel

`swaks` - Command-line SMTP transaction tester

To send a SASL-authenticated email, containing spam in the sample-spam.txt

``` bash
swaks -tls -a -au 'USER' -ap 123123 -t 'MAIL TO:' -f 'RCPT TO:' --body /usr/share/doc/spamassassin-3.4.0/sample-spam.txt
```

You can also send a test email containing the GTUBE test string (SpamAssassin’s equivalent of the EICAR test for antivirus) to see if SpamAssassin flags it as spam.

Put the following line in the body of a test email:

```
XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X
```

The maillog:

```
smtp postfix/smtpd[18427]: connect from smtp.6circles.cc[192.168.137.2]
smtp postfix/smtpd[18427]: C7D5E1075DF8: client=smtp.6circles.cc[192.168.137.2]
smtp postfix/cleanup[18430]: C7D5E1075DF8: message-id=<2024...64..@smtp.6circles.cc>
smtp postfix/smtpd[18427]: disconnect from smtp.6circles.cc[192.168.137.2] ehlo=2 starttls=1 mail=1 rcpt=1 data=1 quit=1 commands=7
smtp postfix/qmgr[18401]: C7D5E1075DF8: from=<kimchen@6circles.cc>, size=3258, nrcpt=1 (queue active)
smtp spamd[17845]: spamd: connection from ::1 [::1]:55896 to port 783, fd 5
smtp spamd[17845]: spamd: setuid to spamd succeeded
smtp spamd[17845]: spamd: processing message <2024..64..@smtp.6circles.cc> for spamd:978
smtp spamd[17845]: spamd: identified spam (999.0/5.0) for spamd:978 in 0.1 seconds, 
smtp spamd[17845]: spamd: result: Y 999 - ALL_TRUSTED,GTUBE,HTML_MESSAGE scantime=0.1,size=3213,user=spamd,uid=978,required_score=5.0,rhost=::1,raddr=::1,rport=55896,mid=<20240918134931.6491930D5976@smtp.6circles.cc>,autolearn=no autolearn_force=no
smtp spamd[17841]: prefork: child states: II
smtp postfix/pickup[18400]: 088A71075F41: uid=978 from=<kimchen@6circles.cc>
smtp postfix/cleanup[18430]: 088A71075F41: message-id=<2024..64..6@smtp.6circles.cc>
smtp postfix/pipe[18431]: C7D5E1075DF8: to=<kimchen@ohio.cc>, relay=spamassassin, delay=0.24, delays=0.02/0.01/0/0.2, dsn=2.0.0, status=sent (delivered via spamassassin service)
smtp postfix/qmgr[18401]: C7D5E1075DF8: removed
smtp postfix/qmgr[18401]: 088A71075F41: from=<kimchen@6circles.cc>, size=3345, nrcpt=1 (queue active)
smtp postfix/local[18435]: 088A71075F41: to=<kimchen@ohio.cc>, relay=local, delay=0.02, delays=0.01/0.01/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
smtp postfix/qmgr[18401]: 088A71075F41: removed
```

**`smtp postfix/pipe[2037]`**: This shows that Postfix used the `pipe` transport method to deliver the email.
**`dsn=2.0.0`**: Delivery status notification (DSN) code `2.0.0` indicates that the message was successfully delivered.
**`relay=spamassassin`**: Postfix passed the email to the `spamassassin` service, which was configured in the `master.cf` file as a content filter.

#### Filtering Mail with Sieve

**Package**:
dovecot-pigeonhole (CentOS)

**Repo**:
Base, appstream


> Custom email folders work with the [[gitea/LINUX 1/EMAIL/Mail Delivery Agents (MDA)#IMAP]] protocol
> Create custom folders [[gitea/LINUX 1/EMAIL/Mail Delivery Agents (MDA)#Mail Location]]

Sieve works with [[gitea/LINUX 1/EMAIL/Postfix (MTA)#LMTP (Local Mail Transport Protocol)]] -> how to configure LMTP

Enable plugin support at protocol level. For [Quota](https://doc.dovecot.org/2.3/configuration_manual/quota_plugin/#quota) and [Sieve](https://doc.dovecot.org/2.3/configuration_manual/sieve/#sieve) here:

> [!NOTE] /etc/dovecot/conf.d/20-lmtp.conf
> ```
>protocol lmtp {
  postmaster_address = postmaster@domainname   # required
  mail_plugins = quota sieve
}
> ```

Test if sieve is enabled

```
telnet 127.0.0.1 4190
Connected to 127.0.0.1.
Escape character is '^]'.
"IMPLEMENTATION" "Dovecot Pigeonhole"
"SIEVE" "fileinto"...
```

To apply rules on a per-user basis, put the following in `~/.dovecot.sieve`

```
require ["fileinto"];

if address "From" "gligana@6circles.cc" {
        fileinto "INBOX/Huinq";
   }

if address :contains "From" "ohio.cc" {
        fileinto "INBOX/Ohio";
   }

if header :contains "X-Spam-Flag" "YES" {
        fileinto "Spam";
   }
```

```
smtp spamd[1248]: prefork: child states: II
smtp dovecot: lmtp(4544): Connect from local
smtp dovecot: lmtp(kimchen): uKP+CbMu62bAEQAAVGon3A: sieve: msgid=<2024..D40C@smtp.6circles.cc>: stored mail into mailbox 'Spam'
smtp dovecot: lmtp(4544): Disconnect from local: Successful quit
smtp postfix/lmtp[4543]: 17133303D40C: to=<kimchen@6circles.cc>, relay=smtp.6circles.cc[private/dovecot-lmtp], delay=0.09, dsn=2.0.0, status=sent (250 2.0.0 <kimchen@6circles.cc> uKP+CbMu62bAEQAAVGon3A Saved)
smtp postfix/qmgr[4393]: 17133303D40C: removed
```

To apply sieve filters globally, in case of non-existent local filters:

> [!NOTE] /etc/dovecot/conf.d/90-sieve.conf
> ```
plugin {
   sieve_default = /var/lib/dovecot/sieve/default.sieve
}

Create the default.sieve file in the location and set the dovecot folder ownership to dovecot:dovecot, o+x permissions, put the default sieve scripts into the file.

#### Antivirus ClamAV

Packages:
**clamav**: End-user tools for the Clam Antivirus scanner
**clamd**: The Clam AntiVirus Daemon
**clamav-freshclam** : Auto-updater for the Clam Antivirus scanner data-files
**clamav-milter** : Milter module for the Clam Antivirus scanner
**clamav-data** : Virus signature data for the Clam Antivirus scanner
**libmilter.so.** : if dependency not found by **clamav-milter**
https://rpmfind.net/linux/rpm2html/search.php?query=libmilter.so.1.0()(64bit)

`/etc/mail/clamav-milter.conf`
```
MilterSocket /run/clamav-milter/clamav-milter.socket
MilterSocketGroup virusgroup
MilterSocketMode 660
User clamilt
ClamdSocket unix:/var/run/clamd.scan/clamd.sock
OnInfected Accept
AddHeader Add
ReportHostname mail.ohio.cc
LogSyslog yes
LogFacility LOG_MAIL
LogVerbose yes
LogInfected Basic
LogClean Basic
```

`/etc/clamd.d/scan.conf`
```
LocalSocket /run/clamd.scan/clamd.sock
User clamscan
```

`/etc/postfix/main.cf`
```
milter_default_action = accept
smtpd_milters = unix:/run/clamav-milter/clamav-milter.socket
```

Now that Postfix has been interfaced with the `clamav-milter.socket`, the `/usr/clamav-milter` dir and `/usr/clamav-milter/clamav-milter.socket` must be accessible by the postfix user, who must be member to the `virusgroup` group, as specified in the .conf files.

``` bash
usermod -aG virusgroup postfix
chown -R clamilt:virusgroup /usr/clamav-milter
```

To test if the user has the necessary permission on the socket

``` bash
runuser -u postfix -- nc -Uv /run/clamav-milter/clamav-milter.socket
# Ncat: Connected to /run/clamav-milter/clamav-milter.socket.
```

Start the services

```
systemctl enable --now clamd@scan clamav-milter
```

Send a test email with the following string to be treated as a virus signature.

``` 
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
```

Sep 26 17:11:34 smtp.6circles.cc clamav-milter[3051]: Clean message from <kimchen@6circles.cc> to <kimchen@6circles.cc>

```
clamav-milter[3051]: Message from <kimchen@6circles.cc> to <kimchen@6circles.cc> infected by Eicar-Signature
```

The altered mail header

```
Received: from [192.168.137.1] (gateway [192.168.137.1])
        by smtp.6circles.cc (Postfix) with ESMTPSA id 42B2A304A208
        for <kimchen@6circles.cc>; Thu, 26 Sep 2024 17:13:20 +0300 (EEST)
X-Virus-Status: Infected (Eicar-Signature)
X-Virus-Scanned: clamav-milter 0.103.11 at smtp.6circles.cc
```

For testing purpose, sort infected messages in another folder

```
require ["fileinto"]; 
if header :contains "X-Virus-Status" "" { 
	fileinto "Spam"; 
	} 

if not header :contains "X-Virus-Status" "Clean" { 
	fileinto "Virus"; 
	}
```

In production environment, quarantine infected messages

`OnInfected Quarantine` in clamav-milter.conf

#### SPF (Sender Policy Framework)

A mail server can be configured to use whatever sender address. We need to ensure that other mail servers do not receive unauthorized mail on behalf of our domain.

- The SPF record is a text record listing servers that are approved to send email from your domain. Each domain should have 1 SPF record.
- Receiving servers check the SPF record to verify that email from your domain is from authorized servers.
* Prevents Email Spoofing: when attackers send emails that appear to come from a trusted domain (e.g., phishing attacks). 

Anatomy of the SPF (TXT) DNS record
https://spfwizard.com/: tool for generating SPF records

```
v='version' mx [a:host] [include:domain] [proto:net/mask or ip/32] [-~?]all(fail_mech)
```

``` bash
# DNS TXT RECORD
v=spf1 include:_spf.google.com inclide:otherprovider ~all
```

* `v=spf1`: Specifies the SPF version.
* `include:_spf.google.com` This allows Google’s mail servers to send emails on behalf of your domain. The `include` mechanism references the SPF record at `_spf.google.com`, which is managed by Google and contains all the IP addresses that Google uses to send emails.
* Fail mechanisms:
	* `~all`: Soft fail** . It means that if an email is sent from an unauthorized server (not included in the SPF record), it should be marked as **suspicious** but still accepted by the recipient's server. 
    - **`-all`**: **Hard fail** — Strict enforcement. Emails from unauthorized servers are rejected.
    - **`?all`**: **Neutral** — No specific action is suggested.

Most basic SPF record. Do to not allow any other servers to use your domain. Means that only the servers in the MX records are authorized to use the domain.

```
 'DOMAIN.'    IN    TXT    "v=spf1 mx -all"
```

Therefore, in order to use other mail providers, like yahoo, google, mailchip, other webmail, they need to be included in the SPF record.

If a zone has multiple MX records, they can be packed into a single SPF TXT record which can be referred to as a source in other zones, like \_spf.google.com contains a list of multiple mail servers.

```
domain.com.
                   IN    MX 10   mx1.
			       IN    MX 10   mx2.
                   IN    TXT     "v=spf1 include:_spf.domain.com ~all"
_spf.domain.com.    IN    TXT     "v=spf1 a:mx1 a:mx2 ~all"
```

Package:
**pypolicyd-spf**: SPF Policy Server for Postfix

Postfix Config

`/etc/postfix/main.cf`
```
smtpd_recipient_restrictions = .... 
	reject_unauth_destination 
	check_policy_service unix:private/policyd-spf 
	reject_rbl_client zen.spamhaus.org ... 
policyd-spf_time_limit = 3600
```

`/etc/postfix/master.cf`
```
policyd-spf unix - n n - 0 spawn 
	user=nobody argv=/usr/libexec/postfix/policyd-spf
```

Legit

```
mail policyd-spf[8200]: prepend Received-SPF: Pass (sender SPF authorized) identity=helo; client-ip=192.168.137.2; helo=smtp.6circles.cc; envelope-from=kimchen@6circles.cc; receiver=<UNKNOWN>
```

Illegit 

```
Sep 26 23:01:43 mail policyd-spf[8910]: prepend Received-SPF: Softfail (mailfrom) identity=mailfrom; client-ip=192.168.137.5; helo=mail.york.cc; envelope-from=cicina@6circles.cc; receiver=<UNKNOWN>
```

#### DKIM (DomainKeys Identified Mail)

DKIM is an email authentication method designed to detect forged sender addresses (spoofing).


Package: opendkim.x86_64, opendkim-tools
Repo: epel; crb (for dependencies)
Configuration: /etc/opendkim.conf
Other conf:/ etc/opendkim/
Sample config files: /usr/share/doc/opendkim/

[OpenDKIM Configuration File](http://www.opendkim.org/opendkim.conf.5.html)

##### How it works

**Signing Outgoing Emails**: When a domain sends an email, the mail server that handles the email signs the message using a cryptographic private key that belongs to the domain. This signature is added to the email headers in the form of a `DKIM-Signature`.

**DKIM Signature**: This signature contains information like the domain, the selector (which identifies the key pair used), and a hash of the message body and headers. The signature proves that the email has not been altered after it was sent by the originating domain.

**Public Key in DNS**: The public key corresponding to the private key is published in the domain's DNS as a TXT record. This allows receiving mail servers to retrieve the public key and verify that the email was genuinely sent by the domain and that its content hasn't been tampered with during transit.

**Verification by Receiving Server**: When the receiving mail server gets the email, it checks the `DKIM-Signature` header and retrieves the public key from DNS. It uses this public key to validate the signature against the email headers and content. If the signature matches, the email is considered authentic.

`/etc/opendkim.conf`
```
Mode    sv
Syslog  yes
SyslogSuccess   yes
LogWhy  yes
UserID  opendkim:opendkim
Socket local:/run/opendkim/opendkim.sock
Umask   002
SoftwareHeader  yes
Canonicalization        relaxed/relaxed
Selector        default
MinimumKeyBits  2048
KeyTable        refile:/etc/opendkim/KeyTable
SigningTable    refile:/etc/opendkim/SigningTable
OversignHeaders From,Subject,Date
SignHeaders From,Subject,Date
```

```
mkdir -m 0750 /etc/opendkim/keys
```

Generate the keys

man opendkim-genkey

``` bash
opendkim-genkey -D /etc/opendkim/keys/ohio.net/ -d ohio.net -r -s default 
```



`/etc/opendkim/KeyTable`
```
default._domainkey.ohio.net ohio.net:default:/etc/opendkim/keys/ohio.net/default.private
```

``` bash
chown -R opendkim:opendkim /etc/opendkim/keys
chmod -R 0640 /etc/opendkim/keys/'DOMAIN'/
```

Connect with Postfix

`/etc/postfix/main.cf`
```
milter_default_action = accept
smtpd_milters = unix:/run/opendkim/opendkim.sock
```

The `opendkim.service` has the `opendkim` user and group hardcoded, which makes the socket inaccessible by postfix. Change the group to `postfix`

```
[Unit]
Description=DomainKeys Identified Mail (DKIM) Milter
After=network.target nss-lookup.target syslog.target
[Service]
Type=simple
EnvironmentFile=-/etc/sysconfig/opendkim
User=opendkim
Group=postfix
```

Get the TXT record from the `/keys/DOMAIN/default.txt` file and put it in the DNS zone file

```
default._domainkey.ohio.net.    IN      TXT     ( "v=DKIM1; k=rsa; "
"p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC/q4equP7EvbejeVpzUsAxW11ipnDV61vob9jMgX.........umnkSAQAB" )
```

Test the verification process

``` bash
opendkim-testkey -d 'DOMAIN' -s 'SELECTOR' -vvv
#opendkim-testkey: using default configfile /etc/opendkim.conf
#opendkim-testkey: checking key 'default._domainkey.6circles.cc'
#opendkim-testkey: key OK
```

```
opendkim[8001]: 24C3B30FD3FC: DKIM-Signature field added (s=default, d=6circles.cc)
```

```
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/simple; d=example.com; s=default;
h=from:subject:to:date:message-id;
bh=wHt0EClK4gqjJh3kt+FPP00bZsM=;
b=abcde...base64-encoded-signature...;
```

- `v=1`: DKIM version.
- `a=rsa-sha256`: The algorithm used to generate the signature (RSA with SHA-256).
- `d=example.com`: The domain that is signing the email.
- `s=default`: The selector used to locate the public key in DNS.
- `h=from:subject:to:date:message-id`: The headers that are signed.
- `bh=wHt0EClK4gqjJh3kt+FPP00bZsM=`: The hash of the email body.
- `b=abcde...`: The actual signature of the message.

```
opendkim[10498]: C4B3430FDEC9: DKIM verification successful
```

```
X-Mozilla-Status: 0001
X-Mozilla-Status2: 00000000
Return-Path: <gligana@ohio.net>
X-Original-To: kimchen@6circles.cc
Delivered-To: kimchen@6circles.cc
Received: by smtp.6circles.cc (Postfix, from userid 997)
	id 5D46930FDEC6; Fri, 27 Sep 2024 20:23:56 +0300 (EEST)
X-Spam-Status: No, score=-0.1 required=5.0 tests=ALL_TRUSTED,DKIM_SIGNED,
	DKIM_VALID,DKIM_VALID_AU,SPF_SOFTFAIL,TVD_SPACE_RATIO autolearn=no
	autolearn_force=no version=3.4.0
Received-SPF: Pass (sender SPF authorized) identity=mailfrom; client-ip=192.168.137.9; helo=mail.ohio.cc; envelope-from=gligana@ohio.net; receiver=kimchen@6circles.cc 
DKIM-Filter: OpenDKIM Filter v2.11.0 smtp.6circles.cc 16E0830FDEC5
Authentication-Results: smtp.6circles.cc;
	dkim=pass (1024-bit key) header.d=ohio.net header.i=@ohio.net header.b="YiPBIUUn"
Received: from mail.ohio.cc (mail.ohio.cc [192.168.137.9])
	(using TLSv1.2 with cipher ADH-AES256-GCM-SHA384 (256/256 bits))
	(No client certificate requested)
	by smtp.6circles.cc (Postfix) with ESMTPS id 16E0830FDEC5
	for <kimchen@6circles.cc>; Fri, 27 Sep 2024 20:23:55 +0300 (EEST)
Received-SPF: Softfail (mailfrom) identity=mailfrom; client-ip=192.168.137.1; helo=[192.168.137.1]; envelope-from=gligana@ohio.net; receiver=<UNKNOWN> 
DKIM-Filter: OpenDKIM Filter v2.11.0 mail.ohio.cc 208083166DF9
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=ohio.net; s=default;
	t=1727457839; bh=kZIohMkvnp2oxwUEoqSRrhYwuM7qxflS0KHr2OFcySE=;
	h=Date:From:Subject:From:Subject:Date;
	b=YiPBIUUnjmtSLJok8zOs3vIfSUW7jdj1nXydUpjmbHg8jUXzJ8n7LeoUYU/lx8oR0
	 nTN69DW8NcDm5rooTR+V3JDRQ/qw3Et/Jbd9hhBdiWilgXC2Uv8UBt+NDyPu3xJGUp
	 aOPUcGOlUB59/UqoQ5OxbLXv2MZrJtHApz7eGTvI=
Message-ID: <bb21c272-c133-4925-ac74-5ccc860ca58c@ohio.net>
Date: Fri, 27 Sep 2024 20:23:56 +0300
To: kimchen@6circles.cc
From: Gliganska Anska <gligana@ohio.net>
Subject: DKIM

SIKISH
```