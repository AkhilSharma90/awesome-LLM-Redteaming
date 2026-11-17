# 20 · Confidentiality & availability attacks

[← Back to index](../README.md)

> Not every attack is a refusal bypass. Some leak the system prompt, extract memorised training data, or simply exhaust the model's budget.

Refusal- and correctness-based metrics miss these entirely. The security outcomes here are **confidentiality** (the model discloses something it shouldn't — its instructions or its training data) and **availability** (the model is made slow or expensive rather than made to say something harmful). Treat system prompts as *not* robust secret stores.

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **PLeak — Prompt Leaking** | Bo Hui, Haolin Yuan, Neil Gong, Philippe Burlina, Yinzhi Cao | 2024 — arXiv | prompt-inject; confidentiality | Automatically searches for queries that make an application **disclose its system prompt**.<br>The objective is application-instruction extraction, not generation of disallowed advice.<br>Treats the target as a closed-box application.<br>Tests whether text meant to *control* the application is also exposed as answerable content — system prompts aren't robust secrets. | [arXiv:2405.06823](https://arxiv.org/abs/2405.06823) |
| **Extracting Training Data from LLMs** | Nicholas Carlini, Florian Tramèr, Eric Wallace et al. | 2020 (USENIX Security 2021) | foundational; confidentiality | A model is queried for generated passages that may **reproduce memorised training examples**.<br>Candidate outputs are assessed to distinguish unusually memorised material from ordinary generation.<br>Demonstrates extraction without prior access to the full training dataset.<br>The failure is memorisation and disclosure, not refusal bypass — a separate privacy surface. | [arXiv:2012.07805](https://arxiv.org/abs/2012.07805) |
| **Divergence Attack** (Scalable Extraction) | Milad Nasr, Nicholas Carlini, Jonathan Hayase et al. | 2023 — arXiv | confidentiality; generation-mode disruption | Studies ways to push an aligned chatbot **away from its usual conversational behaviour**.<br>Divergent generation can expose memorised text that normal chat rarely reveals.<br>Connects this behaviour with scalable extraction across access settings.<br>Targets training-data confidentiality; historical results on specific services aren't claims about their current behaviour. | [arXiv:2311.17035](https://arxiv.org/abs/2311.17035) |
| **OverThink** (slowdown attacks) | Abhinav Kumar, Jaechul Roh, Ali Naseh et al. | 2025; rev. 2026 | prompt-inject; availability/resource exhaustion | Untrusted context contains **distracting reasoning tasks** that consume the model's reasoning budget.<br>The assistant may still produce a substantively correct final answer.<br>The adversarial effect is increased computation, latency, or cost — not an obviously harmful response.<br>Shows why correctness and refusal metrics alone miss availability attacks on reasoning applications. | [arXiv:2502.02542](https://arxiv.org/abs/2502.02542) |

---
[← 19 · Training & model-level](19-training-and-model-level.md) · [Index](../README.md) · [Appendices: A](appendix-A-benchmarks-and-tooling.md) · [B](appendix-B-surveys-and-taxonomies.md) · [C](appendix-C-aliases-and-dedup.md) · [D](appendix-D-unverified-leads.md)
