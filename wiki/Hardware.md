# Hardware

LanternOS's long-term ambition is a stack that is **auditable down to the silicon**. This
page records the hardware vision and the research directions. It is explicitly
*research-and-direction*: nothing here is implemented, and per the architecture-first method
we document the possibilities before committing. Repositories:
[`lantern-hal`](https://github.com/lantern-os/lantern-hal) (abstraction) and [`lantern-boot`](https://github.com/lantern-os/lantern-boot)
(root of trust).

## ISA strategy

- **Long-term target: RISC-V** ([ADR-0002](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0002-riscv-target-isa.md)) —
  open, extensible, sovereign, and a platform for security innovation at the ISA level.
- **Bring-up: x86-64 and `riscv64` under QEMU** for practicality, with all ISA-specific code
  behind the HAL so x86-64 never becomes a strategic dependency.

## Why open hardware

Every layer above the hardware can be perfectly designed and still be undermined by an
opaque, untrustworthy substrate (closed boot ROMs, hidden management engines, unauditable
microcode). [User sovereignty](./Principles.md) is incomplete if the bottom of the stack is a
black box. RISC-V plus open firmware and open toolchains is the only path to a fully
auditable system — even if we can't reach the very bottom soon.

## A possible LanternOS board (research)

A long-horizon sketch of dedicated hardware, to be researched, not built yet:

| Block | Role | LanternOS tie-in |
| --- | --- | --- |
| **RISC-V CPU** | General compute, open ISA | The base target; capability-friendly extensions (CHERI). |
| **Secure enclave** | Key custody, attestation, measured-boot anchor | Backs [crypto](./Cryptography.md) and [identity](./Identity.md). |
| **AI accelerator (NPU)** | Local inference | Powers the [AI runtime](./AI.md); capability-mediated. |
| **Cryptographic accelerator** | Fast AEAD/sign/hash, RNG | Speeds the [crypto service](./Cryptography.md). |
| **FPGA region** | Reconfigurable logic | Experimentation: custom capability/crypto/ISA features. |
| **Networking processor** | Offload, isolation of the net stack | Confines [networking](./Networking.md) further. |
| **Hardware-backed identity** | Per-device root key, true RNG | Anchors keys; mediated to avoid becoming a tracker. |
| **IOMMU** | Confine DMA-capable devices | Makes user-space drivers safe(r). |

## Hardware-assisted security directions

- **CHERI / hardware capabilities** — enforce capability integrity and spatial memory safety
  in hardware, directly backing the software [capability model](./Security.md). The most
  exciting long-term alignment for LanternOS.
- **Memory tagging / pointer masking** — cheap spatial/temporal safety where available.
- **Measured boot & attestation** — `lantern-boot` measures each stage into the enclave so
  the system can *prove* what it is running; balanced against the privacy risk of attestation
  as a fingerprint ([Identity](./Identity.md)).
- **Encrypted memory** — mitigate cold-boot/physical RAM attacks (a current
  [non-goal](./Threat-Model.md) we want to close).

## The HAL seam

[`lantern-hal`](https://github.com/lantern-os/lantern-hal) is the contract that keeps all of this portable. It
abstracts: CPU context switch, page-table format, trap/interrupt entry, timers, the interrupt
controller, the IOMMU, and platform discovery (device tree / ACPI). The portable kernel core
contains no per-ISA logic beyond this seam — that discipline is what makes the
x86-64 → RISC-V journey credible and what keeps the door open to custom hardware.

## Firmware and supply chain

- Prefer **open firmware** (e.g. OpenSBI-class supervisor firmware on RISC-V) and reproducible
  builds so the boot path is auditable.
- Treat the **supply chain** as part of the threat model ([Threat Model](./Threat-Model.md)):
  reproducible builds, provenance, and minimal trusted firmware reduce (not eliminate) the
  risk of tampered hardware/firmware.

## Open questions

- Minimum viable hardware to demonstrate the thesis — repurposed existing RISC-V boards
  first, custom silicon much later (if ever).
- How much hardware-backed security to *require* vs. treat as optional acceleration, given
  bring-up hardware won't have it.
- Reconciling attestation (proving boot state) with anti-fingerprinting.
- Which RISC-V extensions (H, V, crypto, CHERI/pointer-masking) to track and target, and when.
