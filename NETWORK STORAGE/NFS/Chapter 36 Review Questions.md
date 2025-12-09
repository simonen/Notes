
1. What is the name of the configuration file that contains NFS shares?
2. /etc/exports
3. Which services must be started on an NFS client to enable the mounting of
	nothing
1. Which services must be started on an NFS server to offer Kerberos security?
2. nfs-server
3. Which ports need to be open in the firewall to allow full access to the NFS
	server?
	2049, 111, 20048
1. You want to create an NFS share where the web server can read documents as
	well. Write access is not necessary. Which SELinux context label enables you
	to set access as tight as possible?
	public_content_t
1. Which option must be used in /etc/fstab to make sure NFS shares can be
	mounted automatically when rebooting?
	\_netdev for older versions
1. You want to set up an NFS server for an environment where only the lightest
	form of Kerberos protection on the share is supported. Which sec= option
	should you use on the export?
	sec=krb5
1. Your NFS server does not have access to the same user accounts as your NFS
	clients. Which approach is recommended to set security?
	all_squash -> users have same permissions as **nfsnobody**
1. Which file needs to be modified on the NFS server to allow transparency of
	SELinux context type to the client?
	the BULLSHIT file
10. Which NFS version offers transparency of SELinux context types set on the
	server to the NFS client as well?
	4.2