# AI

> AI is not an application. AI is a first-class operating-system capability.

This is one of the defining bets of LanternOS. Conventional systems treat AI as an app or a
cloud API bolted on top. LanternOS treats **local inference, model management, and governed
agency** as OS services — so that AI can be powerful *and* private *and* under the user's
explicit control. Repository: [`lantern-ai-runtime`](https://github.com/lantern-os/lantern-ai-runtime).

## The three jobs of the AI runtime

1. **Local inference.** Run models on-device using the CPU and available accelerators
   (GPU/NPU). Local-first means your prompts, documents, and model outputs don't leave the
   device unless you choose ([Principles](./Principles.md)). Cloud models are an *optional
   peer*, accessed under explicit, audited capabilities.
2. **Model management.** Models are content-addressed artefacts ([Filesystem](./Filesystem.md))
   with provenance and integrity: you know exactly which model/weights ran. Scheduling shares
   scarce accelerator resources fairly across agents.
3. **Capability-scoped, auditable agency.** This is the heart of it (below).

## Agents are capability-scoped — always

The non-negotiable rule:

> **An AI agent never receives unrestricted access. It receives explicit capabilities, and
> everything it does is logged and attributable.**

Concretely:

- An agent is just a confined component ([Runtime](./Runtime.md)) holding a *capability set*.
  It can read this document and call that tool **only because** it was granted those
  capabilities — never ambiently.
- Capabilities are **attenuable and revocable**: you can grant "read this one folder" or
  "send email to this one address for the next hour," and revoke it instantly
  ([RFC-0003](https://github.com/lantern-os/lantern-rfcs/blob/main/rfcs/0003-capability-model.md)).
- The **shell** ([`lantern-shell`](https://github.com/lantern-os/lantern-shell)) is the locus of consent: granting an
  agent a new capability is a visible, user-mediated act, not a silent config change.

## Auditability

Every consequential action an agent takes is recorded in a **tamper-evident audit log**:
which agent, under which capability, did what, to which object, when. This serves several
goals at once:

- **Accountability** — you can answer "what did my assistant actually do yesterday?"
- **Provenance** — outputs written to storage carry which agent/model produced them
  ([Filesystem](./Filesystem.md)).
- **Containment review** — anomalous behaviour is visible and attributable.

Making the log itself trustworthy without bloating the TCB is an [open question](#open-questions).

## Why this matters: the prompt-injection / autonomy problem

Autonomous agents introduce a genuinely new adversary: an agent that is *induced* (e.g. via
prompt injection from data it reads) to misuse its authority. Capability scoping is the
structural defence — **the agent simply cannot act beyond its granted capabilities, no matter
what it is tricked into "wanting."** If an agent has no network capability, no amount of
prompt injection makes it exfiltrate data. This is why we put AI on top of the capability
model rather than beside it. See [Threat Model](./Threat-Model.md) T4.

## Hardware acceleration

- Use GPU/NPU where present, mediated by confined drivers behind the [HAL](./Hardware.md)
  and an IOMMU so an accelerator/driver can't read arbitrary memory.
- The long-term [open hardware](./Hardware.md) vision includes an on-board AI accelerator,
  designed so that the accelerator path is also capability-mediated and auditable.

## What the AI runtime is *not*

- It is **not** a telemetry pipeline. The OS does not ship your interactions anywhere by
  default; that would violate [privacy by default](./Principles.md).
- It is **not** a privileged super-assistant with keys to everything. There is no "the AI can
  do anything" mode; there is only "the AI can do what you granted."

## Open questions

- Model/agent scheduling across CPU/GPU/NPU under fairness and energy constraints.
- The audit log's tamper-evidence and storage cost; how much (if any) lives in the TCB.
- Standard capability *shapes* for common agent powers (read-files, send-message, run-tool)
  so consent is comprehensible to users.
- Confining models that are themselves large and may embed behaviours; sandboxing inference.
- The right consent UX so granting capabilities to agents is meaningful, not click-through.
