1. Where in /proc would you look to find process specific information about the
environment variables the process is using?
	~~/proc/PID/status~~ | /proc/PID/environ
2. Which /proc file would you check to find out information about currently
available disk devices?
	~~/proc/devices~~ | /proc/partitions
3. Where in /proc would you look for detailed information about memory usage?
	/proc/meminfo
4. Which command enables you to enable IP packet forwarding on your server?
	sysctl -w net.ipv4.ip_forward=1
5. How would you permanently enable IP packet forwarding on your server?
	$ "echo net.ipv4.ip_forward=1" > /etc/sysctl.d/ip_forward.conf
6. Which command shows all kernel tunables
	$ sysctl -a
7. Which tunable can you use to tell your server not to react to any ping
requests?
	net.ipv4.icmp_ignore_all=1
8. Which service is started on boot to read sysctl tunables?
	systemd-sysctl
9. Which sysctl tunable sets the hostname of this server?
	/proc/kernel/hostname
	sysctl -w kernel.hostname=HOSTNAME
1. Which command enables you to read and activate settings you have just made
to the /etc/sysctl.d/net.conf file
	~~sysctl --system~~ | sysctl -p /etc/sysctl.d/net.conf
