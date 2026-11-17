# 16 · Agent memory & experience poisoning

[← Back to index](../README.md)

> Corrupt what an agent *remembers*. Persistence comes from the agent's own memory-writing and experience-reuse — not from a continuously present external prompt.

An agent's long-term memory and experience store are a trust surface distinct from both its weights and its live context. The most concerning variants require only **ordinary interaction** — the attacker never directly edits storage — yet the planted records resurface in future sessions. Compare the retrieval-triggered and fine-tuned-agent backdoors in [Appendix C](appendix-C-aliases-and-dedup.md).

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **AgentPoison** | Zhaorun Chen, Zhen Xiang, Chaowei Xiao, Dawn Song, Bo Li | 2024 — NeurIPS | agentic/tool; memory poisoning | An attacker poisons an agent's long-term memory or knowledge base with **malicious demonstrations**.<br>Optimised triggers cause associated queries to retrieve those demonstrations.<br>Untriggered activity can remain comparatively unaffected.<br>The backdoor is implemented through retrieval and in-context behaviour, not fine-tuning; stored examples are a separate trust surface. | [arXiv:2407.12784](https://arxiv.org/abs/2407.12784) |
| **MINJA — Memory INJection Attack** | Shen Dong, Shaochen Xu, Pengfei He et al. | 2025; rev. 2026 | agentic/tool; memory poisoning | The attacker interacts through **queries and observed responses** rather than directly editing memory storage.<br>Interactions induce records that connect later benign queries to malicious reasoning patterns.<br>The agent's own memory-writing process makes those records persistent.<br>Lower attacker privilege than direct poisoning: ordinary interaction becomes an entry point into future recalled context. | [arXiv:2503.03704](https://arxiv.org/abs/2503.03704) |
| **MemoryGraft** | Saksham Sahai Srivastava, Haoyu He | 2025 — arXiv | agentic/tool; experience-memory poisoning | Malicious material is incorporated into an agent's store of apparently **successful past experiences**.<br>Later tasks retrieve those experiences as useful procedural examples.<br>The agent imitates the retrieved procedures and can reproduce their unsafe behaviour.<br>The failure is trust in remembered success: persistence comes from experience reuse, not a persistent external prompt. | [arXiv:2512.16962](https://arxiv.org/abs/2512.16962) |

---
[← 15 · Retrieval poisoning](15-retrieval-poisoning.md) · [Index](../README.md) · [Next: 17 · Agent tools & propagation →](17-agent-tools-and-propagation.md)
