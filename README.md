# lantern-docs

The canonical documentation for LanternOS. The [`wiki/`](./wiki) directory is the project
wiki, written to onboard contributors and future maintainers. It is the *descriptive*
counterpart to the *decision* records in [`lantern-rfcs`](https://github.com/lantern-os/lantern-rfcs): the wiki tells
you what the architecture **is**; the RFCs/ADRs tell you **why** and govern how it changes.

## Reading order

1. [Home](./wiki/Home.md) — the map.
2. [Vision](./wiki/Vision.md) — what we are building and why.
3. [Principles](./wiki/Principles.md) — the non-negotiables that constrain every decision.
4. [Threat Model](./wiki/Threat-Model.md) — who we defend against and what we protect.
5. [Architecture](./wiki/Architecture.md) — the layered system and how it fits together.

Then dive into the layer pages: [Kernel](./wiki/Kernel.md),
[Runtime](./wiki/Runtime.md), [Security](./wiki/Security.md),
[Hardware](./wiki/Hardware.md), [AI](./wiki/AI.md), [Networking](./wiki/Networking.md),
[Filesystem](./wiki/Filesystem.md), [Identity](./wiki/Identity.md),
[Cryptography](./wiki/Cryptography.md), and the [Roadmap](./wiki/Roadmap.md).

## How this wiki relates to the code repos

Each architectural repository carries its own `ARCHITECTURE.md` (the design *of that
component*) and `THREAT_MODEL.md` (the threats *to that component*). The wiki here holds the
**system-wide** view. When the two disagree, file an issue — the discrepancy is itself a bug.

## Conventions

- British English; present tense for current design, future tense for roadmap items.
- Every design claim that fixes a trust boundary links to the RFC/ADR that decided it.
- Diagrams are ASCII so they diff cleanly in review.
