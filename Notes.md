
Documentation tools

Trillium
Bookstack
Mkdocs
Docusaurus

```
"You do not rise to the level of your goals. You fall to the level of your systems." by James Clear ^quote-of-the-day
```

[[2023-01-01#^^quote-of-the-day]]

User-space program - a tool used by the end user to configure some component of the internal operating system. This component could be Netfilter, which is part of the kernel, and the user-space program to configure it -> iptables.


#### SSH

`-D`: establish dynamic port forwarding. Create a SOCKS proxy on the specified local port on the local machine and forward connections through the jump hosts to the target host. After executing the command, applications can be configured to use SOCKS proxy on `localhost:PORT`

To pre-define the connection in the `config` file

> [!NOTE]+  ~/.ssh/config
> ```
> Host "JUMP_HOST"
> 	Hostname "JH_HOSTNAME | IP"
> 	User "USER"
> 	IdentityFile "PRIVATE_KEY"
> 	IdentitiesOnly yes
> 
> Host "TARGET_HOST"
>     Hostname "TH_HOSTNAME | IP"
>     ForwardAgent yes
>     DynamicForward "LOCAL_PORT"
>     { LocalForward "LOCAL_PORT" "TARGET_HOST":22 }
>     ProxyJump "JUMP_HOST"
> ```

`DynamicForward 1080`: This creates a SOCKS proxy on port 1080 on local machine
`ProxyJump`: Connect to the jump host first and establish connection to target host through it
`ForwardAgent` : Enables SSH agent forwarding, which allows authentication credentials to be forwarded to the remote machine