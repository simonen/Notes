
1. Which of the following parameters enables you to specify on which
	network card the Postfix mail server should be listening for incoming
	connections?
	**inet_interfaces**
	
1. Which of the following Postfix parameters enables you to specify where mes-
	sages should be forwarded to?
	b. **relayhost**
	
1. Which of the following mail handling roles describes the part of the con-
	figuration that is responsible for communicating with the mail server of the
	recipient?
	c. **MTA**
	
1. Which is the name of the most important Postfix configuration file?
	**/etc/postfix/main.cf**
	
1. Which of the following is not a Postfix process?
	**d. sendmail**
	
1. After fixing a problem that prevented your mail server from reaching the des-
	tination mail server, you notice that messages are not sent immediately again.
	Which command can you use to mark all messages in the mail queue so that
	they will be delivered immediately?
	**c. postqueue -f**
	
1. Which log file contains detailed information about success and failure of mes-
	sage delivery?
	**b. /var/log/maillog**
	
1. Which command enables you to modify Postfix parameters?
	**b. postconf -e**
	
1. While sending a message, you notice that it is not accepted on the recipient
	Postfix mail server. Which of the following parameters should you change to
	enable proper email reception?
	a. inet_interfaces = all
	
10. You want to test your null mail client to verify that it can send email messages
	successfully. Which of the following reflects correct syntax to send a message
	from the command line to user root at server2, where no further action is
	required?
	**a. mail -s hello root@server2 <.**
