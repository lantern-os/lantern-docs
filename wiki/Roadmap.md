# Roadmap

LanternOS is a multi-year — possibly multi-decade — project. This roadmap is organised by
**phase**, not by date, because the gate between phases is *quality of foundation*, not the
calendar. Each phase has explicit **exit criteria**; we do not advance until they are met.

The guiding order is the [architecture-first](./Architecture.md#architecture-first) sequence:
**documentation → architecture → RFCs → prototypes → implementation.**

---

## Phase 0 — Foundations  *(complete — closed by [RFC-0004](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0004-phase-0-to-phase-1-transition.md)/[ADR-0007](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0007-phase-0-complete-phase-1-opened.md))*

**Goal:** establish the principles, architecture, threat models, governance, and decision
process. No production code.

- [x] Organisation structure and the 14 repositories.
- [x] Core principles, vision, and system threat model.
- [x] Layered architecture documented end to end.
- [x] RFC + ADR process and governance model.
- [x] Seed RFCs (microkernel, capability model) and ADRs (Rust, RISC-V, Wasm).
- [x] Per-component `THREAT_MODEL.md` reviewed by domain stewards (all 11 components;
      `lantern-rfcs`, `lantern-docs`, and `lantern-website` are process/docs repos and
      own no architecture layer, hence no `ARCHITECTURE.md`/`THREAT_MODEL.md`).
- [x] Per-component `ARCHITECTURE.md` reviewed by domain stewards (all 11 components; fixed
      a "TCB" terminology collision in `lantern-kernel` and a missing auditability
      invariant in `lantern-capabilities`, cross-linked ADR-0004/0005/0006).
- [x] Open foundational RFCs (RFC-0002, RFC-0003) advanced to *Accepted* (see
      [ADR-0004](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0004-kernel-responsibilities-and-tcb-boundary.md),
      [ADR-0005](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0005-object-capabilities-as-universal-authority-model.md),
      [ADR-0006](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0006-three-layer-capability-structure.md)).

**Exit criteria:** the architecture is coherent, the threat model is agreed, and the open
foundational RFCs are accepted. A new contributor can understand the *why* of the system from
the docs alone.

---

## Phase 1 — Microkernel prototype  *(current)*

**Goal:** prove the core mechanisms in throwaway/experimental code.

- Boot to a minimal kernel on `riscv64`/x86-64 under QEMU (`lantern-boot`, `lantern-hal`).
- Address spaces, threads, and a scheduler.
- IPC fast-path (endpoints + notifications) with benchmarks.
- Kernel capability mechanism (CSpace, untyped retyping) — the RFC-0003 kernel layer.
- The "narrowing-waterfall" root task starting one trivial user-space service.

**Exit criteria:** a confined user-space "hello service" reachable only via a granted
capability, with IPC latency benchmarked and within target. Prototype code is explicitly
allowed to be thrown away.

---

## Phase 2 — Capability runtime & first services

**Goal:** a usable user-space ecosystem on top of the kernel.

- Service framework: badged endpoints, capability brokering (mint/grant/revoke).
- The WASM runtime with **capability-backed WASI** (ADR-0003).
- First real services: a content-addressed store ([Filesystem](./Filesystem.md) v0) and the
  [crypto](./Cryptography.md) keystore.
- The [SDK](https://github.com/lantern-os/lantern-sdk) v0 so a developer can build and run a confined Wasm app.

**Exit criteria:** a third-party Wasm app runs confined, reads a file *only* via a granted
capability, and cannot touch anything it wasn't granted — demonstrated adversarially.

---

## Phase 3 — Privacy, identity, networking, and AI

**Goal:** light up the differentiating capabilities.

- [Identity](./Identity.md): DIDs, the OS wallet service, verifiable credentials.
- [Networking](./Networking.md): authenticated encrypted channels; P2P discovery; a first
  anonymity-capable transport.
- [AI runtime](./AI.md): local inference, capability-scoped agents, the audit log.
- Encrypted-by-default storage tied to hardware-backed keys; provenance tracking.
- Begin **formal verification** of the IPC and capability paths.

**Exit criteria:** a user can run a capable local AI agent with a visible, revocable
capability set and a faithful audit log; a network identity can be presented without
cross-context linkage; data at rest is encrypted and provenance-tracked.

---

## Phase 4 — Hardware and assurance

**Goal:** move toward the open-hardware and high-assurance vision.

- Run on real RISC-V hardware; IOMMU-confined drivers.
- Integrate a secure enclave for key custody and attestation; measured boot end to end.
- Explore CHERI/hardware-capability backing of the software capability model.
- Extend formal verification coverage; harden against side channels where the ISA allows.

**Exit criteria:** the system runs on auditable hardware with a hardware root of trust, and
the most critical kernel paths have machine-checked proofs.

---

## Phase 5 — Ecosystem and longevity

**Goal:** make it something others build on for decades.

- A stable, versioned, RFC-governed public ABI (syscalls, WIT interfaces, capability types).
- A developer ecosystem and app/agent distribution model.
- Custom-hardware research (the [LanternOS board](./Hardware.md)) if and when justified.
- Long-term cryptographic agility, including post-quantum defaults.

**Exit criteria:** external developers ship capability-aware apps and agents; the architecture
has absorbed real-world use without violating its principles.

---

## How this roadmap changes

Phase boundaries and their exit criteria are governed by the RFC process
([GOVERNANCE](https://github.com/lantern-os/.github/blob/main/GOVERNANCE.md)). We would rather move a date than lower a bar: the entire
premise of LanternOS is that the foundation is worth getting right.
