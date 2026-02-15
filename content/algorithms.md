---
title: Supported Algorithms
description: Complete list of cryptographic algorithms supported by GlaSSLess
layout: layouts/page
---

GlaSSLess provides **275+ algorithm implementations** covering all major cryptographic operations.

## Message Digests (18)

| Algorithm | FIPS Approved |
|-----------|---------------|
| SHA-224, SHA-256, SHA-384, SHA-512 | Yes |
| SHA-512/224, SHA-512/256 | Yes |
| SHA3-224, SHA3-256, SHA3-384, SHA3-512 | Yes |
| SHAKE128, SHAKE256 | Yes |
| BLAKE2b-512, BLAKE2s-256 | Yes |
| MD5, SHA-1 | No |
| SM3, RIPEMD160 | No |

## Ciphers (143)

| Algorithm | Key Sizes | Modes | FIPS Approved |
|-----------|-----------|-------|---------------|
| AES | 128, 192, 256 | ECB, CBC, CFB, CTR, OFB, GCM, CCM, XTS | Yes |
| AES Key Wrap | 128, 192, 256 | KW, KWP | Yes |
| Camellia | 128, 192, 256 | ECB, CBC, CFB, CTR, OFB | No |
| ARIA | 128, 192, 256 | ECB, CBC, CFB, CTR, OFB, GCM | No |
| SM4 | 128 | ECB, CBC, CFB, CTR, OFB | No |
| ChaCha20 | 256 | Stream | No |
| ChaCha20-Poly1305 | 256 | AEAD | No |
| DESede (3DES) | 168 | ECB, CBC | No |
| RSA | 1024-8192 | ECB with PKCS1/OAEP | Yes |

## MACs (20)

| Algorithm | FIPS Approved |
|-----------|---------------|
| HmacSHA224, HmacSHA256, HmacSHA384, HmacSHA512 | Yes |
| HmacSHA3-224, HmacSHA3-256, HmacSHA3-384, HmacSHA3-512 | Yes |
| AESCMAC, AESGMAC | Yes |
| KMAC128, KMAC256 | Yes |
| HmacSHA1 | No |
| Poly1305, SipHash | No |

## Signatures (46+)

| Algorithm | FIPS Approved |
|-----------|---------------|
| SHA256withRSA, SHA384withRSA, SHA512withRSA | Yes |
| SHA256withRSAandMGF1 (RSA-PSS) | Yes |
| SHA256withECDSA, SHA384withECDSA, SHA512withECDSA | Yes |
| SHA3-256withECDSA, SHA3-384withECDSA, SHA3-512withECDSA | Yes |
| SHA256withDSA, SHA384withDSA, SHA512withDSA | Yes |
| Ed25519, Ed448, EdDSA | Yes |
| ML-DSA-44, ML-DSA-65, ML-DSA-87 | Yes (FIPS 204) |
| SLH-DSA-SHA2-128s/f, SLH-DSA-SHA2-192s/f, SLH-DSA-SHA2-256s/f | Yes (FIPS 205) |
| SLH-DSA-SHAKE-128s/f, SLH-DSA-SHAKE-192s/f, SLH-DSA-SHAKE-256s/f | Yes (FIPS 205) |
| SHA1withRSA, SHA1withECDSA, SHA1withDSA | No |

## Key Encapsulation Mechanisms (3)

*Requires OpenSSL 3.5+*

| Algorithm | Security Level | FIPS Approved |
|-----------|----------------|---------------|
| ML-KEM-512 | 128-bit | Yes (FIPS 203) |
| ML-KEM-768 | 192-bit | Yes (FIPS 203) |
| ML-KEM-1024 | 256-bit | Yes (FIPS 203) |

ML-KEM (Module-Lattice-Based Key-Encapsulation Mechanism) provides quantum-resistant key exchange using Java's `KEM` API introduced in JDK 21.

## Key Agreement (5)

| Algorithm | FIPS Approved |
|-----------|---------------|
| ECDH, DH | Yes |
| X25519, X448, XDH | Yes |

## Key Derivation Functions (5)

| Algorithm | FIPS Approved |
|-----------|---------------|
| HKDF-SHA256, HKDF-SHA384, HKDF-SHA512 | Yes |
| HKDF-SHA224 | Yes |
| HKDF-SHA1 | No |

## Secret Key Factories (32)

| Algorithm | FIPS Approved |
|-----------|---------------|
| PBKDF2WithHmacSHA256, PBKDF2WithHmacSHA384, PBKDF2WithHmacSHA512 | Yes |
| PBKDF2WithHmacSHA3-256, PBKDF2WithHmacSHA3-384, PBKDF2WithHmacSHA3-512 | Yes |
| PBKDF2WithHmacSHA1 | No |
| SCRYPT | No |

## Secure Random (3)

| Algorithm | FIPS Approved |
|-----------|---------------|
| NativePRNG, DRBG | Yes |
| SHA1PRNG | No |
