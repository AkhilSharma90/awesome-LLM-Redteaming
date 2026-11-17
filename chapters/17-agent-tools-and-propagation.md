# 17 · Agent tools, delegation & propagation

[← Back to index](../README.md)

> Poison the metadata the agent reads *before* it ever calls a tool; exploit the gap between approval time and use time; and make the injected instruction **propagate** between agents.

Giving a model tools — functions it can call, servers it can connect to, documents and messages it can act on — opens surfaces that a chat-only model never had. Chief among them is tool *metadata*. Before an agent invokes a tool it typically reads that tool's description and schema, and a model that reads text tends to treat text as instruction. A description is therefore not inert documentation; it is content the model ingests, and if it is attacker-authored it can shape behaviour before the advertised tool is ever called.

Two further problems compound this. The first is a timing gap: trust is usually established when a human reviews and approves a tool, but the metadata the agent acts on is read at use time, and the two moments can diverge. A tool approved yesterday can present different instructions today. The second is propagation: agents increasingly communicate with other agents and exchange documents, so an instruction that compromises one assistant can be carried into the inputs of the next, replicating at the level of *instructions* rather than code. The recurring structure beneath all of these is the confused deputy — the agent lends its own authority and permissions to a request that an untrusted source could not have made on its own.

A note on evidence is important for this chapter in particular. Most entries below are vendor or lab advisories. They describe vulnerabilities reported in specific products at specific times, several with assigned CVEs, and they reflect the disclosing party's analysis. They are not independent reproductions, and a described weakness in a past product version is not proof of ongoing exploitability — vendors patch, and the generalisable mechanism usually outlives the particular product bug. Everything here is kept conceptual: no payloads, no reusable exploit chains, only the shape of each mechanism and the lesson it carries. One naming caution recurs below and is worth flagging up front: EchoLeak is not Echo Chamber (see [Appendix C](appendix-C-aliases-and-dedup.md)).

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Tool Poisoning Attack (TPA) / "Line Jumping"** | Invariant Labs; related disclosure by Trail of Bits | 2025 — vendor/lab advisories | agentic/tool; prompt-inject | Malicious instructions appear inside tool descriptions or other model-visible metadata; the agent may ingest them *before* calling the tool. Description text is promoted into authority over the task. Trail of Bits links "line jumping" to this surface. | [Invariant](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks) · [Trail of Bits](https://blog.trailofbits.com/2025/04/21/jumping-the-line-how-mcp-servers-can-attack-you-before-you-ever-use-them/) |
| **MCP Rug Pull** | Invariant Labs | 2025 — advisory | agentic/tool; supply-chain mutation | A previously approved tool changes its description after trust is established; the updated metadata introduces instructions absent during review, and the agent can act under materially changed guidance. A TPA subpattern exploiting the approval-time/use-time gap. | [Invariant](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks) |
| **MCP Tool Shadowing** | Invariant Labs | 2025 — advisory | agentic/tool; confused-deputy | One server's tool description influences use of another server's tools; the agent combines metadata from sources of differing trust, letting untrusted guidance redirect a legitimate capability. A confused deputy. | [Invariant](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks) |
| **Morris-II** | Stav Cohen, Ron Bitton, Ben Nassi | 2024; rev. 2025 | agentic/tool; prompt-inject; propagation | A prompt designed to influence an assistant *and* cause copies of the malicious content to reach other assistants; communication and retrieval links provide the propagation path. Application-mediated replication of instructions — not a model-weight worm. | [arXiv:2403.02817](https://arxiv.org/abs/2403.02817) |
| **EchoLeak** (LLM Scope Violation) | Aim Labs / Aim Security (Itay Ravia) | 2025 — advisory; CVE-2025-32711 | agentic/tool; prompt-inject; confused-deputy | Attacker-controlled content is retrieved into context alongside legitimate organisational data; the injection influences actions over information beyond the attacker's own content. Generalisable mechanism: a scope violation. (≠ Echo Chamber — see [Appendix C](appendix-C-aliases-and-dedup.md).) | [Cato Networks](https://www.catonetworks.com/blog/breaking-down-echoleak/) |
| **AgentFlayer** | Zenity Labs (incl. Tamir Ishay Sharbat) | 2025 — research disclosures | agentic/tool; prompt-inject; confused-deputy | A family of zero-click agent-exploitation methods using content agents consume through ordinary workflows; injected instructions redirect connected tools, conversations, or stored state. A vendor research umbrella, not one portable payload. | [Zenity](https://zenity.io/research/agentflayer-vulnerabilities) |

---

## Tool Poisoning Attack (TPA) / "Line Jumping"

*Instructions hidden in a tool's own description, read and acted on before the tool is ever invoked.*

A Tool Poisoning Attack places malicious instructions inside the metadata an agent reads about a tool — its description, schema, or similar model-visible fields — rather than inside user input. Because an agent typically loads these descriptions into its context when deciding what to do, the instructions can be ingested before the advertised tool is called, and sometimes whether or not it is ever called. Trail of Bits describes the same surface under the name "line jumping," emphasising that a server can influence an agent simply by being listed, ahead of any actual use.

The mechanism works because a model does not reliably distinguish documentation from instruction. Tool descriptions are supplied by whoever authored the tool or server, which in an open ecosystem like MCP need not be a trusted party, yet they arrive in the same context the model treats as authoritative guidance. Description text is thereby promoted into authority over the task.

This is the root pattern for the two Invariant advisories that follow — Rug Pull and Shadowing are both specialisations of it — so the labels are deliberately kept together here to avoid double-counting the same underlying surface. The defensive lesson is that tool metadata is untrusted input and must be handled as such: isolated from instruction-following, reviewed before a server is trusted, and not granted authority merely because it accompanies a tool the agent intends to use.

**Source:** [Invariant](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks) · [Trail of Bits](https://blog.trailofbits.com/2025/04/21/jumping-the-line-how-mcp-servers-can-attack-you-before-you-ever-use-them/)

## MCP Rug Pull

*A tool that was safe when approved quietly rewrites its description afterward.*

The Rug Pull advisory describes a tool whose description changes after a human has reviewed and approved it. The version seen at approval time is benign; a later version introduces instructions that were not present during review. Because the agent reads the current description at use time, it can end up acting under guidance that no one ever vetted. The attack is purely in the metadata — no change to the agent, the user's request, or the tool's advertised purpose is required.

The reason this is effective is that approval and use are separated in time, and trust is anchored to the earlier moment while behaviour is driven by the later one. A one-time review cannot bind a mutable description. This makes Rug Pull a supply-chain-style mutation layered on top of the basic Tool Poisoning surface: it is a TPA subpattern whose distinguishing feature is *when* the poison appears rather than *where*.

For defenders the implication is concrete: tool definitions must be treated as versioned, mutable artifacts, with integrity checks or re-review triggered when metadata changes rather than trust being granted once and assumed to hold. Pinning and change-detection on tool descriptions address the timing gap directly.

**Source:** [Invariant](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks)

## MCP Tool Shadowing

*One server's description reaches across to steer how the agent uses a different, legitimate server's tools.*

Tool Shadowing describes a situation where the description belonging to one server influences the agent's use of tools provided by another. An agent connected to multiple servers assembles their metadata into a single working context, and that context does not carry per-source trust boundaries. Guidance from a low-trust server can therefore redirect how a capability from a higher-trust server is exercised, even though the malicious server never possessed that capability itself.

This is the clearest confused-deputy case in the chapter. The malicious server cannot perform the sensitive action, but it can supply text that persuades the agent to perform it using the legitimate tool, so the agent lends authority the attacker does not hold. The failure is the absence of trust separation when metadata from different origins is combined.

Alongside Rug Pull, Shadowing is the second specialisation of the Tool Poisoning root: where Rug Pull exploits change over time, Shadowing exploits the mixing of trust levels across servers in one session. The defensive lesson is that an agent's tool context needs provenance-aware boundaries — one server's metadata should not be able to govern another's tools — and that cross-server influence is a risk to be isolated, not assumed away.

**Source:** [Invariant](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks)

## Morris-II

*A prompt engineered not only to subvert one assistant but to seed copies of itself into the next.*

Morris-II is a proof-of-concept for self-propagating prompt injection among AI assistants. The crafted input is designed to do two things: influence the assistant that processes it, and cause copies of the malicious content to be carried into the inputs of other assistants. The propagation path is the ordinary communication and retrieval infrastructure the assistants already use — the demonstration is built around an ecosystem of email assistants, where a message processed by one can reach and influence others.

The mechanism is replication of *instructions*, mediated by applications, and it is important to state precisely what that is and is not. Nothing in the model's weights is modified or copied; the model is not infected in any lasting sense. What spreads is text that triggers the same behaviour again downstream, so each hop depends on another assistant choosing to act on attacker-controlled content. It is a research proof of concept, not a report of an in-the-wild worm outbreak.

Morris-II is the propagation member of this chapter: the other entries compromise a single agent, while this one shows that compromise can chain across agents through the links they share. The defensive lesson is that content exchanged between agents is an untrusted input on both ends, and that systems of cooperating assistants need containment at their boundaries — filtering and provenance on inter-agent messages — so that one compromise cannot cascade.

**Source:** [arXiv:2403.02817](https://arxiv.org/abs/2403.02817)

## EchoLeak (LLM Scope Violation)

*Low-trust retrieved content directs an assistant to act on higher-trust organisational data — a scope violation, not an echo.*

EchoLeak is a disclosed vulnerability, assigned CVE-2025-32711, in which attacker-controlled content is retrieved into an assistant's context alongside legitimate organisational information. The injected content then influences actions involving data that lies outside the attacker's own contribution. The disclosed chain combined this injection with application-specific routes by which information could leave the context; the generalisable mechanism beneath the specific product details is what the researchers term an *LLM scope violation* — low-trust material directing the use of higher-trust data.

It works because an assistant with broad read access flattens trust: content pulled in from an untrusted source sits in the same context as sensitive internal data, and the model does not enforce a boundary between "material I may read" and "material that may direct my actions." That is again a confused-deputy failure, here expressed as a scope boundary that the architecture fails to hold.

A naming caution is essential: EchoLeak is *not* Echo Chamber, a separate and unrelated technique; the two are disambiguated in [Appendix C](appendix-C-aliases-and-dedup.md). As a vendor-reported product vulnerability it was disclosed and addressed, so it should be read as evidence of a mechanism rather than a currently exploitable path. The lasting lesson is scope enforcement: low-trust retrieved content must not be allowed to govern operations over higher-trust data, regardless of their sharing a context window.

**Source:** [Cato Networks](https://www.catonetworks.com/blog/breaking-down-echoleak/)

## AgentFlayer

*An umbrella of zero-click exploit chains that reach agents through the content of everyday business workflows.*

AgentFlayer is a Zenity Labs research umbrella covering a family of zero-click agent-exploitation methods. The common ingredient is content an agent consumes in the normal course of work — documents, messages, and other artifacts flowing through ordinary business processes — carrying injected instructions. Those instructions can redirect the agent's connected tools, steer its conversations, or alter stored state, without a user needing to click or approve anything, because the agent processes the poisoned content automatically.

The shared risk across the family is the familiar confused deputy at agentic scale: an assistant acting with permissions that exceed the attacker's control over the original document or message. The attacker controls a piece of content; the agent contributes the authority. Because the trigger is passive consumption of routine inputs, there is often no human decision point at which the attack could be caught.

It is important to treat AgentFlayer as a research umbrella rather than a single portable payload. It names several distinct exploit chains disclosed as vendor research, each tied to particular products and integrations, not one reusable technique — and, as with the other advisories here, the disclosures reflect reported product weaknesses rather than independently reproduced, currently-live exploits. The defensive lesson draws the chapter together: any content an agent ingests automatically is untrusted input, and agents should operate under least privilege with human-meaningful checkpoints on consequential, connected actions so that passively consumed content cannot silently command high-authority tools.

**Source:** [Zenity](https://zenity.io/research/agentflayer-vulnerabilities)

---
[← 16 · Agent memory poisoning](16-agent-memory-poisoning.md) · [Index](../README.md) · [Next: 18 · Decoding-time →](18-decoding-time.md)
