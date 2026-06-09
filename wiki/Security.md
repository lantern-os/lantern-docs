# Security

Security in LanternOS is the result of **architecture**, not vigilance. This page describes
the security model that the whole system inherits. It builds on the
[Threat Model](./Threat-Model.md) and the capability decision
([RFC-0003](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0003-capability-model.md)).

## The one big idea: object capabilities

Authority is a thing you *hold*, not a thing you *are*. A capability is an unforgeable token
that both **designates** an object and **carries the rights** to act on it. Three properties
fall out of this:

- **No ambient authority.** A process with no capabilities can compute and nothing else. It
  cannot "open a file" because there is no global filesystem to open from — only objects it
  holds caps to.
- **No confused deputy.** Because designation and authority are the same token, a service
  cannot be tricked into using *its* authority on an attacker's behalf — the caller must
  supply the capability, which it could only have if it legitimately held it.
- **Least privilege is the default.** Delegation is explicit and **attenuable**: you can only
  ever hand someone an equal or weaker capability, never a stronger one.

See [RFC-0003](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0003-capability-model.md) for the full model
(kernel / service / sealed layers, operations, badging).

## Defence in depth around the core idea

| Mechanism | What it buys |
| --- | --- |
| **Microkernel / tiny TCB** | A driver or service bug is *not* a kernel compromise ([Kernel](./Kernel.md)). |
| **Component isolation** | Each service runs in its own address space; compromise is contained. |
| **Message passing only** | No shared mutable state across boundaries → fewer corruption and TOCTOU bugs. |
| **Memory safety (Rust)** | Removes the dominant class of TCB vulnerabilities ([ADR-0001](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0001-rust-as-primary-language.md)). |
| **Encrypted-by-default storage** | Data at rest is protected on device loss ([Filesystem](./Filesystem.md)). |
| **Hardware-backed trust** | Keys and measurements anchored in hardware ([Hardware](./Hardware.md)). |
| **Measured boot** | The system can prove what software it is running ([`lantern-boot`](https://github.com/lantern-os/lantern-boot)). |
| **Capability-scoped AI** | Autonomous agents are confined and audited ([AI](./AI.md)). |

## Zero trust, applied internally

"Zero trust" is usually a network slogan; in LanternOS it is an *internal* design rule.
Every component treats every other component — and every input — as untrusted until a
capability says otherwise. There is no privileged "inside the machine" the way Unix trusts
root. The network boundary is just one of many; the same scepticism applies between two
local services.

## Hardware-assisted security (direction)

As we move toward RISC-V we intend to *back software guarantees with hardware*:

- **CHERI-style hardware capabilities** to enforce capability integrity and spatial memory
  safety at the pointer level — a natural fit for our model.
- **Secure enclave** for key custody and attestation.
- **IOMMU** to confine DMA-capable drivers so a malicious device/driver cannot read
  arbitrary memory.
- **Pointer masking / memory tagging** where available.

These are research/direction items (see [Hardware](./Hardware.md)), confined behind the HAL,
and never assumed to exist on bring-up hardware.

## What we do *not* rely on

- We do not rely on antivirus-style scanning or behavioural detection as a primary control.
- We do not rely on users making correct security decisions for everything — capabilities
  make the safe path the default and surface consent only where it is meaningful.
- We do not assume the network, peers, or the cloud are honest. Ever.

## Security process

- TCB changes require steward review and a threat-model update (see [GOVERNANCE](https://github.com/lantern-os/.github/blob/main/GOVERNANCE.md)).
- Cryptographic changes require an RFC ([Cryptography](./Cryptography.md)).
- Vulnerability handling follows [SECURITY.md](https://github.com/lantern-os/.github/blob/main/SECURITY.md).

## Open questions

- The exact rights lattice per object type and how it is expressed in the SDK.
- Revocation semantics and cost bounds for deeply delegated capabilities.
- How much CHERI/hardware backing to *require* vs. treat as optional acceleration.
