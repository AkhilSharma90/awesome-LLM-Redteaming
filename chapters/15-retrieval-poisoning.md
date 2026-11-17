# 15 · Retrieval & knowledge-base poisoning

[← Back to index](../README.md)

> Corrupt the evidence a RAG system retrieves. A model can fail while apparently *grounding* its answer — because the grounding itself was maliciously constructed.

Retrieval is a separate trust surface from the model. These attacks optimise a small number of corpus passages to be retrieved for target queries, then let the generator do the rest. Note the distinction: retrieval success alone does not prove the generator followed malicious instructions.

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Poisoning Retrieval Corpora with Adversarial Passages** | Zexuan Zhong, Ziqing Huang, Alexander Wettig, Danqi Chen | 2023 — EMNLP | retrieval/corpus poisoning; gradient/token-level | A **small number of corpus passages** are optimised to be retrieved for many questions.<br>Targets the dense retriever's similarity representation.<br>Retrieved content can be irrelevant or attacker-controlled even though it ranks highly.<br>A retrieval-stage integrity attack and precursor to RAG attacks; retrieval success alone ≠ the generator obeyed. | [arXiv:2310.19156](https://arxiv.org/abs/2310.19156) · [EMNLP](https://aclanthology.org/2023.emnlp-main.849/) |
| **PoisonedRAG** | Wei Zou, Runpeng Geng, Binghui Wang, Jinyuan Jia | 2024 (USENIX Security 2025) | retrieval/corpus poisoning | The attacker introduces passages intended to be **retrieved for a target question**.<br>Those passages supply information that steers generation toward an attacker-selected answer.<br>The method jointly considers the retrieval requirement and the generation objective.<br>A model can fail while apparently grounding its answer in evidence — because the evidence is maliciously constructed. | [arXiv:2402.07867](https://arxiv.org/abs/2402.07867) · [USENIX](https://www.usenix.org/conference/usenixsecurity25/presentation/zou-poisonedrag) |
| **BadRAG** | Jiaqi Xue, Mengxin Zheng, Yebowen Hu, Fei Liu, Xun Chen, Qian Lou | 2024; rev. 2026 | retrieval/corpus poisoning; backdoor | Malicious passages are designed to become relevant when a query contains **selected triggers**.<br>Retrieved passages then pursue objectives such as response manipulation, leakage, or tool misuse.<br>Model weights and the legitimate user query need not be modified.<br>Distinguishing feature: trigger-conditioned retrieval behaviour. | [arXiv:2406.00083](https://arxiv.org/abs/2406.00083) |

---
[← 14 · Audio & speech](14-audio-and-speech.md) · [Index](../README.md) · [Next: 16 · Agent memory poisoning →](16-agent-memory-poisoning.md)
