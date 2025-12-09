
1. Which of the following is not a common approach to prevent against
	brute-force attacks against SSH servers?
	a. Disable X11forwarding
	b. Have SSH listening on a nondefault port
	c. Disable password login
	d. Allow specific users only to log in
2. Which of the following successfully limits SSH server access to users bob and
	lisa only?
	a. LimitUsers bob,lisa
	b. AllowedUsers bob lisa
	c. AllowUsers bob lisa
	d. AllowedUsers bob,lisa
3. Which of the following commands must be used to provide nondefault port
	2022 with the correct SELinux label?
	a. semanage ports -m -t ssh_port_t -p 2022
	b. semanage port -m -t ssh_port_t -p tcp 2022
	c. semanage ports -a -t sshd_port_t -p tcp 2022
	d. semanage port -a -t ssh_port_t -p tcp 2022
4. Which of the following descriptions is correct for the MaxAuthTries option?
	a. After reaching the number of attempts specified here, the account will
	be locked.
	b. This option specifies the maximum number of login attempts. After
	reaching half the number specified here, additional failures are logged.
	c. After reaching the number of attempts specified here, the IP address
	where the login attempts come from is blocked.
	d. The number specified here indicates the maximum amount of login
	attempts per minute.
5. Which log file do you analyze to get information about failed SSH login
	attempts?
	a. /var/log/auth
	b. /var/log/authentication
	c. /var/log/messages
	d. /var/log/secure
6. SSH login in your test environment takes a long time. Which of the following
	options could be most likely responsible for the connection time problems?
	a. UseLogin
	b. GSSAPIAuthentication
	c. UseDNS
	d. TCPKeepAlive
7. Which of the following options is not used to keep SSH connections alive?
	a. TCPKeepAlive
	b. ClientAliveInterval
	c. ClientAliveCountMax
	d. UseDNS
8. Which file on an SSH client computer needs to be added to set the Server-
	KeepAliveInterval for an individual client?
	a. ~/.ssh/ssh_config
	b. ~/.ssh/config
	c. /etc/ssh/config
	d. /etc/ssh/ssh_config
9. Assuming that a passphrase protected public/private key pair has already been
	created, how do you configure your session so that you have to enter the pass-
	phrase once only?
	a. Copy the passphrase to the ~/.ssh/passphrase file.
	b. Run ssh-add /bin/bash, followed by ssh-agent.
	c. Run ssh-agent /bin/bash followed by ssh-add.
	d. This is not possible; you must enter the passphrase each time a connec-
	tion is created
10. Which of the following commands uses the source port 5555 on the local host
	to connect to a remote server called server2.example.com on port 80?
	a. ssh -fNL 5555:localhost:80 root@server2.example.com
	b. ssh -fNL 5555:server2.example.com:80 root@server2.example.com
	c. ssh -fNL 5555:server2.example.com:80 root@localhost
	d. ssh -fNR 5555:localhost:80 root@server2.example.com

