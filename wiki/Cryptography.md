# Cryptography

Cryptography in LanternOS is an **operating-system service**, not a library each app links
and misuses. The system owns key custody, signing, encryption, and identity primitives so
that applications and agents *use* cryptographic authority without ever *holding* raw keys.
Repository: [`lantern-crypto`](https://github.com/lantern-os/lantern-crypto).

## Design stance

1. **Keys never leave the keystore in the clear.** Apps get *capabilities to operations*
   ("sign with key K", "decrypt for context C"), not key material. The private key stays in
   the crypto service — ideally in a hardware enclave ([Hardware](./Hardware.md)).
2. **Hardware-backed where possible.** Signing and key custody bind to a hardware root of
   trust; software-only is a fallback, clearly marked as lower assurance.
3. **Misuse-resistant APIs.** No "encrypt(key, iv)" foot-guns. We expose high-level,
   hard-to-misuse operations (sealed boxes, authenticated channels) à la libsodium/age, not
   raw primitives.
4. **Crypto-agility by default.** Every key, ciphertext, signature, and credential is
   **versioned and algorithm-tagged** so primitives can be rotated without breaking stored
   data — essential for a system meant to last decades.
5. **Post-quantum readiness.** We assume a "harvest now, decrypt later" adversary for
   long-lived secrets and plan **hybrid** classical+PQC for key exchange and signatures.

## Primitive selection (initial, subject to RFC)

Primitive *choices* are governance-gated: any change requires an RFC
([GOVERNANCE](https://github.com/lantern-os/.github/blob/main/GOVERNANCE.md)). A conservative, modern starting set:

| Purpose | Initial choice | Notes |
| --- | --- | --- |
| Hashing / content addressing | BLAKE3 (and SHA-256 where interop needed) | Fast, parallel; used by the filesystem CAS. |
| AEAD (symmetric) | XChaCha20-Poly1305 (AES-256-GCM where HW-accelerated) | Large nonces avoid reuse pitfalls. |
| KDF / password | HKDF; Argon2id for passwords | |
| Signatures | Ed25519 | Plus an ML-DSA (Dilithium) hybrid track for PQC. |
| Key exchange | X25519 | Plus an ML-KEM (Kyber) hybrid track for PQC. |
| Randomness | OS CSPRNG seeded from hardware RNG | Health-checked; never app-supplied. |

These are **defaults, not dogma** — recorded as an ADR once RFC'd, and revisable as the
field moves.

## What the crypto service provides

- **Keystore & key lifecycle:** generation, storage, rotation, destruction; per-key policy
  (e.g. "requires user presence", "non-exportable").
- **Signing & verification:** as a capability, with optional hardware confirmation.
- **Sealed encryption:** encrypt-to-context / encrypt-to-identity without exposing keys.
- **Sealed capabilities:** the cryptographic form of LanternOS capabilities (macaroon-style,
  attenuable, revocable) used for delegation across machines — ties
  [Security](./Security.md) to [Networking](./Networking.md) and [Identity](./Identity.md).
- **Attestation:** statements about the device's measured-boot state, signed by a
  hardware-rooted key.
- **Zero-knowledge primitives (direction):** for authentication that proves a claim without
  revealing the underlying secret or identifier — see [Identity](./Identity.md).

## Relationship to other layers

- [Filesystem](./Filesystem.md): encryption-at-rest and content addressing use crypto.
- [Identity](./Identity.md): DIDs, verifiable credentials, and wallets are built on the
  keystore and signing service.
- [Networking](./Networking.md): authenticated, encrypted channels and onion routing use
  the same primitives and key custody.

## Assurance and `unsafe`

We prefer **audited, well-reviewed implementations** over rolling our own. Constant-time
discipline, no secret-dependent branching, zeroisation of secrets, and side-channel
awareness are requirements, not nice-to-haves. Where we must touch hardware crypto, the
`unsafe` is isolated and reviewed (ADR-0001).

## Open questions

- Exact hybrid-PQC construction and when to make it the default vs. optional.
- Key backup/recovery that is both secure and humane (social recovery? hardware tokens?).
- How attestation interacts with privacy — attestation can be a fingerprint; how do we offer
  it without enabling tracking?
- Whether to expose any low-level primitives to apps at all, or *only* high-level operations.
