
```bash
openssl 'SUBCOMMAND' -help
```

```bash
man openssl-genpkey
```

Inspect a PEM base64-encoded certificate

```bash
openssl x509 -in 'CERT' [-noout] -text
```

`-noout`: Do not print the base64-encoded version of the certificate on screen
`-text`: Convert the base64 string of the PEM certificate to human-readable format

#### Generating and Inspecting RSA Keys

PKCS#8

```bash
openssl genpkey \
-out fd.key \
-algorithm RSA \
-pkeyopt rsa_keygen_bits:2048 \
[-aes-128-cbc]
```

`-aes-128-cbc`: Encrypts the private key with a password
`-pkeyopt`: Specify non-default algorithm options, such as key size. RSA defaults to 2048

Inspect the private key in human readable (-text) format

```bash
openssl pkey|rsa -in 'PRIVATE.KEY' -text -noout
```

Extract the public key

```bash
openssl pkey -in 'PRIVATE.KEY' -noout -text -pubout
```

Extract just the modulus of an RSA public key

```bash
openssl rsa -in 'PRIVATE.KEY' -noout -modulus
```
#### Generating and Inspecting ECDSA Keys

```bash
openssl genpkey -out fd-ecdsa.key \
-algorithm EC \
-pkeyopt ec_paramgen_curve:P-256 \
-aes-128-cbc
```
or
```bash
openssl ecparam -genkey -name prime256v1 -out ecdsa_private.pem
```

- `-pkeyopt ec_paramgen_curve`: The named curve
- `-aes-128-cbc`: Encrypt the key

List available EC named curves

```bash
openssl ecparam -list_curves
```

Extract the public key from the corresponding private key

```bash
openssl pkey -in 'PRIVATE.KEY' -noout -text -pubout
```

Using the `ec` utility

```bash
openssl ec -in 'PRIVATE.KEY' -noout -text -pubin
```

Encrypt a private key (RSA, ECDSA)

```bash
openssl pkcs8 -in privkey.pem -topk8 -v2 aes-256-cbc -out privkey.key
```

Decrypt a private key

```bash
openssl pkcs8 -in encrypted_key.pem -topk8 -nocrypt -out unencrypted_key.pem
```

#### Generating and Inspecting Certificate Signing Requests

Generate a CSR and sign it with an existing private key

```bash
openssl req -new -key 'PRIVATE_KEY.key' -out 'CSR'.csr
```

Generate a new CSR along with a new RSA private key. `privkey.pem` automatically created

```bash
openssl req -new -newkey rsa:2048 -nodes -out 'CSR.pem' -keyout 'PRIVATE.KEY'
```

Non-interactive CSR and private key generation 

```bash
openssl req -new \
-newkey rsa:4096 -nodes \
-out CSR.pem \
-keyout private.key \
-subj "/C=US/ST=California/L=San Francisco/O=MyCompany/OU=IT/CN=mydomain.com"
```

`-nodes`: No DES encryption

Inspect a CSR

```bash
openssl req -in 'CSR' -noout -text
```

Extract the public key (EC) from a CSR. Pipe the encoded public key to the `-pubin` sub-util.

```bash
openssl req -in 'CSR' -pubkey | openssl pkey -pubin -text -noout
```

#### Signing Certificate Requests and Inspecting Certificates

Issue an end-entity certificate. CA keys specified in the .conf file.

```bash
openssl ca -config 'sub-ca.conf' \
-in 'END_ENTITY.CSR' \
-out 'END_ENTITY.CRT' \
-extensions 'client_ext/server_ext'
```

Inspect a certificate

```bash
openssl x509 -in 'CERT.CRT' -noout [-text] [-OPTION] [-ext EXTENSION1,EXTENSION2]
```

OPTIONS:
`serial,issuer,subject...`

EXTENSIONS
`subjectAltName,keyUsage,BasicConstraints...`

Extract the public key from a certificate. Piping to `pkey` converts to human-readable

```bash
openssl x509 -in 'CERT' -pubkey -noout [| openssl pkey -pubin -text -noout]
```

#### Verification

Verify the certs against the CA issuer. Full chain is required anyway.

```bash
openssl verify -CAfile 'FULL-CHAIN.CRT' 'CLIENT/SERVER'.crt
```

If certificates in separate files

```bash
openssl verify -CAfile 'ROOT-CA.CRT' -untrusted 'SUB-CA.CRT' 'END-ENTITY.CRT'
```

Verify and show the full chain

```bash
openssl verify -show_chain -CAfile root-ca.crt -untrusted sub-ca.crt gitlab.crt
```

```
gitlab.crt: OK
Chain:
depth=0: C = BG, ST = Sofia, O = Ohio, OU = IT, CN = gitlab.ohio.cc (untrusted)
depth=1: C = BG, O = Ohio, CN = Sub CA (untrusted)
depth=2: C = BG, O = Ohio, CN = Root CA

```

Print an entire certificate chain

```bash
openssl crl2pkcs7 -nocrl -certfile 'CHAIN.CRT' | openssl pkcs7 -print_certs -text -noout
```

Check revocation with OCSP

```bash
openssl ocsp -issuer 'CA.crt' -cert 'CERT' -url 'http://ocsp' [-req_text]
```

Remove the passphrase from a private key

```bash
openssl ec -in ecdsa-key.pem -out ecdsa-key-no-pass.pem
```

#### Enumerate Server's Supported Ciphers

TCP

```bash
nmap --script ssl-enum-ciphers -p 'PORT' 'HOST'
```

```
PORT   STATE SERVICE
25/tcp open  smtp
| ssl-enum-ciphers:
|   TLSv1.0:
|     ciphers:
|       TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (dh 1024) - D
```

#### Convert Between Formats

#### Test SSL/TLS

Extract the certificate section only

```bash
openssl s_client -connect HOST:PORT [-showcerts] | sed -n '/-BEGIN CERT/,/-END CERT/p' > 'CRT'
```

`-showcerts`: Requests the certificate chain

```bash
openssl s_server -help
openssl s_client -help
```

OPTIONS:
`-verify 1`: Enable client certificate verification
`-www`: Simulate an HTTPS response
`-debug`: Show Handshake details

Test the connection (if local)

```bash
openssl s_client -connect localhost:'PORT'
```

Test mutual authentication by providing a client certificate

```bash
openssl s_client -connect ldap.ohio.cc:636 \
    -CAfile /etc/openldap/certs/cacert.pem \
    -cert /etc/openldap/certs/headoffice.ohio.cc.cert \
    -key /etc/openldap/certs/headoffice.ohio.cc.key \
    -tls1_2 \
    -cipher 'HIGH:!aNULL:!MD5'
```

Set up a basic SSL/TLS server to listen for connections and present a server certificate.

```bash
openssl s_server -CAfile 'FULL.CRT' -cert 'SERVER.CRT' -key 'SERVER.KEY' -accept 'PORT' [OPTIONS]
```

#### Digests

List supported digest algorithms

```bash
openssl dgst -list
```

Create a digest. Defaults to SHA-256

```bash
openssl dgst [-DIGEST] [FILE]
```

Hash strings. Can be be used in the `shadow` file.

```bash
openssl passwd -6 'yourpassword'
```

- `-5`: SHA-256
- `-6`: SHA-512
- `-1`: MD5