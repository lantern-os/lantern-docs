# LanternOS Wiki

Welcome. This wiki is the shared mental model of LanternOS. If you read nothing else, read
the [Vision](./Vision.md), the [Principles](./Principles.md), and the
[Threat Model](./Threat-Model.md) — everything else follows from those three.

## What LanternOS is

A clean-slate, AI-native, privacy-first operating system built on a capability-based
microkernel, memory-safe languages, decentralised networking, and open hardware. It asks
what an OS would be if designed today rather than inheriting the assumptions of the 1980s.

## What LanternOS is not

Not a Linux distribution. Not a Unix clone. Not a Windows replacement. Not a browser OS.
We will borrow good ideas from all of them, but we are not bound by their compatibility
constraints.

## The map

### Foundations
- **[Vision](./Vision.md)** — the thesis and the bet.
- **[Intent Model](./Intent-Model.md)** — the long-term product vision built on that bet.
- **[Principles](./Principles.md)** — six non-negotiables.
- **[Threat Model](./Threat-Model.md)** — assets, adversaries, boundaries.
- **[Architecture](./Architecture.md)** — the layered system.

### Layers
- **[Kernel](./Kernel.md)** — the microkernel and the TCB.
- **[Security](./Security.md)** — capabilities, isolation, zero trust.
- **[Cryptography](./Cryptography.md)** — primitives, key management, agility.
- **[Identity](./Identity.md)** — DIDs, credentials, wallets as OS services.
- **[Filesystem](./Filesystem.md)** — content-addressed, encrypted, capability-gated.
- **[Networking](./Networking.md)** — decentralised, anonymity-capable.
- **[Runtime](./Runtime.md)** — services, WASM, the component model.
- **[AI](./AI.md)** — local inference, capability-scoped agents, audit.
- **[Hardware](./Hardware.md)** — RISC-V, enclaves, accelerators, the future board.

### Direction
- **[Roadmap](./Roadmap.md)** — phases from documentation to silicon.

## How to use this wiki

- Decisions that fix a trust boundary link to an **RFC/ADR** in
  [`lantern-rfcs`](https://github.com/lantern-os/lantern-rfcs). The wiki describes; the RFCs decide.
- Each page ends with **Open questions** — these are invitations to contribute.
- If a page and a repo's `ARCHITECTURE.md` disagree, that is a bug; file an issue.

## Project status

**Phase 3 — Privacy, identity, networking, and AI.** Phase 2 (capability runtime & first
services) is complete, closed by
[RFC-0017](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0017-phase-2-to-phase-3-transition.md)/[ADR-0021](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0021-phase-2-complete-phase-3-opened.md):
a third-party Wasm app runs confined and cannot touch anything it wasn't granted,
demonstrated adversarially. Carried forward as Phase 3's first work — the services and the
runtime run in-process on a host target, not yet confined on the kernel. See the
[Roadmap](./Roadmap.md).
