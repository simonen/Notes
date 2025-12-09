---
tags:
  - centos
  - certificates
---

`man update-crypto-policies`

Manage the policies available to the various cryptographic back-ends.

FILES
- `/etc/crypto-policies/config`
- `/etc/crypto-policies/back-ends`
- `/etc/crypto-policies/local.d`
- `/etc/crypto-policies/state/current`
- `/etc/crypto-policies/state/CURRENT.pol`

To check the current system wide cryptographic policy profile

```
update-crypto-policies --show
```

Typical values:

- **`DEFAULT`** – Balanced security/performance.
- **`FUTURE`** – Stronger ciphers, may break compatibility.
- **`FIPS`** – Only FIPS-approved ciphers.
- **`LEGACY`** – Allows weaker ciphers for compatibility.

The **policy file** that defines the active system-wide cryptographic policy.

`/etc/crypto-policies/state/CURRENT.pol`
```
# Policy DEFAULT dump
# Baseline values for all scopes:
cipher = AES-256-GCM AES-256-CCM CHACHA20-POLY1305 AES-256-CTR AES-256-CBC AES-128-
group = X25519 SECP256R1 X448 SECP521R1 SECP384R1 FFDHE-2048 FFDHE-3072 FFDHE-4096 
hash = SHA2-256 SHA2-384 SHA2-512 SHA3-256 SHA3-384 SHA3-512 SHA2-224 SHA3-224
```

- The `CURRENT.pol` file is automatically generated when a crypto policy is changed.
- Applications like **OpenSSL, GnuTLS, and OpenSSH** reference this file to enforce security settings.

To temporarily enable legacy hashing algorithms

```
update-crypto-policies --set LEGACY
```

To set default crypto policy

```
update-crypto-policies --set DEFAULT
```
#### CA Certificate Trust Stores

`Root Certificate Trust Store`
``` bash
/etc/pki/ca-trust/source/anchors/
```

After adding a trust anchor (CA certificate) 

```
sudo update-ca-trust
```
