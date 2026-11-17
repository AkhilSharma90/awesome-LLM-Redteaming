# 07 · Evolutionary & quality-diversity search

[← Back to index](../README.md)

> Treat jailbreak discovery as open-ended evolution — optimise not just for one winner but for a diverse *population* of distinct attacks.

The methods in this chapter borrow a lesson from evolutionary computation: optimising only for a single best outcome produces a narrow, brittle, and ultimately misleading result. A red-team search that chases maximum attack success alone tends to collapse onto one exploitable quirk, polishing a single prompt while vast regions of the failure space go unprobed. The alternative is to evolve a *population* — to maintain many candidate attacks at once and reward them not just for working but for being *different* from one another.

The unifying idea is therefore **coverage**. By defining axes of variation — risk categories, rhetorical styles, attack strategies — and explicitly rewarding novelty, these systems spread their effort across the space of possible failures instead of converging prematurely. Genetic methods evolve readable prompts through mutation and crossover under a fitness function that balances success against other constraints; quality-diversity methods keep a structured archive and improve the best candidate in each distinct niche; reinforcement-learning variants reward a red-team policy for surfacing previously unseen behaviours. In every case the search is open-ended rather than greedy.

This reframing changes what red teaming *produces*. The output is not a single exploit to be patched but a broad, labelled collection of diverse, effective attacks — a map of a model's weaknesses organised by category and style. That artefact is exactly what defenders need downstream: it seeds evaluation suites that measure coverage rather than a single pass/fail, and it supplies varied, well-characterised training data for safety fine-tuning. The defensive takeaway is that any red-team result reported as one number hides its own blind spots; coverage-oriented search both exposes those blind spots and gives you the material to close them.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **AutoDAN (genetic)** | Xiaogeng Liu et al. | 2023 (ICLR 2024) | evolutionary; stealthy, readable | A genetic algorithm evolves hand-written jailbreaks via mutation and crossover under a fitness function that rewards success *and* low perplexity, keeping results fluent and evading perplexity filters. | [arXiv:2310.04451](https://arxiv.org/abs/2310.04451) |
| **Rainbow Teaming** | Meta, UCL, Oxford (Samvelyan et al.) | 2024 — NeurIPS | quality-diversity | Frames prompt generation as quality-diversity and runs MAP-Elites over a grid of feature axes (e.g. risk category × attack style), improving the best prompt per cell to yield a broad labelled collection. | [arXiv:2402.16822](https://arxiv.org/abs/2402.16822) |
| **CRT — Curiosity-driven Red-Teaming** | Zhang-Wei Hong, Idan Shenfeld, Tsun-Hsuan Wang et al. | 2024 — arXiv | QD; reinforcement learning | Rewards a red-team model for discovering *novel* prompts as well as eliciting undesirable behaviour, so it broadens coverage instead of collapsing onto one narrow failure mode. | [arXiv:2402.19464](https://arxiv.org/abs/2402.19464) |
| **DiveR-CT** | Andrew Zhao, Quentin Xu, Matthieu Lin et al. | 2024 — arXiv | QD; reinforcement learning | Changes the constraints and diversity incentives used to train a red-team model, seeking varied attacks without over-maximising an extreme unsafe-response reward; addresses novelty stagnation. | [arXiv:2405.19026](https://arxiv.org/abs/2405.19026) |

---

## AutoDAN (genetic)
*Xiaogeng Liu et al. · 2023 (ICLR 2024) · `evolutionary; stealthy, readable`*

AutoDAN bridges two previously opposed goals: the automation of token-level optimisation and the fluency of hand-crafted prompts. Earlier gradient-based methods could search for effective adversarial strings but produced gibberish that a simple perplexity filter catches. AutoDAN replaces that search with a **genetic algorithm** seeded from meaningful, human-written jailbreaks, then evolves them through mutation and crossover so the population drifts toward new attacks while remaining linguistically coherent.

The mechanism's leverage is its fitness function, which rewards two things at once: attack success *and* low perplexity. Optimising for both keeps the evolved prompts fluent, which is precisely why they evade the perplexity-based detectors that reliably flag the output of gibberish-producing optimisers. In other words, AutoDAN treats readability not as a cosmetic nicety but as a first-class objective, closing a defensive gap that earlier automated attacks left open.

As a lineage marker, AutoDAN is a foundational entry in the evolutionary branch of this survey — the point where population-based search and stealth were joined. (It should not be confused with the identically named AutoDAN-Zhu, a separate line of work; see [Appendix C](appendix-C-aliases-and-dedup.md).) The defensive lesson is that perplexity filtering alone is a fragile line of defence: once fluency is an explicit optimisation target, the detector it relies on is designed around rather than tripped. Robust defences need semantic and behavioural signals, not surface statistics.

**Source:** [arXiv:2310.04451](https://arxiv.org/abs/2310.04451)

## Rainbow Teaming
*Meta, UCL, Oxford (Samvelyan et al.) · 2024 — NeurIPS · `quality-diversity`*

Rainbow Teaming is the template method for coverage-oriented red teaming. It frames adversarial-prompt generation explicitly as a **quality-diversity** problem and runs MAP-Elites, an algorithm designed to populate an archive of high-performing, meaningfully distinct solutions rather than to find a single optimum. The archive is a grid whose axes are chosen features of an attack — for example risk category crossed with attack style — so each cell represents a qualitatively different kind of prompt.

The mechanism improves the best prompt occupying each cell over successive iterations, which guarantees that the search keeps investing in under-covered regions instead of piling further effort onto whatever worked first. This is why it works as a mapping tool: the output is not one exploit but a broad, labelled collection of diverse and effective attacks, each tagged by the cell it came from. That structure is directly reusable — it reads as a coverage matrix of a model's weaknesses.

Its contribution to the lineage is methodological: it imported a mature quality-diversity algorithm into safety red teaming and showed that systematic, axis-driven exploration beats ad-hoc prompt collection. The defensive lesson follows from the artefact it produces. Because the grid makes blind spots visible — empty or low-scoring cells are categories you have not stress-tested — defenders can use the same framing to audit coverage and to generate balanced, well-labelled training data for hardening across every category and style, not just the easy ones.

**Source:** [arXiv:2402.16822](https://arxiv.org/abs/2402.16822)

## CRT — Curiosity-driven Red-Teaming
*Zhang-Wei Hong, Idan Shenfeld, Tsun-Hsuan Wang et al. · 2024 — arXiv · `QD; reinforcement learning`*

Curiosity-driven Red-Teaming applies the coverage principle inside a reinforcement-learning loop. A red-team model is trained to generate prompts, but its reward is deliberately split: it earns credit both for eliciting undesirable behaviour from the target *and* for discovering prompts that are **novel** relative to what it has already produced. The curiosity term imports an idea from exploration-focused RL — reward the agent for surprise — and repurposes it as a defence-testing objective.

The mechanism matters because a pure success reward has a well-known failure mode: the policy finds one reliable trick and exploits it endlessly, since repeating a known win is the safest way to maximise return. The novelty reward actively penalises that collapse, pushing the model away from prompts resembling its past attempts and toward unexplored corners of the behaviour space. The result is broader coverage of the distinct ways a model can be made to misbehave, rather than a deep rut around a single one.

In the lineage, CRT sits alongside Rainbow Teaming as a diversity-first method but reaches the goal through learned exploration rather than an explicit archive. Its demonstrated value is exactly that exploration diversity: success optimisation alone can collapse to a small, misleading set of attacks that flatters a model's apparent safety. The defensive lesson is to judge red-team outputs by the breadth of behaviours they surface, and to be wary of evaluations whose attacks all cluster around one mode.

**Source:** [arXiv:2402.19464](https://arxiv.org/abs/2402.19464)

## DiveR-CT
*Andrew Zhao, Quentin Xu, Matthieu Lin et al. · 2024 — arXiv · `QD; reinforcement learning`*

DiveR-CT also trains a red-team model with diversity in mind, but its contribution is a reworking of the optimisation itself rather than a new archive or curiosity bonus. It changes the constraints and diversity incentives under which the red-team policy is trained, with the explicit aim of producing varied attacks *without* continually driving an extreme unsafe-response reward to its maximum. The distinction is subtle but real: it is a different training objective, not a renamed Rainbow Teaming.

The mechanism targets two coupled pathologies of diversity-rewarded RL. The first is novelty stagnation, where a model that was exploring eventually settles and stops finding genuinely new attacks. The second is reward over-optimisation, where relentlessly maximising the unsafe-response signal distorts the policy and narrows its output. By rebalancing constraints against incentives, DiveR-CT seeks sustained variety without letting either pathology dominate, keeping the generated population both diverse and meaningful.

Within the lineage it is best read as a refinement of the RL-based diversity methods, addressing the ways their training dynamics degrade over a long run. The defensive lesson is cautionary for anyone building automated red teams: the way you *measure and reward* an attacker shapes what it finds, and naive maximisation of an unsafe-response score can quietly reduce coverage even as the headline reward climbs. Robust evaluation requires objectives that preserve diversity under optimisation pressure.

**Source:** [arXiv:2405.19026](https://arxiv.org/abs/2405.19026)

---
[← 06 · Black-box brute force](06-black-box-brute-force.md) · [Index](../README.md) · [Next: 08 · Multi-agent →](08-multi-agent.md)
