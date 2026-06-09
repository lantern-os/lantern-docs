# Threat Model (system-wide)

This is the **system-level** threat model. Each component repository carries its own
`THREAT_MODEL.md` for threats specific to that layer; this page sets the shared frame they
all inherit. It is a living document and must be updated whenever an RFC changes a trust
boundary (the RFC template makes this mandatory).

## Method

We use a STRIDE-informed, asset-and-boundary-centric approach:

1. Enumerate **assets** worth protecting.
2. Enumerate **adversaries** and their capabilities.
3. Draw **trust boundaries** and state what is trusted on each side.
4. For each boundary, identify threats and the architectural mitigation.
5. State **non-goals** — what we explicitly do not defend against (yet).

## Assets

| Asset | Why it matters |
| --- | --- |
| User cryptographic keys | Root of identity, signing, and decryption. Compromise is catastrophic. |
| User data at rest | Documents, messages, models. Confidentiality and integrity. |
| User identity/metadata | Linkability and tracking are privacy harms even without data theft. |
| The Trusted Computing Base | Boot + kernel + minimal HAL. A flaw here defeats everything above. |
| Capabilities | Authority itself. Forgery or amplification = privilege escalation. |
| AI agent authority & audit log | An agent acting beyond its grant, or an unfaithful log, is a core failure. |
| Boot integrity / root of trust | If boot is subverted, all later guarantees are void. |

## Adversaries

| Adversary | Capabilities assumed | In scope? |
| --- | --- | --- |
| **Malicious application** | Arbitrary code in a confined component | Yes — primary |
| **Malicious / buggy AI agent** | Autonomous actions within granted capabilities, prompt-injected | Yes — primary |
| **Compromised system service** | Full control of one user-space service | Yes |
| **Network attacker** | Observe, modify, inject, block traffic; run malicious peers | Yes |
| **Surveillance / tracker** | Correlate metadata, fingerprint, link identities | Yes |
| **Local non-privileged user/process** | Another confined component on the device | Yes |
| **Physical attacker (opportunistic)** | Steals a powered-off device | Yes (encryption at rest) |
| **Physical attacker (sophisticated)** | Bus probing, cold-boot, fault injection | Partial — see non-goals |
| **Malicious supply chain** | Tampered dependency, firmware, or hardware | Partial — mitigated, not solved |
| **Nation-state with silicon implants** | Subverted hardware below our root of trust | Out of scope (Phase 0) |

## Trust boundaries

```
  ┌─ user / physical world ─────────────────────────────────────────────┐
  │  ┌─ device ──────────────────────────────────────────────────────┐  │
  │  │  ┌─ TCB ────────────────────────┐                              │  │
  │  │  │ boot (root of trust)         │   highest assurance          │  │
  │  │  │ microkernel                  │                              │  │
  │  │  │ minimal HAL                  │                              │  │
  │  │  └──────────────────────────────┘                              │  │
  │  │   ▲ capability-mediated IPC (only channel across this line)    │  │
  │  │  ┌─ confined user space ────────────────────────────────────┐  │  │
  │  │  │ drivers · fs · net · crypto svc · wasm rt · ai rt · shell │  │  │
  │  │  │ each isolated from the others; talk only via caps         │  │  │
  │  │  │  ┌─ apps & agents ───────────────────────────────────┐    │  │  │
  │  │  │  │ least privilege; hold only granted capabilities    │    │  │  │
  │  │  │  └────────────────────────────────────────────────────┘    │  │  │
  │  │  └────────────────────────────────────────────────────────────┘  │  │
  │  └──────────────────────────────────────────────────────────────────┘  │
  │   ▲ network boundary: every remote peer is untrusted                    │
  └─────────────────────────────────────────────────────────────────────────┘
```

Key invariant: **the only way to cross a boundary is to invoke a capability you hold.**
There is no ambient path.

## Threats and mitigations (selected)

| # | Threat | Architectural mitigation |
| --- | --- | --- |
| T1 | A compromised app reads all user files (ambient authority) | Object capabilities; no global FS namespace; app holds caps only to granted objects ([RFC-0003](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0003-capability-model.md)) |
| T2 | A driver bug yields full kernel compromise | Drivers run unprivileged in user space; microkernel TCB ([RFC-0002](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0002-microkernel-architecture.md)) |
| T3 | Confused deputy: a service is tricked into misusing its authority | Designation = authority; badged endpoints; no identity-based ambient rights |
| T4 | AI agent exceeds intent (incl. prompt injection) | Capability-scoped agents + mandatory audit log + user consent on grant ([AI](./AI.md)) |
| T5 | Network observer links a user across services | Per-context identities; anonymity-capable transport; metadata minimisation ([Networking](./Networking.md)) |
| T6 | Device theft exposes data | Encrypted-by-default storage tied to hardware-backed keys ([Filesystem](./Filesystem.md), [Cryptography](./Cryptography.md)) |
| T7 | Boot/firmware tampering | Measured boot, root of trust, attestation ([Hardware](./Hardware.md), `lantern-boot`) |
| T8 | Capability forgery or amplification | Kernel-enforced unforgeability; monotone attenuation; signed sealed caps |
| T9 | Telemetry/fingerprinting leaks identity | Privacy-by-default: no global IDs built; telemetry off by default and auditable |
| T10 | Supply-chain tampering of a dependency | Reproducible builds, dependency review, minimal TCB deps, provenance tracking |

## Non-goals (current)

We are honest about limits. At Phase 0 we do **not** claim to defend against:

- Sophisticated physical attacks: cold-boot RAM extraction, bus/JTAG probing, decapping,
  fault injection. (Mitigated *later* by enclaves/CHERI/encrypted memory; not solved now.)
- Malicious hardware below our root of trust (implanted silicon, subverted fab).
- Side channels (Spectre-class microarchitectural leaks) as a first-order guarantee — we
  track them and mitigate where the ISA allows, but do not claim immunity.
- Coercion of the user ("rubber-hose"); we provide deniability tools later, not magic.

These non-goals are revisited each roadmap phase and as open hardware capabilities mature.

## Open questions

- How do we bound the trust placed in user-space drivers that touch DMA-capable devices
  (IOMMU dependence)?
- What is the minimum hardware root of trust we require, and what do we do on hardware that
  lacks it?
- How do we make the AI audit log itself tamper-evident without bloating the TCB?
