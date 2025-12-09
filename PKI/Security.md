
#### Forward Secrecy

**Forward Secrecy** (also known as **Perfect Forward Secrecy, or PFS**) is a property of cryptographic protocols that ensures the confidentiality of past communications even if the server's private key is compromised in the future. Every connection is individually protected, using a different key.

**Key Concepts of Forward Secrecy**

1. **Session Keys**
	* Session keys are temporary keys used to encrypt the data exchanged in a single session
	* Forward Secrecy ensures that each session has its own unique session key, which is not derived from the server's long-term private key.
2. **Ephemeral Keys**:
	* FS relies on the use of *ephemeral keys*. These are short-lived keys that are generated for a single session and discarded afterward.
	* The Diffie-Hellman (DH) or Elliptic Curve Diffie-Hellman (ECDH) key exchange mechanisms are commonly used to generate ephemeral keys.
* 

**Anti-Replay**: Sequence numbers disallow packets to be re-sent multiple times.
**Non-Repudiation**: Sender cannot later deny sending a message
#### Attacks

##### Sidejacking

Retrieval of session tokens over an unencrypted traffic. Cannot be used against 100% encrypted websites.

**Session tokens**
In web applications, as soon as a user connects to a web site a new session is created. Each session is assigned a secret token (also known as a session ID), which is used to identify ownership. If the attacker finds out the token of an authenticated session, she can gain full access to the web site under the identity of the victim.

#### Cookie Stealing

Even if web traffic is encrypted, an attacker can wait for an unencrypted HTTP transaction to execute attacks involving cookies. Secure and insecure cookies live in the same namespace.

#### Mitigation

**Deploy HTTP Strict Transport Security with subdomain coverage**
	*HTTP Strict Transport Security* (HSTS): enforces encryption on the hostname for which it is enabled. Optionally, can enforce encryption on all subdomains.
**Validate Cookie Integrity**
	The best defense against cookie injection is integrity validation. Achieved with HMAC

#### SSL Stripping

SSL or *HTTPS stripping* attacks exploit the fact that most users begin browsing sessions on a plaintext portion of the website or type addresses without explicitly specifying the *https://* prefix. The initial plaintext traffic is vulnerable.

#### Certificate Warnings

**Misconfigured Virtual Hosting**
	Plaintext sites should not be put on the same IP address as other sites that use encryption on port 443
**Insufficient name coverage**
	If you're hosting a website at *www.example.com*, the certificate should include that name and also *example.com*. Other domain names pointing to the same site, should also be included.
**Self-signed certificates and private CAs**
	Self-signed certs are not appropriate for public websites
**Certificates used by appliances**
**Expired Certificates**
**Misconfiguration**
	For a certificate to be trusted, each user agent is required to establish a chain of trust from the server certificate to a trusted root. Servers are required to provide the full chain, minus the trusted root.

Use HSTS to suppress certificate warnings. If there is an issue with a certificate on an HSTS site, all failures are fatal and cannot be overridden.

**Mixed Content**
	Web sites mixing plaintext and secure pages
	*HTTP Strict Transport Security*
	*Content Security Policy*

> Redirect always to the same entry point on the secure website. ???

#### Protocol Downgrade Attacks

Forward secrecy is lost 

#### Truncation Attack

The attacker is able to prematurely terminate a secure connection.
Exploited applications told their users that they had logged off before they actually did. With access to the device, the attacker could use the application session.

#### Cookie Cutting

Controlling TLS record length allows splitting of HTTP request or response headers.
One application of HTTP response header truncation is known as *cookie cutter* attack; it can be used to downgrade secure cookies into plain cookies.

#### TLS Session Cache Sharing

Servers must issue new keys

#### Insecure Renegotiation

#### User Redirection

If a redirection point on a secure site that leads to a plaintext web site, the TLS connection is effectively downgraded. The attacker can establish full control over the victims browsing

If not using client certificates, disable renegotiation.
#### BEAST

Targets CBC encryption in TLS 1.0. The attack reduces CBC to ECB mode which is insecure.

#### Padding Oracle

#### Session Caching

Session cache sharing (especially among unrelated applications) has a security impact - larger attack surface. Best to avoid. If using tickets, every server must use a different key.

Non-critical servers can cache sessions for up to a day.
Critical servers - an hour

**SSL Offloading and Reverse Proxies**: Terminating encryption at a separate architecture level. This is dangerous because traffic between the proxy and application is not encrypted.

If using Amazon Elastic Load Balancer (or other external infra providers), configure it at the TCP level and terminate TLS at your nodes.

#### Pinning

Allows to specify exactly which CAs are allowed to issue certificates for the domain names.
Currently, only possible via the proprietary mechanism embedded in Chrome.

**Content Security Policy (CSP)**: Reject plaintext links present in a web page

#### HTTP Strict Transport Security (HSTS)

A protocol standard that describes a strict approach to the handling of web site encryption.

Designed to mitigate TLS weaknesses:
* No way of knowing if a site supports TLS
* Tolerance of certificate problems
* Mixed content issues
* Cookie security

Web sites that support HSTS emit *Strict-Transport-Security* header on all encrypted HTTP responses:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

To revoke HSTS, set the max-age to 0

It is best to configure HSTS closest to the user. On the reverse proxy for example.
By default, HSTS is enabled on the hostname that emits the HSTS header. Must be activated on all hostnames in the domain (i.e., www.example.com, example.com). Enumerate all paths that lead to the web site and add HSTS to all of them.

Modify your sites so that each hostname submits at least one request to the root domain name. This will ensure that HSTS is fully enabled on the entire domain namespace, even if your users do not visit the root domain name directly

For extra points, if you have a reverse proxy in front of your web site(s), configure your
HSTS policy centrally at the proxy level. To prevent header injection vulnerabilities
from being used to bypass HSTS, delete any HSTS response headers set by the backend web servers.

Unauthenticated NTP syncing could be insecure


#### Content Security Policy

The Main goal of CSP is defense against *cross-site-scripting* (XSS) attacks.

This policy allows resources to be loaded only from its own origin by default, but allows images to be loaded from any URI, plugin content only from the specified CDN addresses, and external scripts only from *script.example.com*

`Response Header Containing CSP`
```
Content-Security-Policy: default-src 'self'; img-src *; object-src *.cdn.example.com; script-src scripts.example.com
```

CSP are not persistent; they are used only on pages that reference them.

```
Content-Security-Policy: default-src https: 'unsafe-inline' 'unsafe-eval';    connect-src https: wss:
```

`default-src https`: Content is allowed from anywhere if done via https

#### Pinning

*Pinning* is a security technique that can be used to associate a service with one or more cryptographic identities such as certificates and public keys. Domain owners can specify (pin) the CAs that are allowed to issue certificates for their domain names.

The best element to pin is the `SubjectPublicKeyInfo` (SPKI) field of an X.509 certificate.

#### DANE

*DNS-Based Authentication of Named Entities*, is a proposed standard to provide associations between domain names and one or more cryptographic entities. Domain name owners can use the DNS as a separate channel to distribute information needed for robust TLS authentication. Relies on DNSSEC

`TLSA Resource Record`: Carry certificate associations.

`TLSA record`
```
_PORT._PROTOCOL.FQDN. IN TLSA (
	CERTIFICATE_USAGE SELECTOR MATCHING_TYPE SHA256_HASH )
```

```
_443._tcp.www.example.com. IN TLSA (
	0 1 1 d2abde240d7cd3ee6b4b28c54df034b9
		7983a1d16e8a410e4561cb106618e971 )
```

Association is created between a domain name and the public key of a CA (Certificate Usage is 0), identified by the `SubjectPublicKeyInfo` field (Selector is 1) via its hex-encoded SHA256 hash (Matching Type is 1)

#### Certification Authority Authorization

Proposes a new way for domain name owners to authorize CAs to issue certificates for their domain names.

`CAA Resource Record`
```
certs.example.com    CAA O issue "ca.example.net"
```

`issue`: A property tag. Instructions to CAs. `;` tag forbids certificate issuance.