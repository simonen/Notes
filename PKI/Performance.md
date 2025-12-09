
**Distributed session caching**: 
	**Sticky Load Balancing**: The same client is always sent to the same cluster node
	Share the TLS session cache among all the nodes in the cluster

#### SPDY Protocol

Increases HTTP performance

Key size should provide the appropriate level of security but not over that. 
ECDSA is faster and better than RSA 256-bit ECDSA = 3072-bit RSA.

DHE ( slow )
	1024 = 80 bits of security
	2048 = 112 bits of security

ECHDE 
	secp256r1 = 128 bits of security
	secp384r1 30% slower, no meaningful increase in security

Use as few certificates in the chain as possible. Two certificates in the chain is the best: one for the server and one for the issuing CA. Do not include unnecessary certificates in the chain. Root certificate serves no purpose.

USE CAs who use CDNs to distribute revocation information

ChaCha20-Poly1305: for mobile

ECDHE_ECDSA is better in the case of DoS. The client performs 1.5 times more work than the server. Renegotiation amplifies DoS attacks.





