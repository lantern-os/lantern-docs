# Intent Model

> Human → Intent → Personal Agent → Capabilities → Operating System

This page extends [Vision](./Vision.md), it does not compete with it. Every security and
privacy commitment there — capability security, local-first ownership, memory safety,
auditable AI — carries over unchanged; nothing here weakens it. What this page adds is the
**user-facing** side: it replaces the desktop-and-applications interaction model as LanternOS's
*primary* experience, rather than bolting an assistant onto a conventional desktop as an
extra mode. It describes what LanternOS is *for*, once the kernel and capability runtime
described elsewhere in this wiki exist, and is a target to keep the architecture from
accidentally foreclosing that direction — not a near-term spec. See [Roadmap](./Roadmap.md)
for what is actually being built now. Full source: `Lantern OS — Vision and Architectural
Context.md` at the repository root.

## The central idea

Traditional desktop computing: **User → Applications → Files → Operating System**.
LanternOS, long-term: **Human → Intent → Personal Agent → Capabilities → Operating System**.

The user expresses what they want ("show me what happens to our budget if income falls by
20%") rather than choosing an application, finding a file, and operating a workflow manually.
A spreadsheet may still exist underneath and remain inspectable — the point is that operating
spreadsheet software stops being the primary interaction.

This is a spiritual return to the early home-computer `READY.` prompt — a machine the user
instructs and creates with directly — built on natural language and local AI instead of BASIC,
and mediated by a Personal Agent rather than a bare shell.

## Intent, not applications; Contexts, not files

Capabilities like text editing, calculation, structured data, communication, and search remain
useful *implementation* mechanisms, but stop being the user-facing abstraction. Files remain
necessary for persistence, interchange, and low-level operation, and must stay inspectable and
portable — critical user information must never exist only as opaque AI state — but humans
think in activities and subjects, not filenames. LanternOS should investigate a **Context**
abstraction (e.g. "Family finances," "Moving house") associating structured information,
files, conversations, people, history, tasks, permissions, and agent state, so "continue
working on X" resumes the right working set without the user manually reopening six apps.

## The Personal Agent

A Personal Agent is the intermediary between the human and the computing environment —
understanding intent and context, invoking and composing capabilities, creating interfaces on
demand, managing permissions, and escalating to the human only for genuine decisions. It
augments deterministic OS primitives; it does not replace them. See [AI](./AI.md) for how
agency is capability-scoped and auditable at the mechanism level — the Personal Agent is that
mechanism's product-facing role.

**Local AI is the trust boundary.** Private context stays local; the local agent decides what,
if anything, a more powerful external model needs to see, sanitises the request, and
recombines the result with private context afterward. A cloud AI never automatically receives
the user's full digital life just because a task needs stronger inference. See
[Principles](./Principles.md) #2–#3 and [Identity](./Identity.md).

## AI interprets; deterministic components execute

Natural language is probabilistic; system operations must not be. "Back up my projects every
Friday" becomes an explicit, inspectable system object — a Backup Policy with a source,
destination, schedule, failure behaviour, and retention — that the user can view, modify,
disable, and audit. This is the same discipline [AI](./AI.md) already states for capability
grants, applied to every authority-bearing action an agent takes on the user's behalf.

## Progressive disclosure and generated interfaces

The system should make simple things simple ("£4,820") while letting any user descend through
Intent → Result → Details → Operations → API/shell → implementation. A permanent desktop of
windows is not assumed; interface complexity should expand to match the task — a one-line
answer, a generated workspace, or a full conventional spreadsheet, depending on what was
asked. The goal is not to eliminate GUIs; it is to make them contextual rather than permanent.

## Beyond the screen

The same Personal Agent and Context should operate through voice, phones, conventional
monitors, TVs, AR glasses, and spatial displays, adapting along roughly
**Ambient → Conversational → Visual → Spatial → Workstation** as the user's circumstances
change — walking (audio + minimal visual), home (voice + ambient displays), glasses
(audio + vision + spatial overlay), desk (full workstation). With explicit user permission,
vision can supply *context* ("what is that?") while voice supplies *intent* — this must follow
explicit privacy policy and favour local processing; continuous recording of a user's life is
never assumed.

## Attention is a scarce resource

A major design goal is returning human attention to the human: agents should absorb routine
coordination (email triage, status updates, scheduling, chasing responses) and escalate only
decisions, exceptions, and genuinely important information. Work follows the person rather
than requiring the person to stay at a screen — "anything important at work?" while walking
should get a two-sentence answer, with a full workspace available later if the user wants to
go deeper.

## Agent-to-agent computing and minimum disclosure

Long-term, a user's Personal Agent should be able to exchange structured intents and results
with other agents — another person's, an employer's, a hotel's, a bank's — rather than
pretending to be a human operating a website. This implies future protocol needs around
identity, discovery, authorization, capability negotiation, payments, provenance, delegation,
and revocation that current kernel/capability/identity primitives should not foreclose. See
[Security](./Security.md) and [Identity](./Identity.md).

The guiding rule for any such exchange is **minimum necessary disclosure**: reveal only what
an intent requires (dates, headcount, budget for a hotel search) and nothing else (identity,
employer, calendar) by default, preferring proof of a fact ("over 18") over disclosure of the
underlying document ("passport"). LanternOS should support a spectrum of identity —
anonymous, pseudonymous, verified-attribute, verified-person, authorised-representative — for
exactly this reason.

Personal and organisational agents need explicit, auditable trust boundaries: a user should
always be able to answer "what does my employer's agent know about me / what can it access /
what has my agent disclosed / can I revoke that."

## Replacing the desktop, not extending it

The default assumption of Windows/macOS/Linux/Android/iOS — a permanent surface of app icons,
launchers, taskbars, windows, and app-owned data — is not LanternOS's baseline with an agent
added on top. It is the thing being replaced. Before adopting any desktop-era abstraction
(app-as-destination, files as the primary object, a taskbar, a notification centre, an
app-specific assistant, manual copy/paste between programs), ask whether it is fundamental to
computing or merely fundamental to the desktop metaphor. Some will survive — traditional
applications and conventional navigation remain available wherever they're genuinely the best
interface for a task — but they must earn a place under Intent, not be inherited by default.
This is also why the vision leans mobile: work should follow the person (phone, voice, AR
glasses) rather than requiring the person to sit at a screen full of windows, freeing time and
attention rather than consuming more of it (see "Attention is a scarce resource" above).

## What this is not

Not Linux-plus-chatbot. Not a desktop environment with an LLM search box. Not an AI with
unrestricted control of the machine. Not cloud AI APIs wearing an OS costume. Not a project
that removes every conventional interface simply because it is old — traditional applications
and navigation remain available wherever they're genuinely the best interface.

## How this constrains near-term work

This page is explicitly **not** a near-term spec — see [Roadmap](./Roadmap.md): the current
phase is still kernel, IPC, and capabilities. It exists so early primitive choices don't have
to be undone later. When a kernel, capability, or service design decision is on the table, ask:

1. Does this make the kernel smaller, safer, and easier to reason about?
2. Can an agent use this capability without being unnecessarily trusted?
3. Does this support both humans and agents through the same deterministic interface, rather
   than creating a separate privileged "AI path"?

Concretely, this is why [AI](./AI.md) insists agents are capability-scoped confined components
rather than a privileged subsystem, why [`lantern-shell`](https://github.com/lantern-os/lantern-shell) is framed as a
consent broker rather than a window manager, and why [Identity](./Identity.md) targets
partitionable, selectively-disclosable identity rather than one global account.

## Open questions

- Where the Context abstraction actually lives: a service, a capability-set convention, or
  something requiring new kernel/filesystem primitives — undecided; see
  [Filesystem](./Filesystem.md).
- What a minimal Intent Interface looks like before a Personal Agent exists, and how much of
  it `lantern-shell` can gesture at during Phase 2–3 without overbuilding.
- How generated, per-task interfaces are expressed and rendered without becoming another
  privileged "AI path" through the runtime.
- Protocol shape for agent-to-agent intent exchange (identity, discovery, negotiation,
  payments) — deliberately undesigned until [Identity](./Identity.md) and
  [Networking](./Networking.md) mature.
