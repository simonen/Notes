---
tags:
  - email
  - dovecot
  - IMAP
  - POP3
  - openssl
---
Packages 
**mailx**
**s-nail**

[Comparison of email clients - Wikipedia](https://en.wikipedia.org/wiki/Comparison_of_email_clients#Authentication_support) and their authentication mechanisms

#### Sending Messages from Server to Server

To send a simple email message directly to another server or the same if domain not specified.

``` bash
mail -s "SUBJECT" 'USER'
'MESSAGE'
.
EOT
```

The . on the last line terminates the message body and sends the message

To check the inbox

```
mail
```

To reply to a message

```
& r 'N'
```

To use a mail server as a relay host. On a postfix client machine 

``` bash
postconf -e "relayhost = 'MAIL SERVER'"
```


#### Use an External SMTP Server

The per-user configuration is located in ~/.mailrc. System-wide located in /etc/s-nail.rc. This works with SASL authentication configured on the smtp server.

> [!NOTE] ~/.mailrc
> ```
> set v15-compat
unset mta
set mta=smtp://SMTP_USER:PASS@SMTP_SERVER
set from=YOUR_EMAIL_ADDRESS
set realname="YOUR FULL NAME"
> ```

To list messages from a different folder (in s-nail)

``` bash
folder /home/user/mail/'MAILBOX'
& folder /home/kimchen/mail/Spam
"/home/kimchen/mail/Spam": 2 messages
>   1 KIm Chen Un           Wed Sep 18 22:49  89/3440  "[SPAM] asegatka"
    2 Gligana ANA           Thu Sep 19 17:08  91/3521  "[SPAM] Spam TEZT"
```
##### Send a Test Email

``` bash
echo "This is a test email" | s-nail -s "Test Subject" RECEIPIENT_MAIL
```

#### Webmail

Web-based mail clients

Package
squirrel, apache



