# 18 · Decoding-time manipulation

[← Back to index](../README.md)

> Attack *how* the model samples, not *what* you ask. Safety measured under one decoding setup does not automatically generalise to others.

These attacks assume more than API prompt access — they require the ability to influence generation settings or token distributions. The lesson for evaluation: alignment is a property of a *decoding configuration*, and contrasting an aligned model with a base model can undo alignment at inference time without any retraining.

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Generation Exploitation** | Yangsibo Huang, Samyak Gupta, Mengzhou Xia, Kai Li, Danqi Chen | 2023 (ICLR 2024) | decoding-time | Changes **how a model samples** its output rather than the semantic request.<br>Alternative generation settings can expose unsafe continuations uncommon under default decoding.<br>Evaluates alignment across multiple sampling configurations.<br>Requires the relevant generation controls; shows safety under one decoding setup doesn't generalise to others. | [arXiv:2310.06987](https://arxiv.org/abs/2310.06987) |
| **Weak-to-Strong Jailbreaking** | Xuandong Zhao, Xianjun Yang, Tianyu Pang et al. | 2024 (ICML 2025) | decoding-time; model-mixture | Two smaller models — one safer, one unsafe — provide a **contrast** that influences a larger model's token distribution.<br>The contrast steers generation away from the larger model's refusal behaviour.<br>Operates during inference rather than retraining the large model.<br>Not ordinary API prompt transfer: the threat model needs access to manipulate decoding probabilities. | [arXiv:2401.17256](https://arxiv.org/abs/2401.17256) |
| **Emulated Disalignment (ED)** | Zhanhui Zhou, Jie Liu, Zhichen Dong et al. | 2024 — ACL | decoding-time; contrastive generation | Contrasts output distributions of a safety-aligned model and its **pretrained counterpart**.<br>Generation is shifted in the direction opposite to the learned safety change.<br>Related to emulating optimisation against a safety reward.<br>Access to aligned and base model outputs can undermine alignment without new fine-tuning. | [ACL 2024](https://aclanthology.org/2024.acl-long.842/) |

---
[← 17 · Agent tools](17-agent-tools-and-propagation.md) · [Index](../README.md) · [Next: 19 · Training & model-level →](19-training-and-model-level.md)
