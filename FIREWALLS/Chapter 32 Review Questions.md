
1. How would you make sure when using firewalld that the iptables service can-
not be started by accident?
	**systemctl mask iptables**
2. Where would you store custom firewalld service files
	**/etc/firewalld/services**
3. Which line would you include in a custom service file to specify TCP port
2022?
	**\<port protocol="tcp" port="22"/>**
4. Which command enables you to list all services currently available on your
server
	\# firewall-cmd --get-services
5. What type of firewalld rules would you use if rich rules do not offer the solu-
tion you need?
	iptables | direct rules
6. Which command enables you to add a rich rule that allows all hosts with a
source IP address coming from the 10.0.0.0/24 network to access ports 7900
up to 7905?
	\# firewall-cmd --zone-ZONE --add-rich-rule='rule family=ipv4 source address=10.0.0.0/24 port port=7900-7950 protocol=tcp accept'
7. Which rich rule enables you to configure a maximum of three packets per
minute to be logged for the http service?
	...service name=http log limit value=3/m accept
8. What is the difference between Network Address Translation and
masquerading?
	NAT binds ports to IP-addresses to forward packets from private to public addresses and back
9. Which command enables you to allow incoming traffic on port 4404 and for-
ward it to the ssh service on IP address 10.0.0.10?
	--add-forward-port=port=4404:proto=tcp:toport=22:toaddr=10.0.0.10 '

10. Which command is used to enable IP masquerading for all packets going out
on the public zone?
	--zone=public --add-masquerade