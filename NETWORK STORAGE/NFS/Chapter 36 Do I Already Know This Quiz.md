
1. You want to connect an NFS client to a Kerberized NFS server. The mount
	can be made, but users do not get access to the share. Which of the following
	is the most likely explanation?
	c. The user did not get a Kerberos ticket.
2. Which statement about the keytab file is correct?
	a. A Kerberized NFS server needs to have a keytab file containing the Ker-
	beros credentials of the NFS server
3. What is the name of the file where you create NFS exports
	c. /etc/exports
4. Which statement about user access to shares is not true?

5. Which port should be open in the firewall to allow access to an NFSv4 server?
	a. 2049
6. Which command enables you to check the availability of NFS shares behind a
	firewall that has the nfs-server service enabled, assuming that you are using a
	standard NFSv4 setup?
	
7. You want to make sure that it can never happen that NFS clients have read/
	write access to shares. Which of the following is the most secure method to
	guarantee that?
	d. Set the SELinux Boolean nfs_export_all_rw=of
8. Which NFS version do you need if you want SELinux context labels on the
	NFS server to be transparent on the NFS client?
	c. 4.2 or higher
9. Which mount option do you need in /etc/fstab to mount NFS shares on an
	NFS client?
	
10. Which file must exist on the NFS server to enable Kerberized NFS?
	d. /etc/krb5.keytab

