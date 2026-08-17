# Vision

## The thesis

The dominant operating systems were architected for a world that no longer exists. Unix
(1969) and its descendants assumed a trusted multi-user timesharing machine, benign
software, slow and trusted local networks, no cryptographic identity, and no notion of
autonomous software acting on a user's behalf. Windows and the Unix family carry those
assumptions forward through decades of compatibility.

Computing today is the opposite of those assumptions:

- Software is **untrusted by default** — we run code from thousands of strangers daily.
- The network is **hostile and global** — every device is exposed to the entire internet.
- **Identity is cryptographic** — keys, not passwords, are the real credential.
- **AI agents act autonomously** — software now takes actions, not just instructions.
- **Surveillance is the default business model** — telemetry and tracking are pervasive.
- **Open hardware is viable** — RISC-V makes a sovereign stack possible.

LanternOS is a bet that an operating system designed *for these realities from the first
line* can be dramatically more secure, more private, and more capable than one that retrofits
them. The name is the metaphor: a lantern is something you own, that you carry, that lights
only what you point it at — local, personal, and under your control.

## What we are building

An operating system where:

- **Authority is explicit.** Every program runs with exactly the capabilities it was given
  and nothing more. There is no "root", no ambient filesystem, no global network access.
- **Privacy is the default, not a setting.** Data is encrypted at rest, identities are
  partitionable, and the system minimises the metadata it produces about you.
- **The device is sovereign.** It works fully offline. The cloud is an optional peer, never
  a dependency. You own your keys, your data, your models.
- **AI is a first-class, governed capability.** Agents are sandboxed, capability-scoped, and
  auditable. An agent can do powerful things *only* when you grant it the authority to.
- **The whole stack is auditable.** Memory-safe code, a tiny kernel, open hardware, and a
  written decision trail.

## What success looks like

- A microkernel small enough to reason about formally.
- A capability model that makes "this app can read all my files" structurally impossible to
  express by accident.
- A user who can run a capable AI assistant locally, knowing exactly what it can and cannot
  touch, with a log of everything it did.
- A device that does not phone home, cannot be silently fingerprinted, and whose firmware
  and ISA are open to inspection.
- A system whose architecture is still defensible in twenty years.

## What we are explicitly not doing

- Not chasing binary or source compatibility with Linux, Windows, or macOS. Compatibility
  is a value, but never at the cost of the [principles](./Principles.md).
- Not shipping a kernel first. We are [architecture-first](./Architecture.md#architecture-first):
  documentation, threat models, and RFCs precede code.
- Not pretending this is quick. This is a multi-year, possibly multi-decade, project. The
  quality of the foundation matters more than the speed of the first demo.

## The bet, stated plainly

> If memory-safe languages, capability security, cryptographic identity, decentralised
> networking, local AI, and open hardware are all available *today*, then the right move is
> not to bolt them onto a 1970s kernel — it is to design the operating system that assumes
> them. LanternOS is that design.

## Where this leads

The architecture above is the foundation, not the product, and nothing about the long-term
product direction changes it — the security and privacy bet stands as written. What it enables
is described separately in [Intent Model](./Intent-Model.md): a user-facing experience that
*replaces* the desktop-and-applications metaphor with intent, a Personal Agent, and Contexts,
rather than adding an assistant on top of a conventional desktop. Keeping the two documents
separate stops "why a new kernel" from getting conflated with "what it's eventually for," but
both hold at once, and the current phase remains the kernel; see [Roadmap](./Roadmap.md).

## Open questions

- Where exactly is the line between "principled clean slate" and "unusable because nothing
  is compatible"? How much of a compatibility bridge (e.g. a Linux ABI shim in a confined
  component) is acceptable without eroding the model?
- What is the minimum viable hardware that proves the thesis?
