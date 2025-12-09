
`TLS`: A framework for creating cryptographic protocols
Versions prior to TLS 1.2 included hardcoded cryptographic primitives into the protocol.
TLS 1.2 and on, are fully configurable.

Encryption provides confidentiality, not integrity

#### Cipher Suites

`Cipher Suite`: A selection of cryptographic primitives and other parameters that define exactly how security will be implemented. 

Components of a cryptographic suite:

1. Key Exchange Algorithm:
	* Determines how the client and server will securely exchange cryptographic keys
	* Examples:
		* RSA: Does not offer *forward secrecy*. Avoid.
		* DH
		* ECDH(E): Provides *forward secrecy*. 
2. Authentication Algorithm:
	* Verifies the identity of the communicating parties, typically using digital signatures
	* Examples:
		* RSA: Most popular. Moving to 3072-bit keys degrades performance.
		* DSA (Digital Signature Algorithm). Keys limited to 1024 bits. Deprecated
		* ECDSA (Elliptic Curve Digital Signature Algorithm). Better and faster than RSA
3. Symmetric Encryption Algorithm
	* Encrypts the actual data
	* Examples:
		* AES (Advanced Encryption Standard)
		* ChaCha20
		* 3DES (Triple Data Encryption Standard)
4. Message Authentication Code (MAC)
	* Ensures data integrity and authenticity
	* Examples:
		* HMAC-SHA256


```
Proto_Kx_Au_WITH_Enc_Mac
```

```
TLS_AES_256_GCM_SHA384    TLSv1.3 Kx=any    Au=any    Enc=AESGCM(256)    Mac=AEAD
```

```
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
```

* **TLS**: The protocol using the cipher suites
* **ECDHE**: Key-exchange. Use the `secp256r1` curve. 128 bits of security. Better than DHE
* **ECDSA**: Authentication
* **AES_256_GCM**: Symmetric encryption scheme, provides confidentiality and integrity
	* AES: Encryption Algorithm
	* 256: Strength (bits)
	* GCM: (Galois/Counter) Mode for confidentiality
* **SHA384**: (Secure Hash Algorithm) 384-bit. Used for hashing the HMAC process. 
	* Provides integrity verification.

> Suites with ECDSA authentication mechanism require ECDSA keys!

If a server has a single RSA key, it will not show support for any of the ECDSA suites.

Use CBC suites with SHA1 for performance. They offer no better security than SHA256/384

#### Encryption Ciphers

##### Stream Ciphers

One byte of plaintext fed to the algorithm produces one byte of ciphertext.
One byte of `keystream` is combined with one byte of plaintext using XOR logical operation. XOR is reversible, to decrypt, XOR of ciphertext is performed with the same keystream byte.

> Stream ciphers should not be used more than once with the same key

* Better performance in mobile and embedded systems.
* Vulnerable to changing of order of bits in stream - breaking integrity
* Must be paired with MAC to maintain integrity.

ChaCha20: Good
RC4: Bad
##### Block Ciphers

AES-256: Good
3DES
DES

Encrypt entire blocks of data at a time. A block cipher is a transformation function. A key property of block ciphers is that a small change in input produces a large variation in the output. 

`AES (Advanced Encryption Standard)`: Most popular block cipher. Available in 128, 192 and 256 lengths. 

**Padding**: In TLS, the last byte of the encryption block contains padding length, which indicates how many bytes of padding (excluding the padding length byte) there are.

* Better performance in PCs, laptops, servers due to hardware AES acceleration.
* *Deterministic* - they produce the same output for the same input, thus patterns are not obscured in ciphertext.
* Must be paired with **Operation Modes (BCM)** to address this vulnerability.

 **Block Cipher Modes**

BCM are cryptographic schemes designed to extend block ciphers to encrypt data of arbitrary length. All BCMs support confidentiality, but some combine it with authentication.

* `ECB (Electronic Codebook)`: Simplest possible cipher mode. Data length must be an exact multiple of the block size. Requires pre-padding. Bad 
* `CBC (Cipher Block Chaining)`: Introduces the concept of `initialization vector (IV)` which makes the output different every time, even if the input is the same. Slow.
* `CTR (Counter)`: Uses a *nonce* with an added incremental number for each block to encrypt blocks, which guarantees each cipher block is unique. Must be paired with MAC.
* `GCM (Galois/Counter)`: CTR with built-in MAC. AEAD cipher. New, starting with TLS 1.2. Provides confidentiality and integrity. Currently the best

AEAD (Authenticated Encryption with Associated Data): Do symmetric encryption and MAC at the same time. TLS 1.2+

`nonce`: a number used once
#### Hash Functions

A `hash function` is an algorithm that converts an input of arbitrary length into fixed-size output. The result is called `hash, fingerprint, digest` . Used as a compact way to represent and compare large amounts of data.

Cryptographic functions have some important properties:
* Preimage resistance - It is computationally unfeasible to construct a message that produces a hash
* Second preimage resistance
* Collision resistance - Not feasible to find two messages that have the same hash

`SHA1`: 160 bits.
`SHA2`
`Poly1305`: Polynomial MAC
#### Message Authentication Codes

MAC or a `keyed-hash` is a crypto functions that extends hashing with authentication. Only those in possession of the hashing key can produce a valid MAC. Ensures integrity and authenticity.

Any hash function can be used as the basis for a MAC using a construction known as HMAC (Hash Based Message Authentication Code). HMAC works by interleaving the hashing key with the message in a secure way.

AEAD ciphers eliminate separate MAC operations.

Common MAC algorithms in TLS:
* HMAC-SHA256
* HMAC-SHA384
* Poly1305 (Used with ChaCha20-Poly1305)
* Galois Message Authentication Code (Used in GCM mode)
#### Symmetric Key Encryption (Private-key Cryptography)

Uses the same key for both encryption and decryption. Example: Cesar Cipher, Vigenere Cipher. Used to encrypt the actual data during the session.

Original document -> Encrypt with secret key -> Encrypted document -> Decrypt with same key -> Original document

For an encryption algorithm to be useful, it must be shared with others. The security of the ciphertext depends entirely on the key.  Suitable for bulk encryption. Asymmetric encryption protects the sharing of secret keys.

#### Asymmetric Encryption (Public-Key Cryptography)

Uses two keys: public and private.

* Data encrypted with a public key can only be decrypted by the **corresponding** private key.
* Digital signatures encrypted with a private key can only be decrypted with the corresponding public key.

Public-key cryptography is slow and deployed for authentication and negotiation of shared secrets only, which are then used for fast symmetric (private-key cryptography) encryption.
Used to protect the secret key exchange.

`RSA`: Most popular. Recommended strength is 2048 bits, equivalent to 112 symmetric bits.

##### Public and Private Keys

Public and private keys are mathematically bound together by a common modulus. Private keys are created first from which the public keys are derived.

An RSA public key consists of:

**Modulus**
**Exponent**

`RSA public key in a certificate`
```
Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d8:f8:cd:5e:3f:ef:ba:e5:30:c1:19:b6:ce:08:
					...
                    ea:82:b4:4a:d3:2e:59:b6:89:51:65:68:4a:a4:f8:
                    69:c9
                Exponent: 65537 (0x10001)
```

The exponent of the public key matches the modulus of the corresponding private key. This proves the correlation of the key pair.

`RSA public key in the corresponding private key`
```
Public-Key: (2048 bit)
Modulus:
    00:d8:f8:cd:5e:3f:ef:ba:e5:30:c1:19:b6:ce:08:
	...
    ea:82:b4:4a:d3:2e:59:b6:89:51:65:68:4a:a4:f8:
    69:c9
Exponent: 65537 (0x10001)

```
#### Key-exchange Scheme

Used to negotiate encryption keys between client and server securely. Key exchange is determined by the choice of server private key algorithm, key size.

* The client generates a **pre-master** secret and encrypts it with the server's public key. (from server's certificate).
* Sends the encrypted pre-master key to the server.
* The server decrypts the pre-master secret using its private key.

**Session Key Generation**

- Both the client and server use the pre-master secret, along with the client random and server random values, to generate a shared **session key** (symmetric key).
- This session key will be used to encrypt and decrypt data during the session.

**Key Exchange Algorithms**:

`DH (Diffie-Hellman)`: Both parties use a combination of their private keys and the other party's public key to generate a shared secret, which is used as a symmetric key for further encryption
`ECDH (Elliptic Curve DH)`: The server specifies a `named curve`, which is a reference to one of the possible predefined parameters. Most widely supported NIST curves are `secp256r1` and `secp384r1`. 
`RSA`: Should not be used for key exchange. No forward secrecy.
`X448 (Curve448)`:  224 bits of security
`X25519`: 128 bits of security.

Combinations are pre-determined by the protocol
1. RSA
2. DHE_RSA
3. ECDHE_RSA
4. ECDHE_ECDSA

#### Digital Signatures

A cryptographic scheme that makes it possible to verify the authenticity of a digital document or message. 

Producing a signature:
5. Calculate the hash of the document. Output is fixed at 256 bits for SHA2
6. Encode the resulting hash and some additional metadata, like the hashing algorithm used. 
7. Encrypt the encoded hash using the private key. This is the DS. Appended to the doc.

Verifying the signature
8. The receiver calculates the hash independently using the same algorithm 
9. Uses the public key of the sender to decrypt the message and recover the hash
10. Confirm that the same algo was used
11. Compare the calculated hash with the decrypted hash

#### Signature Algorithms

Digital signatures depend on the strength of the CA's private key, and the strength of the hashing algorithm

By using `signature_algorithm` extension, clients submit the `signature-hash` algorithm pairs they support. If the extension is not present, the server picks a signature algorithm from the client's offered cipher suites. 

`Signature algorithms`:  

RSA, DSA, ECDSA

#### Random Number Generation

Cryptographic security relies on known encryption algorithms and `secret keys`. Those keys are simply very long random numbers.

`enthropy`: Adding randomization to numbers with physical events like keystrokes, mouse movements, etc
`seeding`
`AES-256` uses a 256-bit long number as a secret key 

#### The Handshake Protocol

The most elaborate part of the TLS protocol, during which the sides negotiate connection parameters and perform authentication. Requires 6 to 10 messages.

##### Full Handshake

`key exchange, key establishment`

15. Client begins a new handshake and submits its capabilities to the server
16. Server selects connection parameters
17. Server sends its certificate chain (only if server authentication is required)

**Client Hello**:

- The client sends a "`ClientHello`" message to the server, which includes:
    - Supported TLS/SSL versions.
    - Supported cipher suites (encryption algorithms).
    - A random number (used in key generation).
    - Optionally, a list of supported compression methods and extensions (e.g., Server Name Indication, SNI).

The first message in a new handshake

`wireshark`
```
Transport Layer Security
    TLSv1.3 Record Layer: Handshake Protocol: Client Hello
        Content Type: Handshake (22)
        Version: TLS 1.0 (0x0301)
        Handshake Protocol: Client Hello
            Handshake Type: Client Hello (1)
            Version: TLS 1.2 (0x0303)
            Random: 6323641675b55bc5664caa93..7e12ef53f742fafb0a9517
            Session ID: dd6651aa71ea14c89f53293475d5c0a2b7bbe5bb9525fa
            Extension: server_name (len=19) name=gitlab.ohio.cc
                Type: server_name (0)
                Server Name Indication extension
			Signature Hash Algorithms (8 algorithms)
			    Signature Algorithm: ecdsa_secp256r1_sha256 (0x0403)
			    Signature Algorithm: rsa_pss_rsae_sha256 (0x0804)
			    Signature Algorithm: rsa_pss_rsae_sha512 (0x0806)
			    Signature Algorithm: rsa_pkcs1_sha512 (0x0601)
            Extension: supported_versions (len=7) TLS 1.3, TLS 1.2
                Supported Version: TLS 1.3 (0x0304)
                Supported Version: TLS 1.2 (0x0303)
```

**Server Hello**:

- The server responds with a "ServerHello" message, including:
    - Chosen TLS/SSL version.
    - Chosen cipher suite.
    - Another random number.
    - Any agreed-upon extensions.

`wireshark`
```
Transport Layer Security
    TLSv1.3 Record Layer: Handshake Protocol: Server Hello
        Content Type: Handshake (22)
        Version: TLS 1.2 (0x0303)
        Handshake Protocol: Server Hello
            Handshake Type: Server Hello (2)
            Version: TLS 1.2 (0x0303)
            Random: 6b8e15755fb783eb64803b3..c6bcc5f929166dff5dfa30a078bfbbf1e
            Session ID: dd6651aa71ea14c89f532934bd..e8c575675d5c0a2b7bbe5bb9525fa
            Cipher Suite: TLS_AES_128_GCM_SHA256 (0x1301)
            Extension: supported_versions (len=2) TLS 1.3
                Type: supported_versions (43)
                Supported Version: TLS 1.3 (0x0304)
    TLSv1.3 Record Layer: Change Cipher Spec Protocol: Change Cipher Spec
        Content Type: Change Cipher Spec (20)
        Version: TLS 1.2 (0x0303)
        Change Cipher Spec Message
    TLSv1.3 Record Layer: Application Data Protocol: Hypertext Transfer Protocol
        Opaque Type: Application Data (23)
        Version: TLS 1.2 (0x0303)
```

**Server Certificate**:

- The server sends its **digital certificate** to the client. This certificate contains:
    - The server's public key.
    - Information about the server (domain, organization, etc.).
    - The certificate authority (CA) signature verifying the server's identity.

**Server Key Exchange (optional)**:
**Client Certificate Request (optional)**:
**Server Hello Done**:
**Client Key Exchange**:

#### Authentication

Authentication is tightly coupled with key exchange.

`Implicit authentication`: Assuming that only the server with the corresponding private key can decrypt a message, encrypted with its public key

In TLS, integrity validation is part of the encryption process; it's handled either explicitly at the protocol level or implicitly by the negotiated cipher.

Authentication ciphers combine encryption and integrity validation in one algorithm.
`AEAD` (authenticated encryption with associated data). Use a special value called `nonce` which must be unique. Currently, the best encryption mode in TLS.

`renegotiation_info extension`: Correcting flaw in the original renegotiation protocol

#### Extensions

`Server Name Indication (SNI)`: Provides a mechanism through `server_name_extension` to for a client to specify the name of the server it wishes to connect to. Provides support for `virtual secure servers`. 

#### Session Tickets

`Session tickets` introduce a new session resumption mechanism. The server takes all session data, encrypts it and sends it back to the client in the form of a ticket. The client submits the ticket to the server on subsequent connections. Makes it easier for web server clusters, which otherwise need to sync session states among nodes.

A session ticket consists of 48 bytes of cryptographically random data. The data is used for three 16-byte fragments, one each for key name, HMAC secret, and AES key.

> If session tickets are used, ticket keys must be rotated frequently, as they can be used to decrypt the full connection data

#### Protocol Limitations

* TLS operates on Layer6 of the OSI model. Metadata of TCP and lower layers are still in plaintext. 
* The first handshake is not encrypted. Lots of information can still be revealed to a passive observer: client capabilities, intended virtual hosts, host certificate, potentially data to identify the user. 
* Information on the intended resource usage can be obtained through indicated HTTP request and response sizes. Length hiding needed

#### Protocols

Cryptographic primitives such as encryption and hashing algorithms are combined into schemes and protocols to satisfy complex security requirements.

1. Handshake phase
2. Data exchange phase with confidentiality, integrity
3. Shutdown sequence
#### Measuring Strength

[keylength.com](keylength.com) for a high level security levels guidelines

> Secure against whom and for how long?

Deploying strong key sizes is the easiest thing to get right.
128 bits of security is (2^128 operations) is sufficient for most deployments.

Encryption strength mapping for commonly used key sizes across different schemes

| Symmetric | RSA DSA DH | EC  | Hash |
| --------- | ---------- | --- | ---- |
| 112       | 2048       | 224 | 224  |
| 128       | 3072       | 256 | 256  |
| 256       | 15360      | 512 | 512  |

#### Elliptic Curves

X448
X25519
