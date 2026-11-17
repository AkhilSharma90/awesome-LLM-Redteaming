# 01 · Role-play, persona & scenario nesting

[← Back to index](../README.md)

> Put the model inside a fiction where the unsafe act belongs to a character, a game, or a nested story — so it answers "in-world."

These are the oldest and most intuitive jailbreaks. They work through **competing objectives**: the model's helpfulness toward a *role* is set against its safety policy. Modern models resist named personas, but the idea survives as a component inside almost every later framing attack.

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **DAN — "Do Anything Now"** | Online community; catalogued by Shen et al. | 2022–23 | role-play; manual | The archetype. The user tells the model to adopt an alter-ego ("DAN") that has "no restrictions", often with a token-economy or threat to keep it in character.<br>It pits helpfulness toward the *role* against the safety policy.<br>Modern models resist named DAN variants.<br>The persona idea survives inside nearly every later framing attack. | [arXiv:2308.03825](https://arxiv.org/abs/2308.03825) |
| **Persona Modulation** | Rusheb Shah et al. | 2023 | role-play; black-box, transferable | A systematised, automated DAN: an attacker model crafts a tailored persona per harmful goal.<br>The target is steered into a character whose "values" make compliance natural.<br>Because the persona is generated per-task, it transfers across models and scales past hand-written role-play.<br>Reported large jumps in harmful-completion rates on multiple commercial models. | [arXiv:2311.03348](https://arxiv.org/abs/2311.03348) |
| **DeepInception** | Xuan Li et al. | 2023 | nested scenes; black-box | "Hypnotise" the model with a deeply **nested fiction** — a dream within a story within a simulation.<br>Each layer adds characters and distance from reality.<br>By the innermost layer the harmful instruction reads as a tiny detail of an elaborate creative task.<br>The nesting is what makes it stronger than a flat role-play. | [arXiv:2311.03191](https://arxiv.org/abs/2311.03191) |
| **ReNeLLM — "A Wolf in Sheep's Clothing"** | Peng Ding et al. | 2023 (NAACL 2024) | generalised nesting | A generalised, automated nesting framework with two moves.<br>**Prompt rewriting:** paraphrase/transform the request to blunt trigger words.<br>**Scenario nesting:** hide the task inside an innocuous container like code completion, filling a table, or text continuation.<br>The model treats the harmful content as structure to complete rather than a request to judge. | [arXiv:2311.08268](https://arxiv.org/abs/2311.08268) |
| **Skeleton Key** | Microsoft (Mark Russinovich) | 2024 — vendor advisory | guideline override; multi-step | Rather than disguise the request, it persuades the model to **amend its own rules**.<br>It asks the model to treat its guidelines as updated so that, instead of refusing, it merely prefixes a warning.<br>Once the model accepts this "augmented behaviour", a wide range of later requests pass.<br>Microsoft disclosed it affecting most major models; mitigation centres on input/output filtering and system-prompt hardening. | [Microsoft Security blog](https://www.microsoft.com/en-us/security/blog/2024/06/26/mitigating-skeleton-key-a-new-type-of-generative-ai-jailbreak-technique/) |

---
[← 00 · Foundations](00-foundations.md) · [Index](../README.md) · [Next: 02 · Multi-turn →](02-multi-turn.md)
