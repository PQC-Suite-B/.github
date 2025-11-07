# PQC Suite B: Faster Post-Quantum Cryptography with BLAKE3

**Post-quantum signatures** spend a large share of their runtime on
hashing. We propose replacing their hash functions with
[BLAKE3](https://github.com/BLAKE3-team/BLAKE3/), the fastest
widely deployed cryptographic hash.

Authors:

- JP Aumasson / [@veorq](https://x.com/veorq), [Taurus](https://taurushq.com)
- Conor Deegan / [@conor-deegan](https://x.com/ConorDeegan4), [Project 11](https://www.projecteleven.com/)
- Alex Pruden / [@apruden08](https://x.com/apruden08), [Project 11](https://www.projecteleven.com/)
- Zooko Wilcox-O'Hearn / [@zooko](https://x.com/zooko), [Zcash](https://z.cash/)

## Why BLAKE3?

BLAKE3 is already integrated across diverse systems, from blockchains to
video games. It can serve as a drop-in replacement for any hashing
mode: regular hashing, keyed hashing (PRF), key derivation functions
(KDF), or extensible output functions (XOF).

BLAKE3 outperforms the SHA-2 and SHA-3 families and is even competitive
with hardware-accelerated SHA-256, as
[benchmarks](https://bench.cr.yp.to/primitives-hash.html) show.

Built on two decades of cryptanalytic scrutiny, BLAKE3 descends from
BLAKE (designed in 2008 as a SHA3 candidate), which itself derives from
ChaCha (a variant of the 2005 Salsa20 cipher).

Switching to BLAKE3 does not weaken security.

## Faster ML-DSA

[ML-DSA (FIPS 204)](https://csrc.nist.gov/pubs/fips/204/final) is
a lattice-based post-quantum signature scheme based on
[Dilithium](https://pq-crystals.org/dilithium/). It relies on the
SHA3-based functions SHAKE128 and SHAKE256.

We replace those with BLAKE3 and call the new scheme **ML-DSA-B**.

We ran [experimental
benchmarks](https://github.com/PQC-Suite-B/signatures/blob/b3/ml-dsa/HASHES.md),
modifying RustCrypto's ML-DSA with the [reference BLAKE3 Rust
code](https://crates.io/crates/blake3). Preliminary results show that,
depending on the platform, ML-DSA-B can offer the following speed-up:

1. Message pre-hash: up to 60 times faster.
2. Signature: up to 20% faster.
3. Verification: up to 30% faster.

We also include test vectors for ML-DSA-B.

### Apple M3 Results

<p float="middle">
    <img src="out/individual/apple_ml_dsa_64B_sign.png" width="49%" />
    <img src="out/individual/apple_ml_dsa_64B_verify.png" width="49%" />
</p>

<p float="middle">
    <img src="out/individual/apple_ml_dsa_1MiB_sign.png" width="49%" />
    <img src="out/individual/apple_ml_dsa_100MiB_sign.png" width="49%" />
</p>

### Cloud VM (x86_64)

<p float="middle">
    <img src="out/individual/cloud_ml_dsa_64B_sign.png" width="49%" />
    <img src="out/individual/cloud_ml_dsa_64B_verify.png" width="49%" />
</p>

<p float="middle">
    <img src="out/individual/cloud_ml_dsa_1MiB_sign.png" width="49%" />
    <img src="out/individual/cloud_ml_dsa_100MiB_sign.png" width="49%" />
</p>

## Faster SLH-DSA

[SLH-DSA (FIPS 205)](https://csrc.nist.gov/pubs/fips/205/final) is
a hash-based post-quantum signature scheme based on
[SPHINCS+](https://sphincs.org/). It has two variants: one using SHA-256 and one using SHAKE.

We replace those with BLAKE3 and call the new scheme **SLH-DSA-B**.

We ran [experimental
benchmarks](https://github.com/PQC-Suite-B/signatures/blob/b3/slh-dsa/HASHES.md),
modifying RustCrypto's SLH-DSA with the [reference BLAKE3 Rust
code](https://crates.io/crates/blake3). Preliminary results show that,
depending on the platform, SLH-DSA-B can offer the following speed-up:

1. SHAKE is the slowest choice in all benchmarks (4–7× slower) because of its higher per-bit hashing cost.
2. BLAKE3 and SHA2 are in a similar performance range; the faster one depends on hardware.
3. Architecture effects dominate: x86 favors BLAKE3 (SIMD parallelism), Apple M3 favors SHA2 (hardware SHA extensions).

We also include test vectors for SLH-DSA-B.

### Apple M3 Results

#### SLH-DSA-128-S

SLH-DSA `s` variants use smaller, slower parameter sets that trade performance for reduced signature size and stronger security margins. On Apple M3, SHA2 slightly outperforms BLAKE3 in these variants due to the chip's dedicated SHA acceleration, while both are several times faster than SHAKE.

<p float="middle">
    <img src="out/individual/apple_slh_128s_64B_sign.png" width="49%" />
    <img src="out/individual/apple_slh_128s_64B_verify.png" width="49%" />
</p>

<p float="middle">
    <img src="out/individual/apple_slh_128s_1MiB_sign.png" width="49%" />
    <img src="out/individual/apple_slh_128s_100MiB_sign.png" width="49%" />
</p>

#### SLH-DSA-128-F

SLH-DSA `f` variants use larger, faster parameter sets optimized for signing and verification speed at the cost of larger signatures. On Apple M3, the same pattern holds: SHA2 remains the fastest, BLAKE3 close behind, and SHAKE significantly slower.

<p float="middle">
    <img src="out/individual/apple_slh_128f_64B_sign.png" width="49%" />
    <img src="out/individual/apple_slh_128f_64B_verify.png" width="49%" />
</p>

<p float="middle">
    <img src="out/individual/apple_slh_128f_1MiB_sign.png" width="49%" />
    <img src="out/individual/apple_slh_128f_100MiB_sign.png" width="49%" />
</p>

### Cloud VM (x86_64)

#### SLH-DSA-128-S

SLH-DSA `s` variants use smaller, more conservative parameters that prioritize compact signatures over raw speed. On x86_64, BLAKE3 performs best thanks to its SIMD-parallel hash design, while SHA2 trails slightly and SHAKE remains the slowest by a large margin.

<p float="middle">
    <img src="out/individual/cloud_slh_128s_64B_sign.png" width="49%" />
    <img src="out/individual/cloud_slh_128s_64B_verify.png" width="49%" />
</p>

<p float="middle">
    <img src="out/individual/cloud_slh_128s_1MiB_sign.png" width="49%" />
    <img src="out/individual/cloud_slh_128s_100MiB_sign.png" width="49%" />
</p>

#### SLH-DSA-128-F

SLH-DSA `f` variants use larger parameter sets tuned for faster operation at the cost of bigger signatures. On x86_64, the relative ordering is consistent: BLAKE3 is the fastest, SHA2 close behind, and SHAKE several times slower due to its higher per-bit hashing cost.

<p float="middle">
    <img src="out/individual/cloud_slh_128f_64B_sign.png" width="49%" />
    <img src="out/individual/cloud_slh_128f_64B_verify.png" width="49%" />
</p>

<p float="middle">
    <img src="out/individual/cloud_slh_128f_1MiB_sign.png" width="49%" />
    <img src="out/individual/cloud_slh_128f_100MiB_sign.png" width="49%" />
</p>

## Independent results

- By @itzmeanjan:
  - ML-DSA-B [C++ version](https://github.com/itzmeanjan/ml-dsa/tree/ml-dsa-b)
  - [Announcement](https://x.com/meanjanroy/status/1980955869178413544): "Keygen is 20% faster. Signing is 76% faster. Verify is 18% faster."

## Upcoming work

We plan to:

- This fork is temporarily pinned to an earlier [RustCrypto commit](https://github.com/RustCrypto/signatures/commit/f6df3e250c7634bdb72bb2f11e3a4f142be06678). We intend to re-sync with upstream (RustCrypto/signatures) to incorporate the latest ML-DSA changes.

- Evaluate BLAKE3's impact on other post-quantum standards and candidates, including KEMs and NIST's [Additional](https://csrc.nist.gov/Projects/pqc-dig-sig/round-2-additional-signatures) signature schemes.
