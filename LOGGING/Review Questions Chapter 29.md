1. Which **rsyslogd** module should you use to enable reception of **journald** log
messages?
	$ModLoad imjournal
2. What is the name of the deprecated module that can be used to enable receiv-
ing journald messages in rsyslogd?
	**imuxsock**
3. To make sure that the legacy method for receiving messages from journald in
rsyslogd is not used, which additional parameter should be used?
	**$OmitLocalLogging on**
4. Which configuration file contains settings that allow you to further tune the
working of journald?
	**/etc/systemd/journald.conf**
5. Which parameter takes care of forwarding messages from journald to
rsyslogd?
	**ForwardToSyslog=yes**
6. Which rsyslogd module can you use to include messages from a specific
non-rsyslog generated log file?
	imfile
1. Which **rsyslogd** module do you need to use to forward messages to a MariaDB
database?
	$ModLoad **ommysql**
8. Which two lines do you need to include in rsyslog.conf to allow the current
log server to receive messages over TCP?
	$ModLoad imtcp
	$InputTCPServerRun 514
9. How do you configure the local firewalld firewall to allow for log message
reception over TCP port 514
	\# firewall-cmd --add-port=514/tcp --permanent
10. Which line do you include on a server where you want to forward rsyslog mes-
sages to logserver.example.com, which is configured to receive messages using
UDP and the default port?
	\*.* @logserver.example.com:514
