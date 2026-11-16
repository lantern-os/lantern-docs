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

## Phase 1 — Microkernel prototype  *(complete — closed by [RFC-0009](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0009-phase-1-to-phase-2-transition.md)/[ADR-0014](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0014-phase-1-complete-phase-2-opened.md))*

**Goal:** prove the core mechanisms in throwaway/experimental code.

- [x] Boot to a minimal kernel on `riscv64` under QEMU (`lantern-boot`, `lantern-hal`;
      `x86-64` boot deferred, not blocking).
- [x] Address spaces, threads, and a scheduler.
- [x] IPC fast-path (endpoints + notifications) with benchmarks (see
      [ADR-0013](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0013-ipc-latency-benchmark.md)
      — a real, reproducible, unresolved IPC round-trip-loss bug was found and carried
      forward as a known risk, not blocking this exit).
- [x] Kernel capability mechanism (CSpace, untyped retyping) — the RFC-0003 kernel layer.
- [x] The "narrowing-waterfall" root task starting one confined user-space service (see
      [ADR-0012](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0012-vspace-frame-capabilities-and-elf-loader.md)).

**Exit criteria:** a confined user-space "hello service" reachable only via a granted
capability, with IPC latency benchmarked and within target. Prototype code is explicitly
allowed to be thrown away.

---

## Phase 2 — Capability runtime & first services  *(complete — closed by [RFC-0017](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0017-phase-2-to-phase-3-transition.md)/[ADR-0021](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0021-phase-2-complete-phase-3-opened.md))*

**Goal:** a usable user-space ecosystem on top of the kernel.

- [x] Service framework: badged endpoints, capability brokering (mint/grant/revoke) —
      [RFC-0010](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0010-cross-process-capability-transfer-and-brokering.md);
      `lantern-capabilities`' `Broker`, proven against a real `KernelState` and under real
      confined U-mode `ecall`s.
- [x] The WASM runtime with **capability-backed WASI** (ADR-0003) —
      [RFC-0013](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0013-wasm-engine-selection-and-aot-strategy.md)/[RFC-0014](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0014-wit-handle-capability-mapping.md)/[RFC-0016](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0016-filesystem-wit-interface.md):
      the WIT-handle ⇄ capability mapping, deny-by-default, no ambient `wasmtime-wasi`.
- [x] First real services: a content-addressed store ([Filesystem](./Filesystem.md) v0) and
      the [crypto](./Cryptography.md) keystore — both exercised with real `Broker`-minted badges.
- [x] The [SDK](https://github.com/lantern-os/lantern-sdk) v0 so a developer can build and
      run a confined Wasm app —
      [RFC-0015](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0015-capability-manifest-format.md):
      the `lantern.capabilities.toml` parser/validator, interface registry, `GrantPlan`,
      package signing, and the `lantern-sdk build` CLI.
- **Carried forward** ([ADR-0021](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0021-phase-2-complete-phase-3-opened.md)):
      the services and the runtime run as in-process stand-ins on a host target, not yet as
      confined processes on the kernel. Proven as *mechanisms*; a real IPC transport under
      them and the Wasmtime `riscv64` custom-platform port are **Phase 3's foundational
      prerequisite**, not a Phase 2 gap. The Phase 1 IPC round-trip-loss bug is also carried
      forward, still.

**Exit criteria:** a third-party Wasm app runs confined, reads a file *only* via a granted
capability, and cannot touch anything it wasn't granted — demonstrated adversarially.
**Met** — `lantern-example-signer`: packaged by `lantern-sdk build`, run by a host that
verifies the `.lpkg` and traps everything ungranted; its `probe` export shows ungranted
capability slots return `none` and a write through a read-only handle is refused.

---

## Phase 3 — Privacy, identity, networking, and AI  *(complete — closed by [RFC-0036](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0036-phase-3-to-phase-4-transition.md)/[ADR-0041](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0041-phase-3-complete-phase-4-opened.md))*

**Goal:** light up the differentiating capabilities.

- **First:** the confined-execution port carried forward from Phase 2 — `Broker`/`Keystore`/`Store`
  as IPC-reachable confined services, and `lantern-runtime` hosted on `riscv64` via
  Wasmtime's custom-platform hooks against `lantern-hal`/VSpace-Frame capabilities. Designed
  by [RFC-0018](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0018-confined-execution-port.md)
  (Accepted), fixed by
  [ADR-0022](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0022-confined-service-model-and-call-transport.md)
  (confined-service model + badged-endpoint/shared-`Frame` transport) and
  [ADR-0023](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0023-wasmtime-no-std-pulley-hosting.md)
  (Wasmtime `no_std` + Pulley) and
  [ADR-0024](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0024-confined-service-call-protocol.md)
  (the `keystore`/`store` shared-`Frame` wire protocol) — none add anything to the TCB.

- [Identity](./Identity.md): DIDs, the OS wallet service, verifiable credentials.
- [Networking](./Networking.md): authenticated encrypted channels; P2P discovery; a first
  anonymity-capable transport.
- [AI runtime](./AI.md): local inference, capability-scoped agents, the audit log.
- Encrypted-by-default storage; provenance tracking. In this phase the keys are software-held,
  which protects data from other confined programs but **not from theft of the device**.
  Hardware-backed custody is Phase 4 ([RFC-0023](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0023-hardware-key-custody-in-phase-4.md)).
- Begin **formal verification** of the IPC and capability paths.

**Exit criteria:** a user can run a capable local AI agent with a visible, revocable
capability set and a faithful audit log; a network identity can be presented without
cross-context linkage; data at rest is encrypted (under software-held keys, until Phase 4) and
provenance-tracked.

Progress on them:

- [x] **Data at rest encrypted and provenance-tracked:** the store's files are sealed on the disk
  ([RFC-0031](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0031-storage-a-block-driver-and-the-store-on-disk.md)), and each keeps which apps
  wrote it ([RFC-0033](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0033-provenance-for-data-at-rest.md)).
- [x] **A local AI agent:** a confined inference service runs a language model behind a capability, and the file
  assistant, an ordinary app, continues a person's text with it: each of its three capabilities asked for, listed,
  revocable and audited, and its output recorded as its own
  ([RFC-0034](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0034-local-inference-and-a-file-assistant.md)). The model is small (15M
  parameters): the interface is what a larger one, or an accelerator, will sit behind.
- [x] **A network identity without cross-context linkage:** the wallet presents each app a different identity per peer,
  derived from a root only it holds, over a Noise session through a confined UDP stack; the peers, checking with an
  independent Noise implementation, can't link them ([RFC-0035](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0035-pairwise-network-identities.md)).
  What links them today is the network itself: one address, and timing. An anonymity transport is future work.

Carried forward by RFC-0036: **formal verification**, never started here, is Phase 4's first work. DIDs and credentials,
peer discovery and an anonymity transport continue alongside Phase 4.

---

## Phase 4 — Hardware and assurance  *(current)*

**Goal:** move toward the open-hardware and high-assurance vision.

- Run on real RISC-V hardware; IOMMU-confined drivers.
- **Hardware-backed key custody**, moved here from Phase 3 and not to be dropped: integrate a
  secure enclave for key custody and attestation, with a per-device root key and hardware-seeded
  randomness; measured boot end to end. The RFC opening this phase must account for every item in
  [RFC-0023's checklist](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0023-hardware-key-custody-in-phase-4.md#the-phase-4-checklist).
- Explore CHERI/hardware-capability backing of the software capability model.
- Extend formal verification coverage; harden against side channels where the ISA allows.

**Opened by RFC-0036**, which accounts for every item in RFC-0023's checklist, each with a work stream. In order:

1. [ ] **Formal verification** of the capability and IPC paths: an RFC choosing the tool and the first properties. Needs
   no hardware.
2. [ ] The hardware-independent parts of rollback resistance (keyed policy and decision logs, signed audit headers and
   retirements) and of signed models.
3. [ ] **Board bring-up** (the installable track's step 6), then the root of trust, custody, measured and secure boot,
   and IOMMU-confined drivers, as far as the board allows.

**Minimum hardware** (RFC-0036), in two tiers:

- **To run:** RV64GC with Sv39, a PLIC or AIA, an SBI timer, a device tree, and storage the system can drive.
- **To exit this phase:** additionally an IOMMU over every DMA-capable device, an entropy source (`Zkr` or a hardware
  generator), and a root of trust that holds keys and a monotonic counter.

**Exit criteria:** the system runs on auditable hardware with a hardware root of trust holding
every key; device theft yields only ciphertext; the revocation and audit logs resist tampering and
rollback; and the most critical kernel paths have machine-checked proofs.

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

## Track: toward an installable system

**Goal:** an OS a person can install and run, in a VM or on hardware. This track runs alongside the phases and changes no
phase's exit criteria. Added by [RFC-0026](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0026-trusted-path-and-consent.md)/[ADR-0032](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0032-trusted-path-and-consent.md).
Each step needs its own RFC.

1. [x] **An interactive system image**: `lantern-boot-system`. A person decides each grant at a console only the shell can
   reach ([RFC-0026](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0026-trusted-path-and-consent.md)).
2. [x] **Interrupts, a preemption timer and an idle loop**
   ([RFC-0027](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0027-interrupt-delivery-timer-and-idle.md)), and **user-space deadlines with a
   capability-gated clock** ([RFC-0028](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0028-deadlines-and-a-capability-clock.md)). A consent request
   nobody answers is declined after 60 seconds.
3. [x] **Launching from user space**: `apps` and `run NAME` at the console start an app from a signed package after boot,
   up to 4 at once, each reclaimed when it ends ([RFC-0029](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0029-a-confined-launcher.md), built on
   processes made, contained and reclaimed from user space,
   [RFC-0030](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0030-process-lifecycle-faults-and-reclamation.md)).
4. [x] **Storage**: a confined virtio-blk driver and a volume service; the packages, and the sealed store, on a GPT disk
   image instead of inside the boot image, so files outlive a reboot
   ([RFC-0031](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0031-storage-a-block-driver-and-the-store-on-disk.md)).
5. [x] **Installing**: a bootable disk image a VM boots directly, through UEFI and a Lantern EFI loader, with the policy
   store, packages and files on it; installing is writing that image
   ([RFC-0032](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0032-installing-a-bootable-disk.md)). An installer inside LanternOS is future work.
6. [ ] **Real hardware**: Phase 4's RISC-V board work, with [RFC-0023](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0023-hardware-key-custody-in-phase-4.md)'s checklist.

`x86-64` boot stays deferred; QEMU's `riscv64 virt` is the VM target until a decision says otherwise.

---

## How this roadmap changes

Phase boundaries and their exit criteria are governed by the RFC process
([GOVERNANCE](https://github.com/lantern-os/.github/blob/main/GOVERNANCE.md)). We would rather move a date than lower a bar: the entire
premise of LanternOS is that the foundation is worth getting right.
