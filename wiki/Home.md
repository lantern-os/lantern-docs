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

**Phase 4 — Hardware and assurance.** Phase 3 (privacy, identity, networking, and AI) is complete, closed by
[RFC-0036](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0036-phase-3-to-phase-4-transition.md)/[ADR-0041](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0041-phase-3-complete-phase-4-opened.md). Its exit criteria are met:

- data at rest is encrypted and records who wrote it;
- a local AI agent runs with a visible, revocable capability set and a faithful audit log;
- an app presents each network peer an identity no other context's can be linked to.

Phase 4 begins with formal verification of the kernel's capability and IPC paths, then moves onto real RISC-V hardware:
a hardware root of trust, measured boot, and IOMMU-confined drivers. See the
[Roadmap](./Roadmap.md).
