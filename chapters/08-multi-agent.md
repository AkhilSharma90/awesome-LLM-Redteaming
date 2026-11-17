# 08 · Multi-agent & mixture-of-attackers

[← Back to index](../README.md)

> Don't send one attacker — orchestrate a team. Specialised agents plan, attack, judge, and recombine strategies, and the best tactics are *merged*.

The step beyond single-attacker search: by recombining tactics rather than optimising one, these systems compound strengths and coordinate roles. They also increasingly **integrate the rest of this catalogue** as a library of moves — which is why they should be counted once, not re-counted for every underlying attack they invoke.

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **MAD-MAX — Modular And Diverse Malicious Attack MiXtures** | Schoepf et al. | 2025 — arXiv | mixture-of-attackers; automated red team | The clearest "mixture-of-attackers" method.<br>Keeps a modular **attack-style library**, uses LLM agents to cluster which styles are working, then **merges promising attacks together** each iteration to compound strengths.<br>A similarity filter prunes near-duplicates to save queries.<br>Reported large gains over Tree-of-Attacks-with-Pruning in both success rate and efficiency. | [arXiv:2503.06253](https://arxiv.org/abs/2503.06253) |
| **X-Teaming** | Rahman et al. | 2025 — COLM | multi-agent, multi-turn | A collaborative agent pipeline for multi-turn attacks.<br>A **Planner** drafts and adapts diverse attack plans, an **Attacker** executes the conversation, a **Verifier** scores progress, and an **Optimizer** rewrites when stuck.<br>The division of labour lets it plan several turns ahead and recover from refusals.<br>Doubles as a generator of training data for defences. | [arXiv:2504.13203](https://arxiv.org/abs/2504.13203) |
| **RedAgent** | Xu et al. | 2024 — arXiv | context-aware agent; strategy memory | Abstracts the messy zoo of known jailbreaks into reusable **"jailbreak strategies"**.<br>Runs a multi-agent system that selects and composes them for the specific target and context.<br>A self-reflection memory buffer lets it learn from each attempt within a session.<br>Produces context-aware prompts and finds weaknesses efficiently, including in custom GPT-style apps. | [arXiv:2407.16667](https://arxiv.org/abs/2407.16667) |
| **AutoRedTeamer** | Andy Zhou, Kevin Wu, Francesco Pinto, Zhaorun Chen et al. | 2025 — NeurIPS | multi-agent; attacker-LLM search | A red-teaming agent works with a separate **strategy-proposer** agent.<br>Memory guides attack selection, while the proposer incorporates newly described research strategies.<br>It can turn high-level risk categories into test cases.<br>Its contribution is automated integration and orchestration of attacks — not a re-count of every jailbreak it invokes. | [arXiv:2503.15754](https://arxiv.org/abs/2503.15754) |

---
[← 07 · Evolutionary & QD](07-evolutionary-and-quality-diversity.md) · [Index](../README.md) · [Next: 09 · Encoding & obfuscation →](09-encoding-and-obfuscation.md)
