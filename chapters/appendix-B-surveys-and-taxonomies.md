# Appendix B · Surveys & taxonomies

[← Back to index](../README.md)

> The reading maps. Use these to structure background chapters, cross-reference families, and ground threat-model terminology — they are taxonomy sources, not additional attack primitives.

The works below are not attacks; they are the surveys and taxonomies that organise the field. Where the rest of this guide catalogues individual methods, these sources step back and impose structure — grouping techniques by attacker access, lifecycle stage, or threat objective, and defining the shared vocabulary the literature uses to talk about all of them. Reach for them when you want the shape of the forest rather than a single tree.

In practice, use a survey to build the background or framing sections of your own study: start from its categories, then slot the catalogue's method entries into them and follow its citations outward to the primary work. When you need a threat-model backbone — a stable way to describe *who* the attacker is, *what* they can touch, and *what* they are trying to achieve — the **NIST AI 100-2** taxonomy is the most durable anchor here; its lifecycle-and-objective framing maps cleanly onto the attack families elsewhere in this guide and onto the defences in Appendix A. Because a taxonomy reorganises existing knowledge rather than adding to it, none of these entries counts as an additional attack primitive.

| Work | Origin | Year / venue | What it offers | Link |
| --- | --- | --- | --- | --- |
| **Jailbreak Attacks and Defenses Against LLMs: A Survey** | Sibo Yi, Yule Liu, Zhen Sun et al. | 2024 — arXiv | Organises attacks by target access and finer method categories; separates prompt-level from model-level defences. | [arXiv:2407.04295](https://arxiv.org/abs/2407.04295) |
| **JailbreakRadar — Comprehensive Assessment of Jailbreak Attacks** | Chu, Liu, Yang, Shen, Backes, Zhang | 2024 (ACL 2025) | Compares representative methods under a shared evaluation setup, with ablations and tests against multiple defences. | [arXiv:2402.05668](https://arxiv.org/abs/2402.05668) |
| **Red-Teaming for Generative AI: Silver Bullet or Security Theater?** | Feffer, Sinha, Deng, Lipton, Heidari | 2024 — AIES | Surveys what practitioners mean by "AI red teaming"; separates evaluation purpose, artifact, setting, and decisions. | [arXiv:2401.15897](https://arxiv.org/abs/2401.15897) |
| **NIST AI 100-2 E2025 — Adversarial ML: A Taxonomy and Terminology** | Vassilev, Oprea, Fordyce, Anderson, Davies, Hamin; NIST | 2025 — NIST report | Organises adversarial ML by lifecycle stage, attacker knowledge, capability, and objective; covers poisoning, evasion, privacy, and generative-AI threats. A stable threat-model backbone. | [NIST](https://csrc.nist.gov/pubs/ai/100/2/e2025/final) · [DOI](https://doi.org/10.6028/NIST.AI.100-2e2025) |
| **Jailbreaking LLMs & VLMs: Mechanisms, Evaluation, and Unified Defense** | Chen, Li, Li, Zhang, Zhang, He | 2026 — arXiv | Covers text-only and vision-language jailbreaking across attack, defence, and evaluation dimensions. A newer cross-modal reading map. | [arXiv:2601.03594](https://arxiv.org/abs/2601.03594) |

---
[← A · Benchmarks & tooling](appendix-A-benchmarks-and-tooling.md) · [Index](../README.md) · [C · Aliases & dedup →](appendix-C-aliases-and-dedup.md)
