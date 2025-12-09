
1. DNS setup
	1. mail server A record
	2. mail server MX record 

##### The Client

Configure the relay host. This sends the mail directly to the host, bypassing MX records.
**\# postconf  -e 'relayhost = \[MAILSERVER.FQDN]'**

Force ipv4
**\# postconf -e 'inet_protocols = ipv4'**

Ensure the client does not receive messages
**\# postconf -e 'inet_interfaces = loopback-only'**
##### The Mail Server

By default the postfix listens on the loopback interface. To allow listening on all ifaces:
**\# postfix -e 'inet_interfaces = all'**

Force ipv4
**\# postconf -e 'inet_protocols = ipv4'**

Open the ports
**\# firewall-cmd --add-service=smtp --permanent ; firewall-cmd --reload**

