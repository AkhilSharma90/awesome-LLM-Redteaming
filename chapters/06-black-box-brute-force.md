# 06 · Black-box brute force & adaptive search

[← Back to index](../README.md)

> No cleverness, just volume — perturb the same request thousands of ways and keep sampling until one slips through — or adapt the search to whatever feedback the interface leaks.

The attacks in this chapter abandon the idea that a jailbreak must be *clever*. They treat a safety-trained model as a noisy channel and lean on two cheap resources: stochastic variety and whatever feedback the interface happens to leak. Fire enough superficially different versions of the same forbidden request and some variant lands in a region the safeguards did not cover; the model's own sampling temperature and the brittleness of surface-level filters do the rest. What looks like a solved problem at one query becomes a probabilistic near-certainty at a thousand.

The second idea is that even a *sparse* signal is enough to steer that volume. An attacker rarely has gradients or raw classifier scores against a deployed system, but interfaces leak more than operators intend — a token log-probability here, a prefilled assistant turn there, or merely the binary fact that a request was blocked or allowed. Adaptive search in this family converts those thin signals into a gradient-free optimisation loop, concentrating queries where the feedback is most informative rather than spraying uniformly.

The defensive takeaway is uncomfortable: robustness measured at a single attempt badly overstates real safety. A guardrail that refuses a canonical prompt may still fall to the thousandth perturbation, and a classifier that leaks even a yes/no verdict hands the attacker a curriculum. Evaluators should therefore report success as a function of query budget, treat per-attempt refusal rates as a floor rather than a verdict, and minimise the side-channels — log-probs, prefill, verbose error states — that turn a closed box into a searchable one.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Best-of-N (BoN) Jailbreaking** | Anthropic & collaborators (Hughes et al.) | 2024 (NeurIPS 2025) | brute force; multimodal | Apply simple random augmentations to one harmful request and fire many variants until one evades safeguards; success scales with sample count along a power-law, and extends to image and audio perturbations. | [arXiv:2412.03556](https://arxiv.org/abs/2412.03556) |
| **Simple Adaptive Attacks** | Maksym Andriushchenko, Francesco Croce, Nicolas Flammarion | 2024 (ICLR 2025) | brute force; adaptive search | Adapts the search to the specific target and available interface feedback (e.g. log-prob-guided suffix changes), rather than deploying one fixed universal payload. | [arXiv:2404.02151](https://arxiv.org/abs/2404.02151) |
| **BPJ — Boundary Point Jailbreaking** | Davies, Giglemiani, Lau, Winsor, Irving, Gal | 2026 — arXiv | brute force; decision-boundary optimisation | Learns from only a binary blocked-or-allowed signal, using intermediate targets as a curriculum and concentrating evaluation near decision boundaries where feedback is informative. | [arXiv:2602.15001](https://arxiv.org/abs/2602.15001) |

---

## Best-of-N (BoN) Jailbreaking
*Anthropic & collaborators (Hughes et al.) · 2024 (NeurIPS 2025) · `brute force; multimodal`*

Best-of-N is the purest statement of the volume thesis. It takes a single harmful request and generates a large batch of superficially altered copies using only shallow, composable augmentations — reordered characters, random capitalisation, injected typos and similar surface noise — then submits them until one clears the safeguards. No augmentation is individually ingenious; the method relies on the fact that each one nudges the input into a slightly different part of the model's input space, and the safety training did not cover all of them uniformly.

The mechanism works because the relationship between sample count and success is lawful rather than lucky. Across models, attack success rate climbs with the number of variants along a predictable power-law, which means a motivated attacker can simply *buy* a higher success probability with more queries. Crucially, the approach is modality-agnostic: the same logic applies to perturbing image backgrounds or fonts for a vision model, or pitch, speed and background noise for an audio model, so a single framework stresses an entire multimodal stack. Combining BoN with many-shot prompting reportedly improves efficiency further.

Its demonstration reframed how robustness should be reported. A guardrail judged safe at one attempt can be characterised by the number of samples it survives, turning a binary pass/fail into a cost curve. The defensive lesson is to budget-aware testing — measure how refusal rates decay as query volume grows, apply rate-limiting and cross-request correlation to detect augmentation sprays, and extend input normalisation and filtering to every modality, not just text. (Vendor-reported success figures are cited here as published, not as independently reproduced.)

**Source:** [arXiv:2412.03556](https://arxiv.org/abs/2412.03556)

## Simple Adaptive Attacks
*Maksym Andriushchenko, Francesco Croce, Nicolas Flammarion · 2024 (ICLR 2025) · `brute force; adaptive search`*

Where Best-of-N sprays uniformly, Simple Adaptive Attacks tailors the search to the particular target and to whatever feedback its interface exposes. The framework's central claim is that there is no single universal payload worth chasing; the efficient move is to *adapt*, choosing an exploitation strategy per model and per interface. One component iteratively adjusts a randomised suffix using the model's own output likelihood — log-probability information — as a hill-climbing signal, so each query informs the next instead of being an independent gamble.

Other components exploit model-specific affordances rather than any one weakness, which is precisely the point: the strength of the method is the *methodology* of matching the attack to the surface, not a fixed string that can be filtered. Because of this modularity, the family overlaps with other chapters — its prefilling-style component shares territory with prefix-injection attacks — but the contribution counted here is the adaptive-search loop that turns leaked interface signals into guidance.

The lineage matters: it showed that even simple, well-known optimisation ideas become potent when pointed at the specific signals a deployment leaks, undercutting the assumption that black-box access alone confers safety. The defensive lesson is to treat interface side-channels as part of the attack surface. Log-probabilities, verbose scoring and assistant-prefill features are conveniences that double as optimisation oracles, so exposing them should be a deliberate, risk-weighted decision rather than a default.

**Source:** [arXiv:2404.02151](https://arxiv.org/abs/2404.02151)

## BPJ — Boundary Point Jailbreaking
*Davies, Giglemiani, Lau, Winsor, Irving, Gal · 2026 — arXiv · `brute force; decision-boundary optimisation`*

Boundary Point Jailbreaking pushes the sparse-feedback thesis to its limit: it assumes the attacker sees nothing but a binary verdict — the request was *blocked* or *allowed* — with no classifier scores and no gradients. That single bit seems too thin to optimise against, and BPJ's contribution is to show it is not. By structuring the attack around intermediate targets, it builds a curriculum of progressively harder objectives on which even small improvements become measurable despite the coarse signal.

The mechanism's efficiency comes from *where* it spends its queries. Rather than evaluating points at random, the search concentrates near the decision boundary of the safeguard — the region where a small change can flip the verdict and where the blocked-or-allowed signal is therefore most informative. Points deep inside a safe or unsafe region tell the attacker nothing new; points on the knife-edge carry the most bits per query, so the method clusters its evaluation there.

What it demonstrates is a pointed caution for the current defensive paradigm. Strong classifier-based guardrails are often assumed robust precisely because they expose so little, but BPJ shows that minimal feedback still supports adaptive optimisation against them. The defensive lesson is that opacity is not a durable defence: reducing a classifier's output to a single bit raises the attacker's cost but does not remove the boundary they are probing. Defences should assume the decision surface itself is a target and combine detection, randomisation and query-budget monitoring rather than relying on information-hiding alone.

**Source:** [arXiv:2602.15001](https://arxiv.org/abs/2602.15001)

---
[← 05 · Gradient & token-level](05-gradient-and-token-level.md) · [Index](../README.md) · [Next: 07 · Evolutionary & QD →](07-evolutionary-and-quality-diversity.md)
