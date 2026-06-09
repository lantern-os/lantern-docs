# Principles

These six principles are the constitution of LanternOS. Every RFC, every line of code, and
every governance decision is measured against them. When principles conflict in a specific
decision (they sometimes will), the conflict must be made explicit in the relevant RFC and
resolved on the record — never silently.

The ordering below is also, roughly, the priority order when principles collide.

---

## 1. Security by architecture

> Security should be a property of the structure, not a feature bolted on.

We do not rely on a program "behaving well" or on a scanner catching bad behaviour. We make
whole classes of attack **unrepresentable**.

**Avoid:** large trusted computing bases, privileged monolithic services, ambient authority,
global namespaces, implicit access.

**Prefer:** isolation between components, object capabilities, message passing, explicit
permissions, least privilege.

*Test:* "If this component is fully compromised, what is the blast radius?" The answer must
be "exactly the capabilities it held."

---

## 2. Privacy by default

> The user owns their identity, data, models, and keys. The system minimises what it knows.

Privacy is not a settings page; it is the default state. We minimise telemetry, metadata
leakage, tracking, and fingerprinting at the architectural level — for example, by
partitionable identities and by *not building* the global identifiers that enable linkage.

*Test:* "What does this feature let someone learn about the user?" If the answer is
"something they did not consent to", it is a bug.

---

## 3. Local-first ownership

> The cloud is optional. The device is sovereign.

The system is fully functional offline. Storage, AI, communication, and identity live on the
device by default. Cloud and network peers are *optional collaborators*, never required
custodians. Synchronisation is something the user opts into and controls.

*Test:* "Does this still work with the network unplugged?" For core functionality the answer
must be yes.

---

## 4. Memory safety

> The TCB must not fail to memory-corruption bugs.

Rust is the primary language (see [ADR-0001](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0001-rust-as-primary-language.md)).
`unsafe` is isolated behind safe abstractions, justified in-comment, minimised, and reviewed.
We treat `unsafe` density in the TCB as a tracked quality metric and pursue formal
verification for the most critical paths over time.

*Test:* "Can this introduce a use-after-free, overflow, or data race?" If yes, it does not
ship without isolation and review.

---

## 5. Open hardware

> Long-term, the whole stack should be auditable down to the silicon.

The long-term target is RISC-V (see [ADR-0002](https://github.com/lantern-os/lantern-rfcs/blob/main/adr/0002-riscv-target-isa.md)),
with auditable firmware and open toolchains. We avoid unnecessary dependence on proprietary
platforms and confine all ISA- and board-specific assumptions behind the HAL so the system
stays portable.

*Test:* "Does this bake in a dependence on a closed platform we could otherwise avoid?"

---

## 6. AI-native

> AI is an operating-system capability, not an application.

The OS itself provides the agent runtime, model management, local inference, and — crucially
— **capability-scoped, auditable agency**. An agent never receives unrestricted access; it
receives explicit capabilities, and everything it does is logged and attributable.

*Test:* "Can this agent do something the user did not grant it the capability to do?" The
answer must be no.

---

## How principles are applied

- Every RFC's **Motivation** must name the principle it serves and any principle it strains.
- The mandatory **Threat model impact** and **TCB impact** sections operationalise
  principles 1 and 4.
- The **Privacy impact** section operationalises principle 2.
- Reviewers may block a change purely on principle grounds; that block is recorded.

## When principles conflict

Real examples we expect:

- **Privacy vs. AI capability:** a more capable local agent may want broad data access.
  Resolution: scope it with capabilities and an audit log; never grant ambient access.
- **Open hardware vs. practicality:** the best NPU today may be proprietary. Resolution:
  isolate it behind the HAL, document the dependency in an ADR, prefer open paths over time.
- **Local-first vs. decentralised collaboration:** sync needs the network. Resolution: make
  it opt-in, end-to-end encrypted, and functional-when-offline.

Conflicts are not failures of the principles; they are the points where design judgement is
required, and they must be resolved in writing.
