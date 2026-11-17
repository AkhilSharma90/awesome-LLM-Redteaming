# 07 · Evolutionary & quality-diversity search

[← Back to index](../README.md)

> Treat jailbreak discovery as open-ended evolution — optimise not just for one winner but for a diverse *population* of distinct attacks.

The key insight of this family is **coverage**: optimising only for success collapses to a narrow, misleading set of attacks, so these methods explicitly reward novelty and diversity. The output is a labelled collection ideal for building evaluation suites and defence training data.

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **AutoDAN (genetic)** | Xiaogeng Liu et al. | 2023 (ICLR 2024) | evolutionary; stealthy, readable | Bridges token-optimisation and readability with a **genetic algorithm**.<br>Starts from meaningful hand-written jailbreaks and evolves them via mutation and crossover.<br>A fitness function rewards attack success *and* low perplexity, so results stay fluent and evade perplexity filters that catch GCG's gibberish.<br>A foundational entry in the evolutionary lineage (not the identically-named AutoDAN-Zhu — see [Appendix C](appendix-C-aliases-and-dedup.md)). | [arXiv:2310.04451](https://arxiv.org/abs/2310.04451) |
| **Rainbow Teaming** | Meta, UCL, Oxford (Samvelyan et al.) | 2024 — NeurIPS | quality-diversity | Frames adversarial-prompt generation as a **quality-diversity** problem and runs MAP-Elites.<br>It maintains a grid of prompts spanning chosen feature axes (e.g. risk category × attack style) and keeps improving the best prompt in each cell.<br>The output isn't one exploit but a broad, labelled collection of diverse, effective attacks.<br>A template-method for systematic, coverage-oriented red teaming. | [arXiv:2402.16822](https://arxiv.org/abs/2402.16822) |
| **CRT — Curiosity-driven Red-Teaming** | Zhang-Wei Hong, Idan Shenfeld, Tsun-Hsuan Wang et al. | 2024 — arXiv | QD; reinforcement learning | A red-team model is rewarded for discovering **novel** prompts as well as eliciting undesirable behaviour.<br>Novelty rewards discourage repeatedly exploiting the same narrow failure mode.<br>Broadens coverage of the space of unsafe behaviours.<br>Its value is exploration diversity: success optimisation alone can collapse to a small, misleading set. | [arXiv:2402.19464](https://arxiv.org/abs/2402.19464) |
| **DiveR-CT** | Andrew Zhao, Quentin Xu, Matthieu Lin et al. | 2024 — arXiv | QD; reinforcement learning | Changes the optimisation constraints and diversity incentives used to train a red-team model.<br>Seeks varied attacks without continually maximising an extreme unsafe-response reward.<br>Addresses novelty stagnation and reward over-optimisation.<br>A distinct training objective, not a renamed Rainbow Teaming. | [arXiv:2405.19026](https://arxiv.org/abs/2405.19026) |

---
[← 06 · Black-box brute force](06-black-box-brute-force.md) · [Index](../README.md) · [Next: 08 · Multi-agent →](08-multi-agent.md)
