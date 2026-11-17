# 18 · Decoding-time manipulation

[← Back to index](../README.md)

> Attack *how* the model samples, not *what* you ask. Safety measured under one decoding setup does not automatically generalise to others.

Most of this book concerns what a request says. This chapter is about something orthogonal: how the model turns its internal probabilities into tokens. Alignment, it turns out, is not only a property of the weights — it is also a property of the *decoding configuration* under which those weights are run. A model that refuses reliably under its default sampling settings can behave very differently when the generation controls are changed, which means a safety result proven under one decoding setup does not automatically carry over to another.

A second, sharper idea runs through the chapter: alignment can be undone at inference time by *contrast*. If you can compare an aligned model's output distribution against a base or less-safe model's distribution, the difference encodes much of what alignment added — and steering generation against that difference can push behaviour back toward the unaligned model, all without any retraining. The safety change lives in a distribution shift that a contrast can partly reverse on the fly.

What unites these methods, and what sets them apart from the rest of the book, is their threat model. They assume more than API prompt access: the ability to influence sampling parameters, to mix model outputs, or to read and combine token-level distributions from more than one model. That raises the bar for who can mount them, but it also means the usual defence — inspecting or filtering the prompt — is largely beside the point.

The defensive takeaway is that robustness has to be measured as a property of a *deployed decoding pipeline*, not of a model in the abstract. Safety evaluated at default settings says little about exposed generation controls, and any deployment that lets callers adjust sampling, or that serves logits and base-model variants alongside an aligned model, has widened its attack surface in ways prompt-level guardrails cannot close.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Generation Exploitation** | Yangsibo Huang, Samyak Gupta, Mengzhou Xia, Kai Li, Danqi Chen | 2023 (ICLR 2024) | decoding-time | Changes **how a model samples** its output rather than the semantic request.<br>Alternative generation settings can expose unsafe continuations uncommon under default decoding.<br>Evaluates alignment across multiple sampling configurations.<br>Requires the relevant generation controls; shows safety under one decoding setup doesn't generalise to others. | [arXiv:2310.06987](https://arxiv.org/abs/2310.06987) |
| **Weak-to-Strong Jailbreaking** | Xuandong Zhao, Xianjun Yang, Tianyu Pang et al. | 2024 (ICML 2025) | decoding-time; model-mixture | Two smaller models — one safer, one unsafe — provide a **contrast** that influences a larger model's token distribution.<br>The contrast steers generation away from the larger model's refusal behaviour.<br>Operates during inference rather than retraining the large model.<br>Not ordinary API prompt transfer: the threat model needs access to manipulate decoding probabilities. | [arXiv:2401.17256](https://arxiv.org/abs/2401.17256) |
| **Emulated Disalignment (ED)** | Zhanhui Zhou, Jie Liu, Zhichen Dong et al. | 2024 — ACL | decoding-time; contrastive generation | Contrasts output distributions of a safety-aligned model and its **pretrained counterpart**.<br>Generation is shifted in the direction opposite to the learned safety change.<br>Related to emulating optimisation against a safety reward.<br>Access to aligned and base model outputs can undermine alignment without new fine-tuning. | [ACL 2024](https://aclanthology.org/2024.acl-long.842/) |

---

## Generation Exploitation

*Change the sampling, not the question.*

Generation Exploitation makes the foundational point of the chapter: a model's safety behaviour can depend on *how* it samples, not just on what it is asked. By varying generation settings rather than the semantic content of the request, the work shows that alternative decoding configurations can surface unsafe continuations that are uncommon or unseen under a model's default sampling.

The mechanism is important precisely because it decouples the attack from the prompt. The request can stay fixed while the sampling behaviour changes, so a defence built entirely around inspecting or filtering inputs has no obvious purchase. The study evaluates alignment across multiple sampling configurations rather than a single default, and the finding is that passing a safety test at one configuration does not guarantee passing at others. This requires access to the relevant generation controls, which bounds who can exploit it, but it reframes what a safety claim even means.

For defenders, the lesson is that alignment must be measured as a property of the decoding pipeline that is actually exposed. If callers can adjust generation parameters, those settings are part of the attack surface, and evaluation confined to default decoding will overstate robustness. This work is the conceptual anchor for the two contrastive methods that follow.

**Source:** [arXiv:2310.06987](https://arxiv.org/abs/2310.06987)

## Weak-to-Strong Jailbreaking

*A small-model contrast that pulls a large model off its refusals.*

Weak-to-Strong Jailbreaking uses two smaller models — one safer, one less safe — to form a contrast that influences a larger model's token distribution at inference time. The difference between the two small models provides a directional signal, and applying it steers the large model's generation away from its own refusal behaviour.

The mechanism is notable for where it operates. Nothing retrains the large model; the manipulation happens during decoding, by combining distributions rather than by changing weights. That is also why it falls outside ordinary API prompt transfer: the threat model assumes the ability to manipulate decoding probabilities, not merely to send text and read replies. In that sense it is a concrete instantiation of the chapter's contrast idea — the small-model pair acts as a portable encoding of "less safe," transferred onto a stronger target.

The defensive implication is that serving a model whose token-level probabilities can be combined with those of other models exposes alignment to a decoding-time attack that no prompt filter addresses. It also highlights an asymmetry worth planning for: a comparatively weak pair of models can be enough to degrade the safety of a much larger one, so the strength of the target is not, by itself, a defence.

**Source:** [arXiv:2401.17256](https://arxiv.org/abs/2401.17256)

## Emulated Disalignment (ED)

*Reverse the alignment shift by contrasting the aligned model with its base.*

Emulated Disalignment contrasts the output distributions of a safety-aligned model and its pretrained counterpart. The gap between the two encodes much of what alignment changed; by shifting generation in the direction *opposite* to that learned safety change, ED emulates what a disaligned model would produce — without ever fine-tuning anything.

The mechanism connects to the formal view of alignment as optimisation against a safety reward: contrasting aligned and base outputs approximates reversing that optimisation at inference time. The striking consequence is that the base model, usually released as a benign artifact, becomes an instrument for undoing the aligned model's safety when the two are available together. The learned safety behaviour is recoverable — and reversible — from the distributional difference, so possession of both checkpoints is itself a risk factor.

For defenders, ED sharpens the chapter's central warning about release and serving practices. Shipping an aligned model alongside its pretrained base, or exposing output distributions that permit this kind of contrast, can undermine alignment with no additional training required. It sits beside Weak-to-Strong Jailbreaking as a contrastive, decoding-time method, differing mainly in that the contrast is drawn between a model and its own pre-alignment ancestor rather than between two separate smaller models.

**Source:** [ACL 2024](https://aclanthology.org/2024.acl-long.842/)

---
[← 17 · Agent tools](17-agent-tools-and-propagation.md) · [Index](../README.md) · [Next: 19 · Training & model-level →](19-training-and-model-level.md)
