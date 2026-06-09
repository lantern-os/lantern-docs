# Identity

In LanternOS the **user owns their identity** — it is rooted in keys the user controls, not
in an account on someone else's server. Identity is a first-class OS concern, built on the
crypto service. Related repositories: [`lantern-crypto`](https://github.com/lantern-os/lantern-crypto) (custody and
primitives) and the identity service that lives alongside the system services.

## Principles applied

- **Self-sovereign.** Identity is a key pair (and its derivations) the user holds, not a
  username an authority grants. No central registrar is required to *have* an identity.
- **Partitionable by default.** A user has *many* identities/personas, and the system makes
  it easy to keep them unlinkable. Linkage is opt-in, never the default — this is how we
  honour [privacy by default](./Principles.md).
- **Minimal disclosure.** Prove what is necessary and nothing more (you are over 18; you are
  a member; you control this key) without revealing who you are.

## Building blocks

### Decentralised Identifiers (DIDs)
A DID is an identifier the user controls cryptographically, resolvable without a central
authority. LanternOS treats DIDs as the canonical handle for an identity/persona. Multiple
DIDs per user is the norm.

### Verifiable Credentials (VCs)
Signed attestations ("X says Y about this DID"). The user collects credentials in their
wallet and presents them selectively. Combined with zero-knowledge proofs, a user can
present *derived* claims (range proofs, set membership) without revealing the full
credential.

### The wallet as an OS service
This is a deliberate inversion of today's world: **wallets are operating-system services,
not browser extensions.** Keys, credentials, and signing live in the OS (ideally hardware
backed), behind a capability-mediated, user-consented interface. An application asks the OS
to sign or present a credential; it never sees the keys. This removes the largest class of
wallet compromise (malicious or buggy extension with key access) by construction. See
[Cryptography](./Cryptography.md).

### Zero-knowledge authentication (direction)
Login without handing over a reusable secret or a trackable identifier — prove control of a
key or possession of a credential without revealing the identifier that links your sessions.

## Identity and capabilities

Identity and authority are **separate concerns** in LanternOS, and keeping them separate is
important:

- **Capabilities** answer "what may this component *do*" (no identity needed — possession is
  authority; see [Security](./Security.md)).
- **Identity** answers "who is the user / persona acting" — used for *cryptographic* claims,
  signing, and consented disclosure, not for ambient access control.

Sealed (cryptographic) capabilities are where the two meet: a capability can be *bound* to a
DID and *attenuated* before delegation, enabling decentralised, identity-aware sharing
without ambient authority.

## Privacy considerations

- The system does **not** mint a global, stable device or user identifier. There is no
  "advertising ID."
- Per-context identities reduce cross-service linkage; the [networking](./Networking.md)
  layer complements this with per-identity network paths.
- Attestation (proving boot state) is powerful but is itself a fingerprint; identity design
  must mediate when and to whom attestation is revealed.

## Open questions

- DID method(s) to support, and how resolution works in a local-first/offline setting.
- Humane, secure **recovery**: losing the root key cannot mean losing your identity forever,
  yet recovery must not become a backdoor. (Social recovery? Hardware tokens? Sharded keys?)
- How to make persona separation usable so users actually keep identities unlinked.
- The trust model for credential issuers and revocation in a decentralised world.
