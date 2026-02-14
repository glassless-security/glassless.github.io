---
title: Getting Started
description: How to install and use GlaSSLess JCA Provider
layout: layouts/page
---

## Requirements

- **Java**: 25 or later (with `--enable-native-access` flag)
- **OpenSSL**: 3.0 or later (`libcrypto.so.3`)

### OpenSSL Version Compatibility

| OpenSSL Version | Status | Notes |
|-----------------|--------|-------|
| 3.0.x | Supported | Base algorithms |
| 3.1.x | Supported | Full support |
| 3.2.x | Supported | Adds Argon2 KDFs |
| 3.3.x+ | Supported | Latest features |

## Installation

### Maven

```xml
<dependency>
   <groupId>net.glassless</groupId>
   <artifactId>glassless-provider</artifactId>
   <version>0.1-SNAPSHOT</version>
</dependency>
```

### JVM Arguments

The provider requires native access permissions:

```bash
java --enable-native-access=ALL-UNNAMED -jar your-app.jar
```

Or set in `JAVA_TOOL_OPTIONS`:

```bash
export JAVA_TOOL_OPTIONS="--enable-native-access=ALL-UNNAMED"
```

## Usage

### Registering the Provider

```java
import java.security.Security;
import net.glassless.provider.GlaSSLessProvider;

// Option 1: Add provider programmatically
Security.addProvider(new GlaSSLessProvider());

// Option 2: Insert at highest priority
Security.insertProviderAt(new GlaSSLessProvider(), 1);
```

### Basic Examples

#### Message Digest (Hashing)

```java
MessageDigest md = MessageDigest.getInstance("SHA-256", "GlaSSLess");
byte[] hash = md.digest("Hello, World!".getBytes());
```

#### Symmetric Encryption (AES-GCM)

```java
// Generate a key
KeyGenerator keyGen = KeyGenerator.getInstance("AES", "GlaSSLess");
keyGen.init(256);
SecretKey key = keyGen.generateKey();

// Generate IV
byte[] iv = new byte[12];
SecureRandom random = SecureRandom.getInstance("NativePRNG", "GlaSSLess");
random.nextBytes(iv);

// Encrypt
Cipher cipher = Cipher.getInstance("AES_256/GCM/NoPadding", "GlaSSLess");
cipher.init(Cipher.ENCRYPT_MODE, key, new GCMParameterSpec(128, iv));
byte[] ciphertext = cipher.doFinal("Secret message".getBytes());
```

#### Digital Signatures (ECDSA)

```java
// Generate key pair
KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("EC", "GlaSSLess");
keyPairGen.initialize(256);
KeyPair keyPair = keyPairGen.generateKeyPair();

// Sign
Signature signer = Signature.getInstance("SHA256withECDSA", "GlaSSLess");
signer.initSign(keyPair.getPrivate());
signer.update("Data to sign".getBytes());
byte[] signature = signer.sign();

// Verify
Signature verifier = Signature.getInstance("SHA256withECDSA", "GlaSSLess");
verifier.initVerify(keyPair.getPublic());
verifier.update("Data to sign".getBytes());
boolean valid = verifier.verify(signature);
```

#### Ed25519 Signatures

```java
KeyPairGenerator kpg = KeyPairGenerator.getInstance("Ed25519", "GlaSSLess");
KeyPair keyPair = kpg.generateKeyPair();

Signature sig = Signature.getInstance("Ed25519", "GlaSSLess");
sig.initSign(keyPair.getPrivate());
sig.update("Message".getBytes());
byte[] signature = sig.sign();
```

## FIPS Mode

When OpenSSL is configured with FIPS mode enabled, GlaSSLess automatically detects this and excludes non-FIPS-approved algorithms from registration.

```java
GlaSSLessProvider provider = new GlaSSLessProvider();
boolean fipsMode = provider.isFIPSMode();
System.out.println("FIPS Mode: " + (fipsMode ? "ENABLED" : "DISABLED"));
```

## Building from Source

```bash
# Clone the repository
git clone https://github.com/glassless-security/glassless.git
cd glassless

# Build with Maven
mvn clean package

# Run tests
mvn test
```
