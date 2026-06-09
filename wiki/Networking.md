# Networking

LanternOS networking is **decentralised, peer-capable, and anonymity-aware**. The network is
treated as fully hostile (see [Threat Model](./Threat-Model.md)), and the user — not an ISP,
platform, or central server — controls how they appear on it. Repository:
[`lantern-network`](https://github.com/lantern-os/lantern-network).

## Stance

- **The network is untrusted.** Confidentiality, integrity, and authenticity are
  responsibilities of the endpoints, established cryptographically, never assumed from the
  transport.
- **Decentralised by default.** Peer-to-peer and local-first; central servers are optional
  peers, not required intermediaries. Pairs with the [local-first](./Principles.md)
  principle.
- **Privacy of metadata, not just content.** Encrypting payloads is table stakes; LanternOS
  also targets the *metadata* layer — who talks to whom, when, and how much — which is where
  most real-world surveillance happens.
- **Multiple network identities.** An application can present **different network identities
  for different contexts**, and the system makes those identities hard to correlate. This is
  the network-layer expression of partitionable [identity](./Identity.md).

## Building blocks (research/direction)

| Capability | Approach under study | Prior art |
| --- | --- | --- |
| Authenticated encrypted channels | Modern AEAD + key exchange from the crypto service | Noise Protocol, TLS 1.3, WireGuard |
| Peer addressing & discovery | Content/identity-addressed, DHT-based | libp2p, IPFS, Kademlia |
| Anonymous routing | Onion routing for sender/receiver unlinkability | Tor |
| Metadata resistance | Mixnets (batching, cover traffic) | Nym, Loopix |
| Censorship resistance | Pluggable transports, peer relays | Tor bridges |
| Local connectivity | Mesh / ad-hoc peer networking | Briar, Yggdrasil |
| Content distribution | Content-addressed block exchange | IPFS/Bitswap |

LanternOS does not aim to reinvent these; it aims to *integrate the right ones as OS services*
so that privacy-preserving networking is the default an app gets, not a hard thing each app
must reimplement.

## How it fits the architecture

- The network stack is a **confined user-space service** ([Architecture](./Architecture.md)).
  A compromise of the network stack is bounded by its capabilities — notably, it does **not**
  hold the keystore.
- Apps obtain **socket/endpoint capabilities** for specific destinations or network
  identities; they cannot open arbitrary connections ambiently.
- Network identities are backed by keys in the [crypto service](./Cryptography.md) and
  related to (but separable from) [DIDs](./Identity.md).
- Content addressing shared with the [filesystem](./Filesystem.md) makes peer content
  exchange verifiable.

## The metadata problem (why this is hard and important)

Even with perfect payload encryption, an observer who sees *that* you connected to a service,
*when*, and *how often* learns a great deal. True metadata protection (mixnets, cover
traffic) is expensive in latency and bandwidth. LanternOS's position: offer a **spectrum** —
from fast/authenticated (Noise-style) to slow/metadata-resistant (mixnet) — and let the user
or the granting capability choose the trade-off per context, rather than pretending one point
on the curve fits everything.

## Trade-offs

- **Anonymity vs. performance:** onion routing and mixnets add latency; not every connection
  needs them. Selectable per capability/context.
- **Decentralisation vs. reliability/UX:** P2P discovery and NAT traversal are harder than
  "connect to a server." Optional relays help without re-centralising trust.
- **Anonymity vs. abuse:** strong anonymity complicates abuse handling; this is a known,
  open social/technical tension we will document rather than hand-wave.

## Open questions

- Which transport(s) form the mandatory baseline vs. optional modules?
- How to make per-context network identities usable and *actually* unlinkable in practice.
- NAT traversal and discovery in a privacy-preserving way.
- Integrating a mixnet without owning/operating central infrastructure.
- Offline/mesh operation and store-and-forward for intermittent connectivity.
