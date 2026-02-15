---
title: FIPS Compliance
description: FIPS 140-3 compliance and supported cryptographic standards
layout: layouts/page
---

GlaSSLess provides comprehensive FIPS compliance through OpenSSL's FIPS-validated cryptographic module. When FIPS mode is enabled, non-approved algorithms are automatically excluded from registration.

## FIPS Mode Detection

GlaSSLess detects FIPS mode automatically through multiple mechanisms:

1. **Java System Property**: `-Dglassless.fips.mode=true`
2. **OpenSSL FIPS Provider**: Detects if OpenSSL has FIPS enabled
3. **System Crypto Policy**: Reads `/etc/crypto-policies/state/current` (RHEL/Fedora)

```java
import net.glassless.provider.GlaSSLessProvider;
import net.glassless.provider.FIPSStatus;

// Check FIPS status
boolean fipsEnabled = FIPSStatus.isFIPSEnabled();
boolean fipsAvailable = FIPSStatus.isFIPSProviderAvailable();

// Get detailed status
String status = FIPSStatus.getStatusDescription();

// Check provider's FIPS mode
GlaSSLessProvider provider = new GlaSSLessProvider();
System.out.println("FIPS Mode: " + (provider.isFIPSMode() ? "ENABLED" : "DISABLED"));
```

## Supported FIPS Standards

### FIPS 140-3: Cryptographic Module Validation

GlaSSLess delegates cryptographic operations to OpenSSL's FIPS-validated module, providing FIPS 140-3 Level 1 compliance when properly configured.

### FIPS 180-4: Secure Hash Standard (SHS)

| Algorithm | Status |
|-----------|--------|
| SHA-224, SHA-256, SHA-384, SHA-512 | Approved |
| SHA-512/224, SHA-512/256 | Approved |
| SHA-1 | **Deprecated** - excluded in FIPS mode |
| MD5 | **Not Approved** - excluded in FIPS mode |

### FIPS 197: Advanced Encryption Standard (AES)

| Algorithm | Key Sizes | Modes | Status |
|-----------|-----------|-------|--------|
| AES | 128, 192, 256 | ECB, CBC, CFB, CTR, OFB, GCM, CCM, XTS | Approved |
| AES Key Wrap | 128, 192, 256 | KW, KWP | Approved |

### FIPS 198-1: HMAC

| Algorithm | Status |
|-----------|--------|
| HmacSHA224, HmacSHA256, HmacSHA384, HmacSHA512 | Approved |
| HmacSHA3-224, HmacSHA3-256, HmacSHA3-384, HmacSHA3-512 | Approved |
| AESCMAC, AESGMAC | Approved |
| KMAC128, KMAC256 | Approved |
| HmacSHA1 | **Deprecated** - excluded in FIPS mode |

### FIPS 186-5: Digital Signature Standard (DSS)

| Algorithm | Status |
|-----------|--------|
| RSA (SHA256/384/512withRSA) | Approved |
| RSA-PSS (SHA256/384/512withRSAandMGF1) | Approved |
| ECDSA (SHA256/384/512withECDSA) | Approved |
| ECDSA with SHA3 | Approved |
| DSA (SHA256/384/512withDSA) | Approved |
| EdDSA (Ed25519, Ed448) | Approved |
| SHA1-based signatures | **Deprecated** - excluded in FIPS mode |

### FIPS 202: SHA-3 Standard

| Algorithm | Status |
|-----------|--------|
| SHA3-224, SHA3-256, SHA3-384, SHA3-512 | Approved |
| SHAKE128, SHAKE256 | Approved |

### FIPS 203: ML-KEM (Post-Quantum)

*Requires OpenSSL 3.5+*

| Algorithm | Security Level | Status |
|-----------|----------------|--------|
| ML-KEM-512 | 128-bit | Approved |
| ML-KEM-768 | 192-bit | Approved |
| ML-KEM-1024 | 256-bit | Approved |

### FIPS 204: ML-DSA (Post-Quantum)

*Requires OpenSSL 3.5+*

| Algorithm | Security Level | Status |
|-----------|----------------|--------|
| ML-DSA-44 | 128-bit | Approved |
| ML-DSA-65 | 192-bit | Approved |
| ML-DSA-87 | 256-bit | Approved |

### FIPS 205: SLH-DSA (Post-Quantum)

*Requires OpenSSL 3.5+*

| Algorithm | Variants | Status |
|-----------|----------|--------|
| SLH-DSA-SHA2-128 | s (small), f (fast) | Approved |
| SLH-DSA-SHA2-192 | s, f | Approved |
| SLH-DSA-SHA2-256 | s, f | Approved |
| SLH-DSA-SHAKE-128 | s, f | Approved |
| SLH-DSA-SHAKE-192 | s, f | Approved |
| SLH-DSA-SHAKE-256 | s, f | Approved |

## Algorithms Excluded in FIPS Mode

The following algorithms are **not available** when FIPS mode is enabled:

### Message Digests
- MD5
- SHA-1
- SM3, RIPEMD160

### Ciphers
- ChaCha20, ChaCha20-Poly1305
- DESede (3DES)
- Camellia, ARIA, SM4

### MACs
- HmacSHA1
- Poly1305, SipHash

### Signatures
- SHA1withRSA, SHA1withECDSA, SHA1withDSA

### Key Derivation
- SCRYPT
- Argon2 (all variants)
- HKDF-SHA1, PBKDF2WithHmacSHA1

## Enabling FIPS Mode

### Option 1: System Property

```bash
java -Dglassless.fips.mode=true \
     --enable-native-access=ALL-UNNAMED \
     -jar your-app.jar
```

### Option 2: OpenSSL Configuration

Configure OpenSSL to use the FIPS provider by default in `/etc/ssl/openssl.cnf`:

```ini
[openssl_init]
providers = provider_sect

[provider_sect]
fips = fips_sect
base = base_sect

[fips_sect]
activate = 1

[base_sect]
activate = 1
```

### Option 3: RHEL/Fedora Crypto Policy

```bash
# Enable system-wide FIPS mode
sudo fips-mode-setup --enable
sudo reboot
```

## Verifying FIPS Compliance

```java
import net.glassless.provider.GlaSSLessProvider;
import net.glassless.provider.FIPSStatus;
import java.security.*;

// Register provider
GlaSSLessProvider provider = new GlaSSLessProvider();
Security.addProvider(provider);

// Verify FIPS mode
if (provider.isFIPSMode()) {
    System.out.println("Running in FIPS mode");

    // These will work
    MessageDigest sha256 = MessageDigest.getInstance("SHA-256", "GlaSSLess");
    Signature ecdsa = Signature.getInstance("SHA256withECDSA", "GlaSSLess");

    // This will throw NoSuchAlgorithmException
    try {
        MessageDigest md5 = MessageDigest.getInstance("MD5", "GlaSSLess");
    } catch (NoSuchAlgorithmException e) {
        System.out.println("MD5 correctly excluded in FIPS mode");
    }
}
```

## FIPS Compliance Checklist

| Requirement | GlaSSLess Support |
|-------------|-------------------|
| FIPS 140-3 validated module | Via OpenSSL FIPS provider |
| Automatic FIPS detection | Yes |
| Non-approved algorithm exclusion | Yes |
| Post-quantum FIPS standards | FIPS 203, 204, 205 |
| Runtime FIPS status query | Yes |
| System property override | Yes |

## OpenSSL FIPS Provider Requirements

| OpenSSL Version | FIPS Module | Notes |
|-----------------|-------------|-------|
| 3.0.x | FIPS 140-2 | Base FIPS support |
| 3.1.x - 3.4.x | FIPS 140-2 | Enhanced algorithm support |
| 3.5.x+ | FIPS 140-3 | Post-quantum cryptography |

For full FIPS compliance, ensure:
1. OpenSSL is built with FIPS module support
2. FIPS provider is properly configured and activated
3. GlaSSLess detects FIPS mode (verify with `FIPSStatus.isFIPSEnabled()`)
