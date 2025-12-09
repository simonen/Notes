
1. Which parameter do you need to change to make sure that Postfix listens on
	other IP addresses than just the loopback address
	**inet_interfaces** 
	
2. After sending messages from the null client that you have configured, you
	notice that all messages contain the server name in the sender mail address.
	Which parameter do you need to change in the Postfix configuration to make
	sure that it contains the domain name only?
	**myorigin**

3. You want your Postfix mail server to accept email messages for three differ-
	ent domains. Which parameter do you need to change to tell your mail server
	which domains it is responsible for?
	mydestination
	
4. You want to make sure that your Postfix server defaults to IPv4 and does not
	use IPv6 at all. Which parameter do you need to change to accomplish this?
	inet_protocols = ipv4

5. You want to set up your null client to use smtp.example.com as the relay host.
	You also want to make sure that no DNS MX record lookup is done, but the
	message is sent directly to the relay host. Which line should be included in the
	postfix configuration file?
	**relayhost =** \[MAIL_SERVER]
	
6. Which command enables you to get an overview of all Postfix parameters that
	are currently defined with the value they are using
	**postconf**
	
7. You have just tried to send a message which did not succeed. You have identi-
	fied why the message could not be sent but notice that the message still has
	not been delivered. How can you schedule immediate message delivery?
	**postqueue -f**
	
8. Which file contains information about the message sending process?
	**/var/log/maillogs**
1. Which SELinux contexts/Booleans must be changed to guarantee successful
	Postfix operations?
	no need to set SELinux context on a default setup
	
10. Which service must be added to the firewall configuration to guarantee suc-
	cessful email delivery?
	**smtp**