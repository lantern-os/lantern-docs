# Filesystem

LanternOS storage is **content-addressed, encrypted by default, capability-gated, and
history-preserving**. There is no global filesystem namespace that any process can walk —
data is reached only through capabilities to objects. Repository:
[`lantern-filesystem`](https://github.com/lantern-os/lantern-filesystem).

## Why not a Unix filesystem

A hierarchical, ambient, mutable filesystem with path-and-permission access is one of the
biggest sources of ambient authority in conventional systems ("the app can read `~/.ssh`").
It also conflates *naming* with *authority* and *location* with *identity*. LanternOS
separates these.

## Core ideas

### Content addressing (CAS)
Data is named by the cryptographic hash of its content. The same bytes have the same name
everywhere; tampering changes the name; deduplication is free. This gives **integrity by
construction** and is the natural substrate for [decentralised networking](./Networking.md)
(content can be fetched from any peer and verified).

### Encrypted by default
All persistent data is encrypted at rest with keys held by the [crypto service](./Cryptography.md),
bound where possible to a hardware root of trust. Device theft yields ciphertext. Encryption
is not an opt-in feature; plaintext-at-rest is not an offered mode.

### Capability-gated access
You read or write an object because you hold a capability to it, not because you can name a
path. "Open a file by capability, not by path" eliminates path-based ambient access and
TOCTOU-by-path bugs. A directory is itself just an object granting capabilities to its
entries.

### Immutable history & provenance
The store is **append-friendly and history-preserving**: updates create new versions rather
than destroying old ones, enabling snapshots, time travel, and audit. Each version can carry
**provenance** — which component/agent wrote it, under which capability — which is essential
for trusting (or distrusting) [AI agent](./AI.md) outputs.

### Snapshots
Cheap, consistent point-in-time views, naturally enabled by content addressing and immutable
history. The basis for backup, rollback, and reproducibility.

## Sketch of the layering

```
   names & directories (capability objects)        ← what apps see
        │
   versioned object model (history, provenance)
        │
   encryption layer (per-object keys via crypto svc)
        │
   content-addressed block store (hash-named blocks, dedup)
        │
   device drivers / block devices (user-space, confined)
```

The whole stack is a **confined user-space service**: a bug in the filesystem cannot reach
the kernel or other services, only the data it was given capabilities to.

## Trade-offs (documented honestly)

| Choice | Benefit | Cost |
| --- | --- | --- |
| Content addressing | Integrity, dedup, P2P-friendly | Mutation = rewrite + re-link; GC complexity |
| Encrypted by default | Confidentiality on loss | Key management is now critical-path; no plaintext recovery without keys |
| Immutable history | Snapshots, audit, provenance | Storage growth; needs compaction/GC and retention policy |
| Capability access | No ambient authority | Need ergonomic ways to *grant* access (UX challenge) |
| No global namespace | Eliminates path-based attacks | Familiar tools/workflows must be rethought |

These trade-offs are why this is documented as a *vision* with open questions, not yet built.

## Relationship to other layers

- [Cryptography](./Cryptography.md): keys, AEAD, content hashing.
- [Networking](./Networking.md): content-addressed blocks sync naturally between peers.
- [AI](./AI.md): provenance lets the system attribute and audit what an agent wrote.
- [Capabilities](./Security.md): file/directory objects are capability objects.

## Open questions

- Garbage collection and retention for an immutable, deduplicated, encrypted store.
- Efficient mutation/large-file/random-write patterns over a CAS.
- Searchable encryption vs. local plaintext indices (and their own confidentiality).
- How key rotation interacts with long-lived encrypted history (re-encrypt vs. key-wrapping).
- Sync/conflict model for local-first multi-device editing (CRDTs?).
