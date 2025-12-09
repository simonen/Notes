
`/etc/samba/smb.conf`

- `[global]` for server-wide settings.
- `[homes]` for user home directories.
- `[netlogon]` and `[sysvol]` for domain services in Samba AD/DC setups.
- `[printers]` and `[print$]` for printer sharing.
- `[public]`: for guest-access file shares.

##### Global directives

- `workgroup`
	a collection of computers sharing information on the network neighborhood. Each computer in the workgroup handles its own authentication. Central auth is achieved when one of the hosts becomes a `Primary Domain Controller (PDC)`
	- `workgroup = WORKGROUP`
- `server role`: 
	Specifies the role that the Samba server will play in a network. It is a critical configuration parameter in Samba’s **`smb.conf`** file, and it determines the server's functionality
	- `standalone server`
	- `active directory domain controller` 
	- `classic domain controller`
	- `member server`
	- `netbios backup domain controller`
- `server string`:
	Description of the Samba server
	- `server string = Samba File Server`
- `netbios name`: 
	Local broadcast protocol that handles connection information between computers. Used to match names to IP addresses in WINS servers.
	- `netbios name = DC1`
- `security`:
	- `user`: Samba prompts for a username and password
	- `share`
	- `domain`
	- `ads`
	- `passdb backend`
	- `encrypt passwords`
	- `log level`
	- `interfaces`
	- `hosts allow`
- `hosts `:
	Access control
	- `hosts {allow | deny} = '10.0. means 10.0.0.0/16 or just an IP' [EXCEPT] 'HOST'`

##### Share Directives

Directives
- `[share_name]`
	- `path:`
	- `read only`:
		if set to `no` overrides the share to writable
	- `browsable`
	- `guest ok`
	-  `public`
	- `writable`
	- `write list`
		- `write list = @'GROUP' 'USER1' USER2`
	- `valid users`
		- valid users = +/WORKGROUP/user
	- `create mask`
	- `directory mask`
	- `force user`
		New files and folders will be owned by the specified Unix user. **Forced user must be a member of the group** that has access to the shared directory to avoid permission issues.
	- `force group`
	- `map acl inherit`
	- `nt acl support = no`
		Disables support for Windows-style ACLs on the Samba share.