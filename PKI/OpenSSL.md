---
tags:
  - certificates
  - openssl
---
Apress – Pro Linux System Administration 2nd 2017.pdf

Types of certificates
* issued by a commercial CA 
* issued by noncommercial CA - Let's Encrypt
* issued by a self-managed CA
* self-signed - Subject = Issuer, hence no root certificate to add to the client. The untrusted certificate error will not go away even if the cert is added to the trusted root CA

Determine OpenSSL version and configuration

```bash
openssl version [-a]
```

```
OPENSSLDIR: "/etc/pki/tls"
```

View supported ciphers

``` bash
openssl ciphers -v ALL
```


`<status> <expiration date> <serial number> <revocation date> <DN of the cert subject>`

Cert statuses in the index file

`R`: Revoked
`V`: Valid
`E`: Expired

Create a self-signed certificate and a private key for the CA

``` bash
openssl req -new -x509 -newkey rsa:4096 -keyout private/cakey.pem \ 
-out certs/cacert.pem -days 3650 \
-subj '/C=BG/ST=Sofia/L=Sofia/O=Example Inc/OU=IT/CN=ca.example.com/emailAddress=admin@example.com/'
```

`-x509`: indicates that the cert is self-signed

Create a certificate signing request key pair

``` bash
openssl req -new -newkey rsa:4096 -nodes -keyout CN.key -out CN.req
```

Sign the certificate signing request

``` bash
openssl ca -out CN.cert -cert certs/cacert.pem -infiles CN.req
```

#### Certificate Extensions (for web servers)

Browsers require additional v3 SSL extensions in the certificates, specifically the **Subject Alternative Name** (`subjectAltName`) extension

1. Generate the CSR with Subject Alternative Name (SAN)
2. Sign the CSR with the SAN v3 extension

`csr_san.conf`
```
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = req_distinguished_name
req_extensions     = req_ext

[ req_distinguished_name ]
C  = BG
ST = Sofia
L  = Sofia
O  = Ohio Inc
OU = IT
CN = headoffice.ohio.cc

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = headoffice.ohio.cc
DNS.2 = www.headoffice.ohio.cc
```

Create the request

``` bash
openssl req -new -key headoffice.key -out headoffice.csr -config csr_san.cnf
```

`openssl_ext.cnf`
```
[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = headoffice.ohio.cc
DNS.2 = www.headoffice.ohio.cc
```

Sign the request 

``` bash
openssl ca -out headoffice.ohio.cc.crt -cert cacert.pem \
-extfile openssl_ext.cnf -extensions v3_req -infiles headoffice.ohio.cc.csr
```

Verify if the certificate has the X509v3 Subject Alternative Name extension

``` bash
openssl x509 -in headoffice.crt -noout -text | grep -A1 "Subject Alternative Name"
```

```
            X509v3 Subject Alternative Name:
                DNS:headoffice.ohio.cc, DNS:www.headoffice.ohio.cc
```

Import the CA certificate into the **web browser** certificate store.

#### Securing the Certificate Authority

1. The CA should be located in a dedicated machine, password encrypted disk and disconnected from the network when not being used. Or outright shutdown.
2. An Intermediate CA should be used to sign certificates on behalf of the root CA
3. Private keys should not be lying around on hosts that are not using them

```bash
openssl req -x509 -days 365 -key server.key -in server.csr -out server.crt -extensions v3_ca -extfile openssl.cnf
```

#### Key Usage Extension

Specify what exactly the certificate must be used for.

```
[ req ]
distinguished_name  = req_distinguished_name
req_extensions      = v3_req

[ v3_req ]
keyUsage = critical,digitalSignature,keyEncipherment
extendedKeyUsage = clientAuth
```

Compare cert and key moduli 

``` bash
openssl x509 -noout -modulus -in ".cert" | openssl md5
```

``` bash
openssl rsa -noout -modulus -in ".key" | openssl md5
```
