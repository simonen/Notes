---
tags:
  - postfix
  - mail
  - dns
  - centos
---
#### Roles in Mail Handling

* **MTA** (Mail Transfer Agent): The solution that delivers messages from server to server. Uses the **SMTP** (Simple Message Transfer Protocol). Uses the **MX** (Mail Exchange) record to find the mail server. The core email communication on the internet is between MTA servers only. **Postfix, Sendmail**. Has a very limited set of "commands"
* **MDA** (Message Delivery Agent): Delivers message to the user's inbox once the MTA has received it. By default the msg is delivered to the mailbox of the linux user that exists locally on the server where the mail is coming in **/var/spool/mail/$USER**
* **MUA** (Mail User Agent): The program that the user is using to read and send email. Thunderbird, outlook, etc
* **Null Client**: a machine that does not receive messages, but has all the configuration needed to send mail messages to other hosts. This happens by using a relay host
* **Open relay**: The mail server accepts mail from unauthenticated users from everywhere. This is problematic and could result in the mail server being blacklisted by other servers if it is detected operating as an open relay. Should be regularly tested for open relay behaviour using www.mailradar.com/openrelay

#### Email Transmission Process

Protocols:
**SMTP** (Simple Message Transfer Protocol):  Communication protocol for transmitting mail. Port 25. 
**ESMTP** (Extended SMTP): Has more features and commands. Includeс authentication and encryption
**MX** record: identifies the correct mail server
**POP** Post Office Protocol: downloads messages
**IMAP** Internet Message Access Protocol: online mailbox access

1. user1@server1.example.com uses a mail client to write a message
2. The user sends the message, which is handled by the local Postfix process
3. The local Postfix process is configured as a null client and relays the msg to smtp.example.com, which acts as a relay host
4. smtp.example.com performs a DNS MX record lookup to find that server2.example.org is configured to handle incoming mail for example.org
5. The message is forwarded to server2.example.org
6. server2.example.org delivers the message in the local mailbox of the user in /var/spool/mail/$USER
7. The user uses an MUA which is configured to use IMAP to fetch the message from the local mailbox

#### Email Addresses

The most basic form of an email address is:
* username
* hostname
* domain name or fqdn, separated by '@'
#### DNS and Mail

[[LINUX/NETWORK SERVICES/DNS/BIND#The Zone file]]

Every zone that sends and receives emails has an **MX** record for the mail server responsible for the zone. 

```
MX 'PREFERENCE_NUMBER' 'SERVER_FQDN.'
```

Lower preference means higher priority.

```
                        NS      delphos.olympus.local.
                        MX 10   prometheus.olympus.local
```

It would be nice to have PTR records too in the reverse zone

The clients must be able to properly resolve the mail servers within their zones. Exchanging mail servers must also be able to resolve each other.

``` bash
dig MX 'ZONE' +short
```

```
root@ss1:~# dig valhalla.local -t mx +short
10 prometheus.valhalla.local.
```

Alternatively, the host command can be used

```
host 'SERVER' or 'IP'
```

#### Basic Email Setup

1. Set up the outgoing smtp server
	port 25 must be open
	By default, postfix listens on localhost. Configure **inet_interfaces** to listen to the appropriate address
	Configure myorigin, mydomain
	Test with telnet 
	Configure basic sasl authentication


