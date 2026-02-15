---
title: Post-Quantum Hybrid Key Exchange
description: Quantum-resistant hybrid key encapsulation combining classical and post-quantum algorithms
layout: layouts/page
---

GlaSSLess provides **post-quantum hybrid key encapsulation mechanisms** that combine classical elliptic curve key exchange with ML-KEM for defense-in-depth cryptography. This approach ensures security against both classical and quantum computer attacks.

## Why Hybrid Key Exchange?

Hybrid key exchange provides defense-in-depth:

1. **If ML-KEM is broken**: Classical X25519/X448 still provides security
2. **If classical crypto is broken by quantum computers**: ML-KEM provides security
3. **Cryptographic agility**: Easy to transition as standards evolve

This protects against **Harvest Now, Decrypt Later (HNDL)** attacks, where adversaries collect encrypted data today to decrypt with future quantum computers.

## Supported Hybrid KEMs

*Requires OpenSSL 3.5+*

| Algorithm | Classical | PQC | Security Level | FIPS Approved |
|-----------|-----------|-----|----------------|---------------|
| X25519MLKEM768 | X25519 | ML-KEM-768 | 192-bit | Yes |
| X448MLKEM1024 | X448 | ML-KEM-1024 | 256-bit | Yes |

### Key and Ciphertext Sizes

| Algorithm | Public Key | Private Key | Ciphertext | Shared Secret |
|-----------|------------|-------------|------------|---------------|
| X25519MLKEM768 | 1216 bytes | 2432 bytes | 1120 bytes | 64 bytes |
| X448MLKEM1024 | 1664 bytes | 3168 bytes | 1632 bytes | 64 bytes |

## Basic Usage

```java
import javax.crypto.KEM;
import java.security.*;
import net.glassless.provider.GlaSSLessProvider;

Security.addProvider(new GlaSSLessProvider());

// Generate hybrid key pair
KeyPairGenerator kpg = KeyPairGenerator.getInstance("X25519MLKEM768", "GlaSSLess");
KeyPair keyPair = kpg.generateKeyPair();

// Encapsulation (sender side)
KEM kem = KEM.getInstance("X25519MLKEM768", "GlaSSLess");
KEM.Encapsulator encapsulator = kem.newEncapsulator(keyPair.getPublic());
KEM.Encapsulated encapsulated = encapsulator.encapsulate();

// Get shared secret and ciphertext
SecretKey sharedSecret = encapsulated.key();           // 64 bytes
byte[] ciphertext = encapsulated.encapsulation();      // 1120 bytes

// Decapsulation (receiver side)
KEM.Decapsulator decapsulator = kem.newDecapsulator(keyPair.getPrivate());
SecretKey recoveredSecret = decapsulator.decapsulate(ciphertext);

// Both parties now have identical 64-byte shared secrets
```

## Deriving Application Keys

The 64-byte shared secret should be used with a KDF to derive application keys:

```java
import javax.crypto.KDF;
import javax.crypto.spec.HKDFParameterSpec;

// Derive an AES-256 key from the shared secret
KDF hkdf = KDF.getInstance("HKDF-SHA256", "GlaSSLess");

byte[] info = "my-application v1.0".getBytes();
HKDFParameterSpec params = HKDFParameterSpec
    .ofExtract()
    .addIKM(sharedSecret.getEncoded())
    .addSalt(new byte[32])
    .thenExpand(info, 32);  // 32 bytes for AES-256

SecretKey aesKey = hkdf.deriveKey("AES", params);
```

## Partial Key Extraction

Extract a specific number of bytes directly during encapsulation:

```java
// Extract only 32 bytes for AES-256 key
KEM.Encapsulated encapsulated = encapsulator.encapsulate(0, 32, "AES");
SecretKey aesKey = encapsulated.key();  // 32-byte AES key
```

## Key Serialization

Hybrid KEM keys use a custom encoding format (not standard ASN.1):

```java
// Serialize keys
byte[] publicKeyBytes = keyPair.getPublic().getEncoded();
byte[] privateKeyBytes = keyPair.getPrivate().getEncoded();

// Reconstruct keys
KeyFactory kf = KeyFactory.getInstance("X25519MLKEM768", "GlaSSLess");

X509EncodedKeySpec pubSpec = new X509EncodedKeySpec(publicKeyBytes);
PublicKey reconstructedPublic = kf.generatePublic(pubSpec);

PKCS8EncodedKeySpec privSpec = new PKCS8EncodedKeySpec(privateKeyBytes);
PrivateKey reconstructedPrivate = kf.generatePrivate(privSpec);
```

## Checking Availability

```java
import net.glassless.provider.internal.OpenSSLCrypto;

// Check if hybrid KEMs are available (requires OpenSSL 3.5+)
if (OpenSSLCrypto.isAlgorithmAvailable("KEYMGMT", "X25519MLKEM768")) {
    System.out.println("X25519MLKEM768 is available");
} else {
    System.out.println("Requires OpenSSL 3.5+");
}
```

## TLS 1.3 Integration

### Current Status

JDK 25 does **not** support hybrid named groups in JSSE. Native TLS 1.3 hybrid key exchange is targeted for **JDK 27** (September 2026) via [JEP 527](https://openjdk.org/jeps/527).

### Application-Layer Workaround

Until JDK 27, you can perform hybrid key exchange at the application layer over a classical TLS connection:

```java
// Establish classical TLS 1.3 connection
SSLSocket socket = (SSLSocket) sslContext.getSocketFactory()
    .createSocket("server.example.com", 443);
socket.startHandshake();

// Perform hybrid key exchange over the TLS channel
ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
ObjectInputStream in = new ObjectInputStream(socket.getInputStream());

// Server: Generate and send hybrid public key
KeyPairGenerator kpg = KeyPairGenerator.getInstance("X25519MLKEM768", "GlaSSLess");
KeyPair keyPair = kpg.generateKeyPair();
out.writeObject(keyPair.getPublic().getEncoded());

// Client: Encapsulate and send ciphertext
byte[] serverPubKeyBytes = (byte[]) in.readObject();
KeyFactory kf = KeyFactory.getInstance("X25519MLKEM768", "GlaSSLess");
PublicKey serverPubKey = kf.generatePublic(new X509EncodedKeySpec(serverPubKeyBytes));

KEM kem = KEM.getInstance("X25519MLKEM768", "GlaSSLess");
KEM.Encapsulated encapsulated = kem.newEncapsulator(serverPubKey).encapsulate();
out.writeObject(encapsulated.encapsulation());

// Both sides derive application keys from the hybrid shared secret
```

## Standards and Specifications

| Standard | Description |
|----------|-------------|
| [draft-ietf-tls-hybrid-design](https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/) | TLS 1.3 hybrid key exchange |
| [FIPS 203](https://csrc.nist.gov/pubs/fips/203/final) | ML-KEM specification |
| [RFC 7748](https://tools.ietf.org/html/rfc7748) | X25519 and X448 |

## Algorithm Selection Guide

| Use Case | Recommended Algorithm |
|----------|----------------------|
| General purpose | X25519MLKEM768 |
| Maximum security | X448MLKEM1024 |
| Interoperability (future TLS) | X25519MLKEM768 |

**X25519MLKEM768** is recommended as the default choice and will be the default hybrid group in JEP 527 for TLS 1.3.
