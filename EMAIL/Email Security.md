---
tags:
  - mail
  - postfix
  - smtp
---
#### Securing Postfix

- `smtpd_client_restrictions = permit_sasl_authenticated, reject`
554 5.7.1 <unknown[192.168.137.10]>: Client host rejected: Access denied
- `smtpd_sender_restrictions = permit_sasl_authenticated, reject`
554 5.7.1 <4u4un>: Sender address rejected: Access denied
- `smtpd_recipient_restrictions = permit_sasl_authenticated, reject`
554 5.7.1 <kibuna@ohio.net>: Recipient address rejected: Access denied

Rules act much like ACL's - first to match, wins.

#### `smtpd_client_restrictions`

[Client Restrictions Options](https://www.postfix.org/postconf.5.html#smtpd_client_restrictions)

The `smtpd_client_restrictions` parameter in Postfix controls access to the SMTP server based on the client's IP address or hostname. This parameter allows you to define rules that decide whether a client (the connecting email sender) is allowed to communicate with the server.

```
smtpd_client_restrictions =
    permit_mynetworks,
    reject_unauth_pipelining
    reject_rbl_client zen.spamhaus.org,
    reject_unknown_client_hostname,
    reject_plaintext_session
    permit_sasl_authenticated,
    reject
```

Common Restrictions:

1. **`permit_mynetworks`**: Allows connections from clients that are listed in the `mynetworks` parameter (e.g., your internal network or trusted IP addresses).
2. **`permit_sasl_authenticated`**: Allows clients that have successfully authenticated using SASL.
3. **`reject_rbl_client <dnsbl>`**: Rejects clients based on their IP address being listed in a DNS-based blocklist (DNSBL). In the example above, `zen.spamhaus.org` is a popular blocklist.
4. **`reject_unknown_client_hostname`**: Rejects clients that don't have a resolvable hostname via DNS reverse lookup (PTR record).
5. **`reject`**: The `reject` directive denies access to any client that doesn't meet the previous conditions.

This parameter is usually used to block unwanted or potentially malicious email senders based on their IP address or domain. It is applied early in the SMTP transaction and can block clients before they send the email content.

Common Use Cases for `smtpd_client_restrictions`

1. **Preventing Spam**: By using DNSBLs like Spamhaus or Barracuda, you can reject clients known for sending spam.
2. **Restricting to Internal Users**: Only allow clients from specific networks (via `mynetworks`) or authenticated users.
3. **Blocking Clients with Bad Hostnames**: Clients that fail to resolve or use non-compliant hostnames can be blocked.

#### `smtpd_sender_restrictions` 

in Postfix defines a set of rules for controlling which sender addresses are allowed or denied during the SMTP `MAIL FROM` stage of the email transaction. It helps prevent unwanted emails by restricting who can send emails through your server.

##### Common Use Cases for `smtpd_sender_restrictions`

1. **Preventing Forged Senders**: You can use this to ensure that only specific senders (e.g., senders from your domain) are allowed to send emails via your server.
2. **Rejecting Invalid Senders**: Block senders from non-existent domains or who use fake email addresses.
3. **Allowing Authenticated Users**: Allow users who have authenticated (e.g., via SASL) to bypass certain restrictions.

```
smtpd_sender_restrictions = 
    permit_mynetworks,
    reject_non_fqdn_sender,
    reject_unknown_sender_domain,
    reject_unauth_pipelining,
    reject_unlisted_sender,
    reject_sender_login_mismatch
    permit_sasl_authenticated,
```

- **`permit_mynetworks`**: Allow sending from IP addresses listed in the `mynetworks` parameter.
- **`permit_sasl_authenticated`**: Allow authenticated users (e.g., those who log in via SASL authentication).
- **`reject_non_fqdn_sender`**: Reject senders that don't use a fully qualified domain name (FQDN).
- **`reject_unknown_sender_domain`**: Reject senders whose domain can't be resolved via DNS.
- **`reject_unauth_pipelining`**: Prevent sending commands without waiting for responses, a technique often used in spam.
- **`reject_unlisted_sender`**: Reject senders that don't exist in your Postfix system (useful for virtual domains).
- **`reject_sender_login_mismatch`**: Reject emails where the sender address doesn’t match the login used for authentication.

To enforce the envelop sender address to be owned by the SASL-authenticated user, a lookup table is needed

`/etc/postfix/main.cf`
```
smtpd_sender_login_maps = hash:/etc/postfix/ce_senders
smtpd_recipient_restrictions = 
 reject_sender_login_mismatch, 
 permit_sasl_authenticated
 reject
```

`/etc/postfix/ce_senders`
```
(envelop address)   (SASL login name)
kibuna               kibuna@ohio.net
kibuna@ohio.net      kibuna@ohio.net
@ohio.net            kibuna gligana fikretka@ohio.net kimchen@ohio.net
helpdesk@ohio.net    kibuna@ohio.net tikretka@ohio.net
```

### `smtpd_recipient_restrictions`

The `smtpd_recipient_restrictions` parameter in Postfix controls access to the SMTP server based on the recipient address. It allows you to define rules that determine whether an incoming email is accepted or rejected based on the recipient's email address or domain.

```
smtpd_recipient_restrictions =
    permit_mynetworks,
    permit_sasl_authenticated,
    reject_unauth_destination,
    reject_non_fqdn_recipient,
    reject_unknown_recipient_domain,
    reject_rbl_client zen.spamhaus.org,
    reject
```

- **`permit_mynetworks`**: Allows emails sent from clients in the `mynetworks` parameter (trusted internal networks).
- **`permit_sasl_authenticated`**: Allows emails from clients that have successfully authenticated via SASL.
- **`reject_unauth_destination`**: Rejects mail to recipients that are not hosted on your server and for which your server is not a relay. This is essential for preventing open relays (i.e., your server being used to send mail to arbitrary destinations).
- **`reject_non_fqdn_recipient`**: Rejects recipients that are not fully qualified domain names (FQDN), which means the domain part of the recipient address must be complete (e.g., `user@domain.com`).
- **`reject_unknown_recipient_domain`**: Rejects recipients with a domain that cannot be resolved by DNS (e.g., if the domain part of the recipient's email address does not exist or doesn't resolve).
- **`reject_rbl_client <dnsbl>`**: Rejects the message if the client's IP is listed in a DNS-based blocklist (like `zen.spamhaus.org`).
- **`reject`**: Rejects any recipients that do not meet any of the prior permit conditions.

`reject_unauth_destination`: ... Postfix rejects email relaying to domains it is not responsible for (i.e, not in mydestinations or virtual_domains)

- **Local recipients**: Email addresses within the domains that the server is configured to handle (as specified by the `mydestination`, `virtual_mailbox_domains`, `relay_domains`, etc.).
- **Authorized relay domains**: Domains that the server is explicitly configured to relay mail for, based on your settings.

This means the mail server will **reject emails** that:

- Try to relay through your server to a third-party domain unless the user is authenticated.
- Are addressed to domains your server isn't configured to handle.
##### Common Use Cases for `smtpd_recipient_restrictions`:

1. **Preventing Open Relay**: The most important function of this parameter is to ensure the server does not allow unauthenticated users to relay mail to external destinations. This is accomplished using `reject_unauth_destination`.
2. **Ensuring Validity of Recipients**: By rejecting invalid recipient domains (with `reject_unknown_recipient_domain` and `reject_non_fqdn_recipient`), you can filter out invalid or misconfigured emails.
3. **Blocking Spam Based on IP**: Using blocklists like Spamhaus (`reject_rbl_client`), you can reject messages coming from known spam sources.
4. **Securing External Users**: External users who need to relay mail through your server (e.g., employees working remotely) must authenticate (SASL) to bypass restrictions.

##### Key Directive Explanation:

- **`permit_mynetworks`**: Allows mail from trusted networks defined in `mynetworks`.
- **`permit_sasl_authenticated`**: Allows mail from authenticated users (required if you want to let external users relay mail).
- **`reject_unauth_destination`**: This is essential to prevent your server from being an open relay. It ensures the server only accepts mail for domains that it is responsible for (i.e., `mydestination`, `relay_domains`, or `virtual_alias_domains`).
- **`reject_non_fqdn_recipient`**: Prevents mail addressed to incomplete recipient addresses like `user@domain` instead of `user@domain.com`.
- **`reject_unknown_recipient_domain`**: Rejects mail if the recipient's domain is not valid (cannot be resolved via DNS).

#### Relaying

To prevent an SMTP server from becoming an open relay

`/etc/postfix/main.cf`
```
smtpd_recipient_restrictions = reject_unauth_destination
```

To allow relays to specific domains

```
relay_domanins = 'DOMAIN1' 'DOMAIN_N'
```

Postfix smtp (client) authentication against the relay.

`/etc/postfix/main.cf`
```
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_mechanism_filter = plain, login
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = may
smtp_use_tls = yes
```
### Example Scenario

1. **Legitimate Email**:
    
    - The mail server accepts emails addressed to its own domain (e.g., `@example.com`), as specified by `mydestination`, because the domain is a local recipient.
    - It also allows authenticated users (via SMTP AUTH) to send emails to external domains.
2. **Unauthorized Relay Attempt**:
    
    - If someone tries to send an email from a random external domain (e.g., `spammer.com`) to another external domain (e.g., `victim.com`) through your mail server, `reject_unauth_destination` will **block** it because your mail server isn't configured to relay mail for these domains unless authentication is provided.

Without this setting, the server will act as an `open relay`

``` bash
# The sender rejects unauthenticated outgoing connection
warning: unknown[10.0.3.80]: SASL LOGIN authentication failed: UGFzc3dvcmQ6
```

Incoming unauthenticated connections are rejected now. As seen on the client server that does not enforce SMTP authentication

``` bash
# Sender server receives message from client server that connection has been rejected
 status=deferred (host prometheus.olympus.local[10.0.3.89] said: 450 4.7.1 Client host rejected: cannot find your hostname, [10.0.3.126] (in reply to RCPT TO command))
```

#### SMTP TLS Encryption

Control TLS debug info

```
smtp(d)_tls_loglevel = [1 2 3]
```

**Port 25 – Opportunistic TLS (STARTTLS)**

Port 25 is the standard port for **SMTP**. On this port, encryption is not required by default, but can be negotiated using **STARTTLS**. STARTTLS upgrades a plain-text connection to an encrypted one, but only after the connection is already established. This allows the connection to start unencrypted and switch to TLS when both parties agree.

When you see Postfix working on port 25, it's likely using **opportunistic encryption** with STARTTLS. This is normal behavior for SMTP communications.

- **No Encryption**: By default, an SMTP connection on port 25 starts as plain text.
- **STARTTLS**: If the client supports it, the connection can be upgraded to encrypted TLS.

Postfix encrypts email only if the other servers support it.

Force TLS on port 25. Block Non-TLS connections

`/etc/postfix/main.cf`
```
smtpd_tls_security_level = encrypt
```

```
smtp postfix/smtpd[2117]: initializing the server-side TLS engine
client mandated to use TLS: 530 5.7.0 Must issue a STARTTLS command first
```
##### Port 587 - (Submission)

Port 587 is the **submission** port, should be the default port. Uses TLS

To configure Postfix for the submission port, ensure the following lines are in `master.cf`:

`/etc/postfix/master.cf`
```
submission inet n       -       n       -       -       smtpd
```

##### **Port 465 – SMTPS (Implicit TLS)**

Port 465 is used for **SMTPS**, which is SMTP over SSL. With this, the connection is encrypted from the very start. If you want Postfix to offer full-time encryption from the start (not just after a STARTTLS negotiation), you need to configure Postfix to listen on port 465 as well.

To configure postfix to use SMTPS on port 465

`/etc/postfix/master.cf`
```
smtps inet n - n - - smtpd
```

