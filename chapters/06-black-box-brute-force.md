# 06 · Black-box brute force & adaptive search

[← Back to index](../README.md)

> No cleverness, just volume — perturb the same request thousands of ways and keep sampling until one slips through — or adapt the search to whatever feedback the interface leaks.

The unifying lesson: stochastic sampling and sparse feedback are, on their own, potent attacks. Even a single blocked-or-allowed signal can support adaptive optimisation.

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Best-of-N (BoN) Jailbreaking** | Anthropic & collaborators (Hughes et al.) | 2024 (NeurIPS 2025) | brute force; multimodal | Take one harmful request, apply simple random augmentations — shuffled characters, random capitalisation, typos — and fire many variants until one evades the safeguards.<br>Success grows with sample count along a predictable power-law.<br>It extends beyond text: perturb image backgrounds/fonts for vision models, or pitch/speed/noise for audio.<br>Combining BoN with many-shot further boosts efficiency. | [arXiv:2412.03556](https://arxiv.org/abs/2412.03556) |
| **Simple Adaptive Attacks** | Maksym Andriushchenko, Francesco Croce, Nicolas Flammarion | 2024 (ICLR 2025) | brute force; adaptive search | The framework adapts its search to the particular target and available interface feedback.<br>One component uses randomised suffix changes guided by output likelihood (logprob) information.<br>Other components exploit model-specific interfaces, so it is not one fixed universal payload.<br>Count the adaptive-search methodology here; its prefilling component overlaps with prefix injection. | [arXiv:2404.02151](https://arxiv.org/abs/2404.02151) |
| **BPJ — Boundary Point Jailbreaking** | Davies, Giglemiani, Lau, Winsor, Irving, Gal | 2026 — arXiv | brute force; decision-boundary optimisation | The attack learns from a binary **blocked-or-allowed** signal rather than classifier scores or gradients.<br>Intermediate targets create a curriculum on which small improvements can be measured.<br>The search concentrates evaluation near decision boundaries where feedback is informative.<br>Demonstrates that strong classifier-based safeguards can still be adaptively optimised against with only sparse feedback. | [arXiv:2602.15001](https://arxiv.org/abs/2602.15001) |

---
[← 05 · Gradient & token-level](05-gradient-and-token-level.md) · [Index](../README.md) · [Next: 07 · Evolutionary & QD →](07-evolutionary-and-quality-diversity.md)
