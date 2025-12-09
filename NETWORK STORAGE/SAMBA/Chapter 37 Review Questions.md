
1. What is the minimal configuration to put in smb.conf to create a share that
	grants access to the /data directory?
	\[sharename]
	path = /PATH
	
2. How do you configure a share that allows write access to all users that have
	write permissions on the Linux file system?
	writable = yes or read only = no
3. How do you limit write access to a share to members of a specific group only?
	write list @groups
4. Which SELinux Boolean do you use to enable users to access home directories
	on this server through the SMB server?
	use_samba_home_dirs on
5. How do you limit access to a specific share to hosts on the 192.168.10.0/24
	network only?
	host allow = 192.168.10.0/24
6. Which command can you use as root to show a list of all Samba users cur-
	rently defined on your server?
	pdbedit -L
7. What does a user need to do to access a share that is configured as a multiuser
	share, after connecting to that server?
	cifscreds add SMBSERVER
8. How do you mount a Samba share as a multiuser share where user lisa is used
	as the minimal user account?
	credentials file having username=lisa password, mount -o credentials=CREDS,multiuser,ntlmssp,
9. How can you prevent users from seeing Samba mount credentials in the /etc/
	fstab file?
	credentials file with right permissions
10. Which command enables you to list all Samba shares available on a specific
	server?
	smbclient -L SMBSERVER