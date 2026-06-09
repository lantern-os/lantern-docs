# Kernel

The LanternOS kernel is a **capability-based microkernel**. Its job is to be small, fast,
and correct — small enough to eventually verify, fast enough that user-space services don't
pay an unacceptable IPC tax, and correct because everything else trusts it. Repository:
[`lantern-kernel`](https://github.com/lantern-os/lantern-kernel). Decision: [RFC-0002](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0002-microkernel-architecture.md).

## Responsibilities (the whole list)

The kernel does exactly five things:

1. **Scheduling** — manage threads and scheduling contexts; provide the mechanism, let user
   space set policy within it.
2. **Memory isolation** — address spaces and page tables; hand out physical memory as
   *untyped* memory that user space retypes into objects. The kernel does **no dynamic heap
   allocation after boot** — all kernel object memory comes from user-provided untyped
   regions, so the kernel cannot be driven out of memory by a misbehaving component.
3. **IPC** — synchronous endpoints (call/reply) and asynchronous notifications. This is the
   only cross-component communication primitive.
4. **Capability enforcement** — every object is named by a capability in a per-process
   capability space (CSpace); the kernel checks rights on every operation.
5. **Interrupt handling** — convert hardware interrupts into notifications delivered to
   user-space driver threads holding the relevant IRQ capability.

## Explicitly not in the kernel

Device drivers, filesystems, the network stack, cryptographic services, the WASM/AI
runtimes, and *all policy*. These are ordinary, unprivileged user-space components. The
kernel does not know what a file or a packet is.

## Kernel objects (initial sketch)

| Object | Purpose |
| --- | --- |
| **Untyped memory** | Raw physical memory; retyped into other objects. The source of all allocation. |
| **CNode** | A node in a CSpace; stores capabilities. |
| **TCB** (thread control block) | A schedulable thread of execution. |
| **Endpoint** | Synchronous IPC rendezvous; may be *badged*. |
| **Notification** | Asynchronous signalling; bound to IRQs for drivers. |
| **VSpace / page tables** | Address-space mappings. |
| **Frame** | A page of physical memory mappable into a VSpace. |
| **IRQ handler** | Capability to receive a specific interrupt. |
| **Scheduling context** | Budget/period for time-based isolation (model TBD). |

This list is deliberately close to seL4's; we start from a design that has been *proven*
implementable and verifiable, then diverge only with justification.

## IPC: the hot path

A microkernel lives or dies by IPC performance (the central lesson of the L4 lineage). The
kernel commits to:

- **Register fast-path** for small synchronous messages (no memory traffic).
- **Zero-copy bulk transfer** via shared frames granted by capability.
- **Core-local scheduling and data structures** to avoid cross-core lock contention.
- Benchmarks gate IPC changes; a regression in IPC latency blocks merge.

## Memory model

- Physical memory becomes **untyped** at boot, owned initially by the root task.
- User space **retypes** untyped into typed kernel objects, accounting for the memory
  explicitly. There is no hidden kernel allocator.
- This makes memory **a capability** like everything else, gives spatial isolation by
  construction, and prevents kernel memory exhaustion attacks.

## Concurrency and assurance

- The concurrency model (single big kernel lock vs. fine-grained vs. event-based, à la
  seL4) is an **open question** tracked under RFC-0002 follow-ups; it has major implications
  for both performance and verifiability.
- Long-term we treat the IPC and capability paths as **formal verification targets**
  (see [Roadmap](./Roadmap.md) Phase 3+). Writing the kernel in Rust (ADR-0001) and keeping
  it tiny is what makes that ambition realistic.

## Portability

All ISA- and platform-specific code lives behind the HAL ([`lantern-hal`](https://github.com/lantern-os/lantern-hal)):
context switch, page-table format, trap entry, timer, and interrupt controller. The portable
kernel core should not contain `#[cfg(target_arch)]` beyond the HAL seam. This is what keeps
the x86-64 (development) → RISC-V (target) path honest ([ADR-0002](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0002-riscv-target-isa.md)).

## Threat model summary

The kernel *is* the TCB's core. A flaw here is maximal severity. Mitigations: minimal size,
safe Rust with audited `unsafe`, no post-boot allocation, capability checks on every path,
and eventual verification. Full detail in [`lantern-kernel/THREAT_MODEL.md`](https://github.com/lantern-os/lantern-kernel/blob/main/THREAT_MODEL.md)
and the [system threat model](./Threat-Model.md).

## Open questions

- Scheduling-context model: full MCS or a simpler initial scheme?
- Kernel concurrency model and its verification cost.
- Exact syscall/IPC ABI and message register layout.
- How much of interrupt handling and the IOMMU sits at the HAL seam vs. the portable core.
