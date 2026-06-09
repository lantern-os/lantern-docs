# Architecture

This page describes how the layers of LanternOS fit together. Each layer has a dedicated
wiki page and a repository; this is the connective tissue.

## Architecture-first

Before anything else, a statement of method. LanternOS follows a strict order:

```
1. Documentation   →   2. Architecture   →   3. RFCs   →   4. Prototypes   →   5. Implementation
```

We do **not** rush to code. Early kernel and capability decisions constrain everything for
years; getting them wrong is far more expensive than writing them down carefully first. The
quality of the architecture is more important than the speed of implementation. This is a
deliberate, recorded choice — see [CONTRIBUTING](https://github.com/lantern-os/.github/blob/main/CONTRIBUTING.md) and
[GOVERNANCE](https://github.com/lantern-os/.github/blob/main/GOVERNANCE.md).

## The layered model

```
   ┌──────────────────────────────────────────────────────────┐
   │  Applications & AI agents      (Wasm components, confined) │  userland
   ├──────────────────────────────────────────────────────────┤
   │  AI runtime        lantern-ai-runtime                      │  agent runtime,
   │                                                            │  model mgmt, audit
   ├──────────────────────────────────────────────────────────┤
   │  WASM runtime      lantern-runtime                         │  component model,
   │                                                            │  capability-backed WASI
   ├──────────────────────────────────────────────────────────┤
   │  Capability runtime  lantern-capabilities                  │  service/sealed caps,
   │                                                            │  brokering, attenuation
   ├──────────────────────────────────────────────────────────┤
   │  User-space system services                                │  confined,
   │   fs · net · crypto · drivers · identity                   │  mutually isolated
   │   lantern-filesystem lantern-network lantern-crypto        │
   ├══════════════════════════════════════════════════════════┤  ◀── TCB boundary
   │  Microkernel       lantern-kernel                          │  sched, mem, IPC,
   │   scheduling · memory · IPC · capability enforce · IRQ     │  caps, interrupts
   ├──────────────────────────────────────────────────────────┤
   │  HAL  lantern-hal    ·    Boot  lantern-boot               │  machine layer,
   │                                                            │  root of trust
   ├──────────────────────────────────────────────────────────┤
   │  Hardware            x86-64 (dev) → RISC-V (target)        │
   └──────────────────────────────────────────────────────────┘
```

The double line is the **TCB boundary** (see [Threat Model](./Threat-Model.md)). Everything
above it is unprivileged. `lantern-sdk` and `lantern-shell` are cross-cutting (tooling and
user surface) and sit beside this stack rather than inside one layer.

## Two cross-cutting fabrics

Two ideas run vertically through every layer:

### 1. Capabilities are the only authority

There is no ambient authority anywhere in the system. Authority is always a capability you
were explicitly handed. This is implemented at three cooperating layers
([RFC-0003](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0003-capability-model.md)):

- **Kernel capabilities** — seL4-style handles to kernel objects, checked on every syscall.
- **Service capabilities** — higher-level handles (a file, a socket) brokered by user-space
  services over kernel endpoints.
- **Sealed/cryptographic capabilities** — signed, attenuable tokens for delegation that
  must persist or cross machines (the decentralised case).

### 2. Everything is message passing

Components never share mutable state across a boundary; they exchange messages over IPC. The
kernel provides synchronous call/reply endpoints and asynchronous notifications, plus
zero-copy shared memory granted by capability for bulk transfer. This is what makes strong
isolation affordable.

## Boot-to-userland flow

1. **`lantern-boot`** establishes the root of trust: measured boot, verifies the kernel
   image, records measurements for later attestation, hands off.
2. **`lantern-kernel`** initialises, takes ownership of physical memory as *untyped* memory,
   and starts the **root task** holding the initial capabilities to all resources.
3. The **root task** retypes untyped memory and *delegates* capabilities to start the core
   user-space services (driver manager, filesystem, network, crypto, identity). It then
   gives away the broad capabilities it no longer needs — authority only ever narrows.
4. The **capability runtime** and **WASM runtime** start; the **AI runtime** starts on top.
5. The **shell** ([`lantern-shell`](https://github.com/lantern-os/lantern-shell)) presents the user surface and
   becomes the locus of consent: it is where the user grants capabilities to apps and agents.

This "narrowing waterfall" of authority from a single root task is the structural expression
of least privilege.

## Why these boundaries

| Boundary | Rationale |
| --- | --- |
| Kernel vs. everything | Minimise the TCB; a driver/service bug must not be kernel-level. |
| Service vs. service | Compromise of one service (say, the network stack) must not reach another (say, the keystore). |
| App/agent vs. system | Untrusted code (especially AI agents) runs with only granted authority. |
| Sealed caps at the network edge | Delegation across machines needs a cryptographic, not in-memory, form. |

## How the repositories compose

- **Down the stack:** `shell`/`apps` → `ai-runtime` → `runtime` → `capabilities` →
  `services` → `kernel` → `hal`/`boot` → `hardware`.
- **Sideways:** `crypto` underpins `filesystem`, `network`, and `identity`; `capabilities`
  underpins all services; `sdk` generates bindings against the runtime and service
  interfaces.
- **Process:** `rfcs` governs changes to any of the above; `docs` describes the whole.

## Open questions

- The precise IPC ABI (register layout, message structure) — gated on
  [RFC-0002](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0002-microkernel-architecture.md) follow-ups.
- Whether the root task is a single component or a small set with separated duties.
- How driver isolation interacts with DMA and the IOMMU on each target.
- The bring-up host: hosted-on-Linux vs. hypervisor vs. bare-metal-on-QEMU first.
