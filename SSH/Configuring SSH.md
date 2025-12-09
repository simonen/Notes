
#### Limiting User Access

Add the **AllowUsers** *USER1 USER2* etc option in **/etc/ssh/sshd_config**

**MaxAuthTries**: for logging purposes. Does not block connections.

#### Other Useful sshd options

Session options

**UseDNS**: makes sure the resolved hostname maps back to the host's IP address
Could be the reason for slow client ssh connection
**MaxSessions**: max number of sessions that can connect to a ssh server from one IP. Default 10
#### Connection Keepalive Options

Server-side configurations

**TCPKeepAlive**: ensures that inactive connections are released
**ClientAliveInterval**: the interval in seconds after which the server sends a packet to the client if it is inactive
**ClientAliveCountMax**: how many of these packets should be sent

**ClientAliveInter** 30 and **ClientAliveCountMax** 10 means inactive connections connections are kept for about 5 minutes

Client-side options to send keepalive traffic to the server Can be set per user in `~/.ssh/config`:
**ServerAliveInterval**
**ServerAliveCountMax**

#### Banners

Banners present a notice to logging users. For default banner message

`/etc/ssh/ssh_banner`
```
************************************************************
*      WARNING: Authorized Users Only!                     *
*  All activities are logged. Unauthorized access is      *
*  prohibited.                                             *
************************************************************
```

Specify the banner in the configuration

`/etc/ssh/sshd_config`
```
Banner /etc/ssh/ssh_banner
```

```
Using username "fikretka".
Pre-authentication banner message from server:
| ************************************************************
| *      WARNING: Authorized Users Only!                     *
| *  All activities are logged. Unauthorized access is      *
| *  prohibited.                                             *
| ************************************************************
|
End of banner message from server
```

Custom banners can be specified per user

`/etc/ssh/sshd_config.d/rule.conf`
```
Match User bob,charlie
    Banner /etc/ssh/restricted_banner
```