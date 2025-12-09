
1. You want to connect an NFS client to a Kerberized NFS server. The mount
	can be made, but users do not get access to the share. Which of the following
	is the most likely explanation?
	
2. Which statement about the keytab file is correct?
3. What is the name of the file where you create NFS exports?
	**/etc/exports**
4. Which statement about user access to shares is not true?
5. Which port should be open in the firewall to allow access to an NFSv4 server
	**a) 2049**
6. Which command enables you to check the availability of NFS shares behind a
	firewall that has the nfs-server service enabled, assuming that you are using a
	standard NFSv4 setup?
	a. showmount -e nfsserver
7. You want to make sure that it can never happen that NFS clients have read/
	write access to shares. Which of the following is the most secure method to
	guarantee that?
	c. Make nfsnobody the owner of all shares and set permission mode to 500
8. Which NFS version do you need if you want SELinux context labels on the
	NFS server to be transparent on the NFS client?
	b. 4 or higher
9. Which mount option do you need in /etc/fstab to mount NFS shares on an
	NFS client?
	c. _netdev
10. Which file must exist on the NFS server to enable Kerberized NFS?
