> **THE SYSTEM CLOCK MUST BE SYNCED FOR CERTIFICATES TO SHOW AS VALID!**
#### Internet PKI

The goal of PKI is to enable secure communication between parties that have never met before. The current model relies on third parties called `certification authorities (CAs)` to issue certificates we trust.

* **Subscriber**: The end entity that wishes to provide secure services, which require certificate
* **Registration Authority**: Performs necessary identity validations before requesting a CA to issue a certificate.
* **Certification Authority**: The party we trust to issue certificates that confirm subscriber's identities. Required to provide up-to-date revocation information online
* **Relying Party**: The certificate consumer. Technically, web browsers or other programs that perform certificate validation. They do this by operating *root trust stores* that contain the ultimately trusted certificates (*trust anchors*) of some CAs.

In PKI *trust* means that a certificate can be validated by a CA we have in the trust store.

For a TLS connection to be trusted, every client must perform two basic checks:
* The (server) certificate applies to the intended hostname
* All chain certificates (including the leaf) must be checked to see that:
	* They have not expired
	* Their signatures are valid
* An intermediate certificate might need to satisfy:
	* Can be used to sign other certificates for the intended purposes
	* Can be used to sign (or not) other CA certificates
	* Can be used to sign the hostname in the leaf certificate
#### Certificates

A certificate is a public key with additional fields (validity, subject, etc.)

A digital certificate binds a public key to an entity (e.g., a user, device, or server) and is issued by a trusted Certificate Authority (CA).

A certificate is a digital document that contains:
* *a public key*
* information about the entity associated with it
* *digital signature* from the certificate issuer.

`Abstract Syntax Notation One (ASN.1)`: Is a standard interface description language for defining data structures that can be serialized and de-serialized in a cross-platform, language-neutral way. Widely used in protocols like LDAP, SNMP, X.509 certificates, and more.

#### Certificate Types

* **Domain Validation (DV) certificates**: OID 2.23.140.1.2.1
	* Automatic issuance
	* Cheaper
	* Same level of encryption as EV
* **Extended Validation (EV) certificates***: OID 2.23.140.1.1
	* Establish a link between a domain name and the legal entity behind it. (Individuals cannot obtain EV certificates)
	* The identity of the domain owner is known and encoded in the certificate
	* Manual verification process makes forgery extremely difficult.
	* Practical benefits are minimal
	* Modern browsers no longer emphasize the presence of EV certificates.


| Feature          | Domain Validation (DV)               | Extended Validation (EV)                            |
| ---------------- | ------------------------------------ | --------------------------------------------------- |
| Validation Level | Verifies domain control              | Verifies legal and operational entity               |
| Issuance Time    | Minutes to hours                     | Days to weeks                                       |
| Cost             | Lower cost                           | Higher cost                                         |
| Browser Display  | Padlock                              | Padlock + organization name                         |
| User Assurance   | Low (confirms domain control)        | High (confirms legitimate entity)                   |
| Use case         | Blogs, personal sites, non-sensitive | E-commerce, financial services, large organizations |
|                  |                                      |                                                     |


DV and EV certificates provide the same level of encryption and secure the data transmitted between the client and the server. Primary difference lies in the level of trust they provide regarding the identity of the entity operating the website.

#### Certificate Validation

A certificate proves that the end entity owns the corresponding private key.

1. **Is the certificate valid**? Signature verification
	1. The client checks the certificate signature by hashing the certificate data using the stated signature algorithm, decrypts the signature using the CA's public key. Both hashes must match. This proves that the certificate data hash had been signed with CA's private key.
2. Is the server the true owner of the certificate?
	2. Client generates a random value.
	3. Encrypts it with the server's public key
	4. Server decrypts it with its private key
	5. Value is used to generate a session key
3. *Expiration Check*
4. **Revocation Check**: The client checks if any certificate has been revoked via a CRL or *Online Certificate Status Protocol*

##### Certificate Hostnames

The main purpose of a certificate is to establish trust for the appropriate hostnames.

> Unrelated websites and sites operated by multiple teams should not share a certificate
#### Certificate Fields

* **Version**: Three certificate versions: 1, 2, and 3, encoded as values 0, 1, and 2. Version 1 supports only basic fields, version 2 adds unique identifiers, version 3 adds extensions
* **Serial Number**: Uniquely identify a certificate issued by a CA. Must be unpredictable and contain at least 20 bits of entropy.
* **Signature Algorithm**: The algorithm used to sign the certificate.
* **Issuer**: The *distinguished name* (DN) of the issuing authority (the CA).
* **Validity**: Starting and ending date.
* **Subject**: The distinguished name (DN) of the entity associated with the public key for which the certificate is issued. Self-signed certificates have the same DN in their *Subject* and *Issuer* fields. The *Subject* field is deprecated in favor of the *Subject Alternative Name* extension to specify multiple hostnames.
* **Subject Public key info**: Contains the public key
* **Extensions**: Optional fields used in X.509v3 certificates (i.e., SAN)

#### Extensions

Certificate extensions are additional fields that allow for more flexibility and functionality beyond basic elements. Introduced in version 3. (x.509v3)

A X.509v3 certificate supports two main categories of extensions:
5. **Critical Extensions**: These must be understood and processed by any system using the certificate. The system must reject the certificate if it doesn't recognize a critical extension in it.
6. **Non-Critical Extensions**: Optional, systems can ignore them if they don't recognize them.

[Standard Extensions](https://docs.openssl.org/master/man5/x509v3_config/#description)

* `Basic Constraint`: Specifies whether the certificate is a CA certificate (which can sign other certificates) and the maximum number of intermediate certificates that can be issued from it via the *path length constraint* field. Non-CA certs have this extension omitted or set to 'FALSE'. This extension is critical. 

```
X509v3 Basic Constraints: critical
    CA:TRUE, pathlen:0
```

* `Key Usage (KU) `: Restrict what a certificate can be used for. If present, only the listed uses are allowed. If not present, no restrictions.
* `Extended Key Usage (EKU)`: This extension allows for more fine-grained control than the `Key Usage` extension.

[Key Usage values](https://docs.openssl.org/master/man5/x509v3_config/#key-usage)
This example is typical for a web server certificate, which does not allow for code signing
[Extended Key Usage values](https://docs.openssl.org/master/man5/x509v3_config/#extended-key-usage)

```
X509v3 Key Usage: critical
	Digital Signature, Key Encipherment
X509v3 Extended Key Usage:
	TLS Web Server Authentication, TLS Web Client Authentication
```

* `CRL Distribution Points`: Lists the addresses where the CA's Certificate Revocation List (CRL) information can be found.

```
X509v3 CRL Distribution Points: 
	Full Name:
		URI:http://crl.starfieldtech.com/sfs3-20.crl
```

* `Certificate Policies`: This extension is used to indicate the policy under which the certificate was issued
	* `OID 2.23.140.1.2.1`: This unique object identifier indicates a domain-validated certificate.
	* `CPS`: Certificate Policy Statement. Document

```
X509v3 Certificate Policies:
	Policy: 2.23.140.1.2.1 
	Policy: 1.3.6.1.4.1.44947.1.1.1 
		CPS: http://cps.letsencrypt.org
```

* `Authority Information Access (AIA)`: Contains two important pieces of information. 
	* Lists the address of the CA's '*Online Certificate Status Protocol*' (OCSP) responder, which can be used to check for certificate revocation in real time.
	* CA Issuers: Next certificate in the chain

```
Authority Information Access:
	OCSP - URI:http://ocsp.int-x3.letsencrypt.org 
	CA Issuers - URI:http://cert.int-x3.letsencrypt.org/
```

* `Subject Key Identifier`:  A unique identifier for the public key in the certificate, derived from the public key itself. Allows easy identification of the key used without referencing the full public key. CAs must include this extension and use the same identifier in the AKI extension of all its issued certificates.
* `Authority Key Identifier`: The value of the AKI must match the value specified in the SKI in the issuing certificate (CA). Can be used to identify the parent certificate.

```
X509v3 Subject Key Identifier:
	A1:EC:11:C6:E1:E8:F7:E6:98:85:FA:9A:53:F8:B8:F1:D6:88:F9:A3
X509v3 Authority Key Identifier: (The SKI of the issuing CA)
	keyid:A8:4A:6A:63:04:7D:DD:BA:E6:D1:39:B7:A6:45:65:EF:F3:A8:EC:A1 
```

* `Subject Alternative Name`: Used to list all hostnames for which the certificate is valid. If not present, clients fall back to using the information provided by the CN, which is part of the Subject field. If present, CN field is ignored.

```
X509v3 Subject Alternative Name: 
	DNS:www.feistyduck.com, DNS:feistyduck.com
```

* `Certificate Transparency (CT)`: Recent. Used to carry proof of logging to various public CT logs. *Signed Certificate Timestamps (SCTs)* : Used to recognize a certificate as valid

```
CT Precertificate SCTs:
	Signed Certificate Timestamp:
		Version : v1 (0x0) 
		Log ID : 5E:A7:73:F9:DF:56:C...:84:A1:12:12:84:18:75:96:81:71:45:58
	Timestamp : Aug 3 00:10:45.300 2020 GMT
	Extensions: none 
	Signature : ecdsa-with-SHA256 2:5C:E3...:0C:27:54:9B: DA:E1:3F:0F:8E:09:60
```

* `Name Constraints`: Limits the scope of the CA's ability to issue certificates to specific namespaces, such as certain domains or IP ranges. This indicates that only certificates with a DNS name under `ohio.cc/org` would be valid. 

Types of names that can be constrained:
* DNS Names
* IP Addresses
* Email Addresses
* URI
* Directory names

```
X509v3 Name Constraints:
                Permitted:
                  DNS:ohio.cc
                  DNS:ohio.org
                Excluded:
                  IP:0.0.0.0/0.0.0.0
                  IP:0:0:0:0:0:0:0:0/0:0:0:0:0:0:0:0
```

* `Server Name Indication (SNI)`: This extension allows a client (a web browser) to specify the hostname of the server it is trying to connect to so the server can present the correct certificate. This is important for servers that host multiple websites (virtual hosts) on a single IP address.
* `ESNI`: Encrypted SNI. Encrypts the hostname in the ClientHello message.
#### Certificate Chains

A **Certificate Chain** or a **certification path** is an ordered list of certificates that enables trust between a client and a server. It links a specific certificate (usually a server's certificate) to a **trusted root certificate authority (CA)** through a series of **intermediate certificates**. Each server must provide a *chain of certificates* that leads to a trusted root.

Extra certificates (e.g., the root, which is not needed) slow down the handshake.
##### Components of a Certificate Chain

7. **End-Entity Certificate** (Leaf Certificate):
	* This is the certificate issued to the entity (such as a website, app or service)
8. **Intermediate Certificate**:
	* These certificates sit between the end-entity certificate and the root CA. They are issued by the root CA or another intermediate CA and are used to establish a chain of trust to the root CA. It is used to sign the end-entity (leaf) certificates.
	* Intermediate CA
9. **Root Certificate**
	* This is the top-level certificate issued but a trusted certificate authority. It is self-signed and trusted by the client. It is the anchor of the trust chain. Should be used to issue subordinate CAs only, not leaf certificates. Must be kept offline.

`Example Certificate Chain`
```
End-entity Certificate: www.example.com
	Issued by: Intermediate CA 1

Intermediate CA 1
	Issued by: Intermediate CA 2

Intermediate CA 2
	Issued by: Root CA

Root CA
	Self-signed
```

The issuer of a certificate must match the subject of the next in the chain.

```
depth=2 C=US, O=Google Trust Services LLC, CN=GTS Root R4 (Root CA)
verify return:1
depth=1 C=US, O=Google Trust Services, CN=WE1 (Intermediate CA)
verify return:1
depth=0 CN=hardenize.com (Leaf, end-entity, server)
verify return:1
---
Certificate chain
 0 s:CN=hardenize.com
   i:C=US, O=Google Trust Services, CN=WE1
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: ecdsa-with-SHA256
   v:NotBefore: Dec  4 05:21:35 2024 GMT; NotAfter: Mar  4 05:21:34 2025 GMT
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
 1 s:C=US, O=Google Trust Services, CN=WE1
   i:C=US, O=Google Trust Services LLC, CN=GTS Root R4
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: ecdsa-with-SHA384
   v:NotBefore: Dec 13 09:00:00 2023 GMT; NotAfter: Feb 20 14:00:00 2029 GMT
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
 2 s:C=US, O=Google Trust Services LLC, CN=GTS Root R4
   i:C=BE, O=GlobalSign nv-sa, OU=Root CA, CN=GlobalSign Root CA
   a:PKEY: id-ecPublicKey, 384 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 15 03:43:21 2023 GMT; NotAfter: Jan 28 00:00:42 2028 GMT
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
```

Potential Issues

* **Invalid Certificates**: If any certificate in the chain is expired, revoked or otherwise invalid, the verification process will fail
* **Incomplete Chain**: If the server does not provide the full chain, the client may not be able to verify the chain properly.
* **Mismatched Domain**: If the leaf certificate does not match the domain the client is trying to access, connection will be rejected.

#### Relying Parties

An entity or system that depends on the authenticity  and integrity of a digital certificate to make trust-based decisions. 

For RPs to be able to validate subscriber certificates, they must keep a collection of root CA certificates they trust. (in their root stores)

#### Certificate Lifecycle

The certificate lifecycle begins when a subscriber prepares a *Certificate Signing Request* (CSR) and submits it to the CA. CSR carries the relevant public key and demonstrates ownership of the corresponding private key.

**Domain validation** 
*  *Domain validated* (DV) certificates are issued based on proof of control over a domain name. Issuance is automated and very quick.
**Organization validation**
* *Organization validated (OV)* certificates require identity and authenticity verification. 
**Extended Validation**
* *Extended validation* (EV) certificates require very strict identity and authenticity verification. Introduced to address the lack of consistency in OV certificates. Might take weeks to obtain an EV certificate.

#### Revocation

Certificates are revoked when the associated private keys are compromised or no longer needed. There are two standards for certificate revocation:

* **Certificate Revocation List**
	A CRL is a list of all serial numbers belonging to revoked certificates that have not yet expired. CAs maintain one or more such lists. Every certificate should contain the location of the corresponding CRL in the *CRL Distribution Points* extension. Slow.
* **Online Certificate Status Protocol (OCSP)**
	*Online Certificate Status Protocol* (OCSP) allow relying parties to obtain the revocation status of a single certificate. OCSP servers are known as *OCSP responders.* The location of the CA's OCSP responder is encoded in the *Authority Information Access* extension.
	* OCSP responses are valid for up to 10 days; up to 12 months for intermediates.
	* **OCSP Stapling**
		*OCSP Stapling* is a technique where the server takes on the responsibility of querying the OCSP responder for its own certificate status and then 'staples' the status (a signed OCSP response) to the TLS handshake. This allows the client to verify the certificate's status without directly contacting the OCSP server.

Baseline Requirements allow CRL and OCSP information to stay valid for up to 10 days (12 months for intermediate certificates). It takes at least 10 days for the revocation information to fully propagate.
Soft-fail policy in browsers will obtain revocation information but ignore failures.

#### OCSP Stapling (Online Certificate Status Protocol Stapling)

OCSP is a protocol used by the client to check the revocation status of a certificate in real-time.
The client queries the CA's OCSP server to confirm if a certificate has been revoked.

Each client must contact the OCSP server.
Reveals to the CA which website a client is visiting
If the OCSP server is down, the connection might fail to proceed (soft fail)

#### Weaknesses

CAs can issue certificates for arbitrary domains, without permission from their owners.

#### SSL/TLS Offloading

**SSL Offloading** is the process of moving SSL/TLS encryption/decryption from the main application server to a dedicated device or software. This reduces the workload on the application server.

##### **How SSL Offloading Works**:
10. **Client Connection**:
	* A client initiates a secure connection to the server via HTTPS
11. **Offloading Device**:
	* A dedicated device (i.e., a load balancer, proxy or a specialized hardware (SSL terminator)) handles the TLS handshake, encryption and decryption.
12. **Plaintext Communication**: 
	* After decryption, the device forwards the unencrypted traffic to the application server. Responses are encrypted by the offloading device before being sent back to the client

##### Type of SSL Offloading
13. **SSL Termination:**
	* SSL is terminated at the offloading device. Traffic between the offloading device and server is unencrypted.
14. **SSL Bridging:**
	* SSL is terminated at the offloading device and re-encrypted before being sent to the server.
15. **SSL Pass-Through:**
	* SSL traffic is passed directly to the backend servers without decryption. The offloading device acts like a proxy.

##### Offloading Devices
16. Load Balancers
17. Content Delivery Networks (CDNs) (e.g., Cloudflare, Akamai)
18. Hardware Security Modules (HSMs)
19. Web Application Firewalls