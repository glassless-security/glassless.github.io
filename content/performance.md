---
title: Performance
description: Performance benchmarks comparing GlaSSLess with JDK implementations
layout: layouts/page
---

GlaSSLess provides significant performance advantages for asymmetric cryptography operations while JDK implementations excel at small-data symmetric operations due to HotSpot intrinsics.

## Where GlaSSLess Excels

| Operation | Typical Speedup |
|-----------|-----------------|
| ECDH Key Agreement | ~6x faster |
| Ed25519 Signing | ~8x faster |
| Ed25519 Verification | ~5-6x faster |
| EC Key Generation (P-256/P-384) | ~2-3x faster |
| Ed25519/X25519 Key Generation | ~2-5x faster |
| SecureRandom (large buffers) | ~10-25x faster |
| SHA3/SHAKE (large data) | ~1.5-2x faster |

## Where JDK Excels

| Operation | JDK Advantage |
|-----------|---------------|
| SHA-256/SHA-512 (small data <1KB) | ~4-6x faster |
| HMAC (small data <1KB) | ~4-8x faster |
| SecureRandom (small buffers <64B) | ~2x faster |

For large data (16KB+), symmetric operation performance converges between implementations.

## Benchmarks

JMH microbenchmarks compare performance between JDK and GlaSSLess implementations.

### Running Benchmarks

```bash
# Run all benchmarks
mvn test -Pbenchmarks

# Generate detailed performance report
./scripts/generate-performance-report.sh
```

### Benchmark Categories

| Benchmark | Algorithms | Data Sizes |
|-----------|------------|------------|
| MessageDigestBenchmark | SHA-256, SHA-512, SHA3-256 | 64B, 1KB, 16KB, 1MB |
| CipherBenchmark | AES/GCM, AES/CBC, ChaCha20-Poly1305 | 64B, 1KB, 16KB, 1MB |
| MacBenchmark | HmacSHA256, HmacSHA512, HmacSHA3-256 | 64B, 1KB, 16KB, 1MB |
| SignatureBenchmark | SHA256withECDSA, SHA384withECDSA, Ed25519 | - |
| KeyAgreementBenchmark | ECDH, X25519 | - |
| KeyPairGeneratorBenchmark | EC P-256/P-384, RSA-2048/4096, Ed25519, X25519 | - |
| SecureRandomBenchmark | NativePRNG | 16B to 4KB |

Results are saved to `target/jmh-results.json`.

## Recommendations

### Use GlaSSLess for:

- **Ed25519/Ed448 signatures** - Significant speedup (5-8x)
- **X25519/X448 key agreement** - Excellent performance gains
- **ECDH with P-256/P-384** - Substantial improvement
- **Large random number generation** - Much faster for buffers >256 bytes
- **SHA3/SHAKE hashing** - Better OpenSSL implementation

### Use JDK for:

- **Small data hashing (<1KB)** - HotSpot intrinsics are highly optimized
- **Small HMAC operations** - JDK is faster for small inputs
- **Tiny random numbers** - Lower overhead for small requests

### Hybrid Approach

For optimal performance, register GlaSSLess at a low priority and explicitly request it for operations where it excels:

```java
// Register at low priority
Security.addProvider(new GlaSSLessProvider());

// Explicitly use GlaSSLess for Ed25519
Signature sig = Signature.getInstance("Ed25519", "GlaSSLess");

// Let JDK handle SHA-256 for small data
MessageDigest md = MessageDigest.getInstance("SHA-256");
```
