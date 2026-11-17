# 11 · Structural / policy-format spoofing

[← Back to index](../README.md)

> Imitate the *system's own control language* — config files, policies, developer messages — so the model reads the attack as a trusted instruction, not user content.

A language model is trained on an enormous amount of text that is *about control*: configuration files, policy documents, access-control rules, developer messages, system prompts, specifications that say what is and is not permitted. From all of it the model learns a strong prior that text written in that register is authoritative — that a structured block of rules is something to be obeyed, not merely something a user happens to have typed. Instruction tuning reinforces the prior, because it teaches the model to defer to governing instructions over conversational content.

That learned deference is the vulnerability this chapter isolates. If a model treats control-language as governance by virtue of its *form*, then an attacker who can produce convincing control-language can smuggle a request into the governance channel. The payload is not hidden and not encoded; it is re-labelled. Wrapped in the right structure, a disallowed request stops reading like an ask the model should weigh against policy and starts reading like policy the model should enforce — the request is laundered through the shape of authority rather than the substance of persuasion.

The defensive problem is that authority, for a current model, is inferred from surface form rather than verified from provenance. The model has no reliable way to confirm that a block that *looks* like a system policy actually came from the system. The durable lesson is that trust must be established out of band — bound to the channel a message arrives on and cryptographically or architecturally enforced — never granted on the basis of formatting alone. Until a model can tell genuine governance from an imitation of it, any text that can mimic the control plane is a candidate attack.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Policy Puppetry** | HiddenLayer | 2025 — vendor advisory | policy-format spoof; near-universal | Disguises a request as a policy/configuration file of the kind models treat as authoritative guidance, so it is parsed as governance rather than a user ask. | [HiddenLayer](https://hiddenlayer.com/innovation-hub/novel-universal-bypass-for-all-major-llms/) |

> **Related:** [H-CoT](10-persuasion-and-reasoning-spoofing.md) spoofs a reasoning model's *deliberation* format; the [prompt-injection](12-prompt-injection.md) chapter covers spoofing an application's *developer instructions*.

---

## Policy Puppetry
*A request re-dressed as the system's own safety policy, parsed as governance rather than as an ask.*

Policy Puppetry, disclosed by the security vendor HiddenLayer, disguises a request as a policy or configuration file — structured in an XML- or JSON-like form of the kind models are trained to treat as authoritative safety guidance. The harmful objective is embedded not as a question but as a "rule" inside that structure, frequently combined with role-play and light surface obfuscation such as leetspeak. The effect is that the model parses the input as governance it is meant to apply, rather than as a user request it should evaluate against its policy.

The mechanism is a direct consequence of how models are trained. Having seen vast quantities of instruction, policy, and configuration data, a model learns to grant that register a presumption of authority; it cannot verify that a policy-shaped block genuinely originates from the system, so it defers to the form. Policy Puppetry exploits exactly this: by occupying the governance register, the request bypasses the deliberation the model would ordinarily apply to an ordinary ask. The role-play and encoding are secondary — they help steer the output — but the core move is the structural impersonation of the control plane.

What made the disclosure notable is its claimed breadth. HiddenLayer reported that a single template, with only minor adjustments, worked across models from every major lab, and attributed that near-universality to the common training exposure to instruction and policy data shared across the industry — a systemic weakness rather than a single vendor's bug. This places it alongside the other spoofing attacks in this guide: where [H-CoT](10-persuasion-and-reasoning-spoofing.md) imitates a reasoning model's deliberation format and [prompt injection](12-prompt-injection.md) imitates an application's developer instructions, Policy Puppetry imitates the safety policy itself. The defensive lesson is that formatting must never confer trust. A model and the system around it need to bind authority to verified provenance — to the channel a message actually arrived on — so that text imitating a policy file is treated as the untrusted user content it is.

**Source:** [HiddenLayer](https://hiddenlayer.com/innovation-hub/novel-universal-bypass-for-all-major-llms/)

---
[← 10 · Persuasion](10-persuasion-and-reasoning-spoofing.md) · [Index](../README.md) · [Next: 12 · Prompt injection →](12-prompt-injection.md)
