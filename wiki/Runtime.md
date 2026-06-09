# Runtime

The runtime layer is where **applications and agents actually execute**. It sits above the
system services and below the AI runtime. It has two parts: the **service framework** (how
user-space system services are built and composed) and the **WASM runtime** (how portable,
sandboxed applications run). Repositories: [`lantern-runtime`](https://github.com/lantern-os/lantern-runtime) and
[`lantern-capabilities`](https://github.com/lantern-os/lantern-capabilities).

## Why WebAssembly

LanternOS uses **WebAssembly with the Component Model and WASI Preview 2** as its portable
application ABI ([ADR-0003](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0003-wasm-as-portable-app-abi.md)). The
fit is unusually good:

- **Portable** across x86-64 (dev) and RISC-V (target) — write once, run on any LanternOS ISA.
- **Sandboxed by default** — Wasm has *no* ambient capabilities; a module can only call host
  functions it was explicitly given. This is the *same* idea as our capability model, seen
  from the application side.
- **Language-agnostic** — the ecosystem isn't limited to Rust; anything that compiles to Wasm
  can be a LanternOS app.
- **Typed interfaces (WIT)** — the Component Model gives strongly-typed, composable interface
  contracts that the [SDK](https://github.com/lantern-os/lantern-sdk) generates bindings from.

## Capability-backed WASI (the key twist)

Standard WASI exposes ambient-ish capabilities (a preopened directory, a clock, sockets).
LanternOS does **not** do this. Instead, every WASI/host interface is **backed by LanternOS
object capabilities**:

- A component receives a file handle only if the user/granting component handed it a file
  capability. No preopened "current directory" with broad access.
- Network access is a socket capability to a specific destination/identity, not an open door.
- Clocks, randomness, and environment are mediated and minimised (a fingerprinting concern).

So "Wasm deny-by-default sandbox" and "LanternOS no-ambient-authority" are literally the same
mechanism. An app does exactly what its capabilities permit.

## Execution model

- Components are **AOT-compiled** (ahead-of-time) and validated before execution for
  performance and to avoid JIT in sensitive contexts.
- Each component runs in a confined host process with its own capability set; components
  compose by passing typed interfaces and capabilities, never shared mutable memory.
- The host engine itself is a confined user-space service — a bug in the Wasm engine is
  bounded by *its* capabilities, not the kernel's.

## The service framework

System services (filesystem, network, crypto, drivers, identity) are built on a common
framework that provides:

- **Endpoint plumbing** over kernel IPC, with **badged** endpoints so a service can safely
  multiplex mutually distrusting clients ([Security](./Security.md)).
- **Capability brokering**: minting attenuated capabilities, granting them over IPC, and
  revoking them — the practical surface of [RFC-0003](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0003-capability-model.md).
- **The narrowing-waterfall startup** ([Architecture](./Architecture.md)): services receive
  exactly the capabilities they need from the root task and no more.

## Native vs. Wasm

| Tier | Mechanism | Examples |
| --- | --- | --- |
| TCB | Native Rust, in-kernel | scheduler, IPC, capability check |
| System services | Native Rust, confined user space | fs, net, crypto, drivers |
| Apps & many agents | Wasm components, confined | user applications, AI agents |

Performance-critical services stay native; the portable, untrusted, sandboxed tier is Wasm.

## Open questions

- Which Wasm engine to adopt/build on, and the AOT pipeline shape.
- Pinning and versioning the WASI/Component-Model surface (it is still maturing) as a
  governed public ABI.
- The exact mapping from WIT-typed handles to LanternOS service/sealed capabilities.
- Resource accounting (CPU/memory budgets) for components, tied to scheduling contexts.
