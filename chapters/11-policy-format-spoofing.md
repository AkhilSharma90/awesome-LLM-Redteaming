# 11 · Structural / policy-format spoofing

[← Back to index](../README.md)

> Imitate the *system's own control language* — config files, policies, developer messages — so the model reads the attack as a trusted instruction, not user content.

Models are trained on vast amounts of instruction/policy/configuration data and learn to treat it as authoritative. That learned deference is the vulnerability: wrap a request in the right structure and it is parsed as governance rather than as an ask.

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Policy Puppetry** | HiddenLayer | 2025 — vendor advisory | policy-format spoof; near-universal | Disguises the request as a **policy / configuration file** (XML- or JSON-like) of the kind models treat as authoritative safety guidance.<br>The harmful goal is embedded as a "rule", often with role-play and light encoding like leetspeak.<br>The model parses it as governance it should obey rather than a user ask.<br>HiddenLayer reported a single template working, with minor tweaks, across models from every major lab — attributing it to how models are trained on instruction/policy data. | [HiddenLayer](https://hiddenlayer.com/innovation-hub/novel-universal-bypass-for-all-major-llms/) |

> **Related:** [H-CoT](10-persuasion-and-reasoning-spoofing.md) spoofs a reasoning model's *deliberation* format; the [prompt-injection](12-prompt-injection.md) chapter covers spoofing an application's *developer instructions*.

---
[← 10 · Persuasion](10-persuasion-and-reasoning-spoofing.md) · [Index](../README.md) · [Next: 12 · Prompt injection →](12-prompt-injection.md)
