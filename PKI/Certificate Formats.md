
1. **DER (Distinguished Encoding Rules)**: 
	* A binary encoded format used to encode data in ASN.1, often used for certificates. The format used on the wire. Not typically used for exchanging files
	* File extensions: `.der .cert .crt .csr`
	* Java
2. **PEM (Privacy-Enhanced Mail)**: 
	* A base64-encoded version of DER with delimiters (---BEGIN CERTIFICATE--- and ---END CERTIFICATE---) used for storage and transmission of certificates. 
	* Can contain multiple certificates
	* File extensions: `.pem .cert .crt .key .csr`
	* Apache, NGINX, Linux, Cisco routers, ASAs, BigIP F5

`PKCS#[1-15]` - Public-key Cryptography Standard

- **PKCS#1**: For RSA private keys.
- **PKCS#7**: A broader standard for encrypted and signed messages, supporting various cryptographic enhancements beyond just RSA. Base64-encoded. ---BEGIN PKCS7---
	- File extensions: `.pkcs7 .p7b .p7c`
	- SMIME (Secure Multi Purpose Internet Mail Extensions), Java, Windows
- **PKCS#8**: Current NEW standard format for private keys. RSA, ECDSA, Ed25519..
- **PKCS#12** (Personal Information eXchange): 
	- Formerly known as **PFX certificate**. A binary format for storing the certificate, private key and optionally the CA certificate chain.
	- File extensions: `.p12` or `.pfx` `.pkcs12`
	- IIS, Windows, Tomcat, Firefox

Read a CRL in DER format in OpenSSL

```bash
openssl crl -in 'FILE' -inform DER -noout -text
```

TLS Version numeric representation

```perl
TLS_VERSIONS = {
    # SSL
    0x0002: "SSL_2_0", # 2
    0x0300: "SSL_3_0", # 768
    # TLS:
    0x0301: "TLS_1_0", # 769
    0x0302: "TLS_1_1", # 770
    0x0303: "TLS_1_2", # 771
    0x0304: "TLS_1_3", # 772
    # DTLS
    0x0100: "PROTOCOL_DTLS_1_0_OPENSSL_PRE_0_9_8f", # 256
    0x7f10: "TLS_1_3_DRAFT_16", # 32528
    0x7f12: "TLS_1_3_DRAFT_18", # 32530
    0xfeff: "DTLS_1_0", # 65279
    0xfefd: "DTLS_1_1", # 65277
}
```

#### Certificates

A PEM base64-encoded certificate

```
-----BEGIN CERTIFICATE-----
MIICjjCCAjOgAwIBAgIQf/NXaJvCTjAtkOGKQb0OHzAKBggqhkjOPQQDAjBQMSQw
...TRUNCATED...
L2MucGtpLmdvb2cvci9nc3I0LmNybDATBgNVHSAEDDAKMAgGBmeBDAECATAKBggq
IQDspFWa3fj7nLgouSdkcPy1SdOR2AGm9OQWs7veyXsBwA==
-----END CERTIFICATE-----
```

The human-readable version of the PEM certificate

```
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            20:42:47:ce:fe:e5:1f:df:3c:ba:14:48:2f:ed:6e:11
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = BG, O = Ohio, CN = Root CA
        Validity
            Not Before: Feb  2 16:29:29 2025 GMT
            Not After : Jan 31 16:29:29 2035 GMT
        Subject: C = BG, ST = Sofia, O = Ohio, OU = IT, CN = node1
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b5:50:a3:3f:f7:43:ef:b7:26:00:d6:77:45:79:
					...TRUNCATED...
                    9e:44:30:2f:cf:dd:79:9e:38:d9:44:fc:5a:d6:8d:
                    e7:8b
                Exponent: 65537 (0x10001)
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        2d:d0:d2:dc:47:3c:f3:a0:ed:13:d0:28:c7:cb:95:c7:83:47:
        46:fd:c1:53:e2:19:27:ff:fb:b1:3e:d6:44:0f:22:89:73:5d:
		...TRUNCATED...
        b4:20:9d:3f:79:d0:d8:7f:3b:ab:df:a7:8f:a9:6b:ab:13:b9:
        e9:04:7e:66:43:6b:f2:06
```

#### PKCS#8 Private Key Formats

* **Unencrypted PKCS#8**

`PEM`
```
-----BEGIN PRIVATE KEY-----
(Base64-encoded key data)
-----END PRIVATE KEY-----
```

* **Encrypted PKCS#8**. Uses **PBKDF2 + AES** for encryption

`PEM`
```
-----BEGIN ENCRYPTED PRIVATE KEY----- 
(Base64-encoded key data) 
-----END ENCRYPTED PRIVATE KEY-----
```

#### PKCS#1 Private Key Format

`PEM`
```
-----BEGIN RSA PRIVATE KEY-----
(Base64-encoded key data) 
-----END RSA PRIVATE KEY-----
```

