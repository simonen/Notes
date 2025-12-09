
1. Which two commands do you need to cache the passphrase that is set on your
	private key?
	**ssh-agent /bin/bash** ; **ssh-add**
2. You want to disallow root login and only allow user lisa to log in to your
	server. How would you do that?
	**AllowUsers lisa**, **PermitRootLogin** no (not needed as AllowUsers allows lisa only)
3. How do you configure your SSH server to listen on two different ports?
	Port NEW_PORT twice; firewall-cmd --add-port=NEW_PORT/TCP
4. What options do you use to create an SSH tunnel where the ssh command
	
	will establish a background connection and will not expect any specific
	command?
	**ssh -fN**
1. When configuring a cache to store the passphrase for your key, where will this
	passphrase be stored?
	**it is stored in RAM**
1. You want to configure local port forwarding on your SSH server, where local
	port 5555 is forwarded to port 80 on server2.example.com. How do you do
	that?
	**ssh -fNL 5555:localhost:80 server2.example.com**
1. Which command enables you to forward a port 8088 on server.somewhere.
	com to the web server listening on your local host?
	ssh -fNR 80:localhost:8088  server.somewhere.com
1. How do you configure SELinux to allow SSH to bind to port 2022?
	semanage port -a -t ssh_port_t -p tcp PORTNUMBER
1. How do you configure the firewall on the SSH server to allow incoming con-
	nections to port 2022?
	\# firewall-cmd --add-port=2022/tcp --permanent ; firewall-cmd --reload
1. To allow for remote port forwarding, which additional setting must be config-
	ured in sshd_config?
	**GatewayPort yes**