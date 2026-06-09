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

**Phase 0 — Foundations.** Architecture and documentation only. See the [Roadmap](./Roadmap.md).
