# 15 · Retrieval & knowledge-base poisoning

[← Back to index](../README.md)

> Corrupt the evidence a RAG system retrieves. A model can fail while apparently *grounding* its answer — because the grounding itself was maliciously constructed.

Retrieval-augmented generation is usually presented as a safety feature: rather than trusting a model's parametric memory, the system fetches supporting passages from a corpus and asks the model to answer from them. That architecture, however, introduces a second trust surface that is easy to overlook. The retriever and the corpus it searches are not the model, and they fail in their own ways. An attacker who can place even a small amount of content into the indexed corpus — a wiki, a support knowledge base, a scraped web crawl, a shared document store — is attacking the evidence, not the weights.

The attacks in this chapter share a common shape. A handful of passages are optimised so that a dense retriever ranks them highly for a chosen set of queries; once retrieved, those passages supply the content from which the model composes its answer. The important consequence is that the model can produce a confidently wrong or attacker-chosen response while *appearing* to do everything right: it cites retrieved text, it stays on topic, it grounds its claims in the context it was given. The grounding is exactly the problem, because the grounding was authored by the adversary.

A second distinction runs through the whole chapter and is easy to lose. Retrieval success is not the same as generation compliance. Showing that a crafted passage is retrieved for a query demonstrates an integrity failure at the retrieval stage; it does not by itself prove the generator obeyed an embedded instruction. The methods below separate these concerns to different degrees, and a careful reader should keep the two claims apart when assessing any reported result. All descriptions here are conceptual: the mechanisms are discussed abstractly, without reproducible passage content or optimisation recipes.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Poisoning Retrieval Corpora with Adversarial Passages** | Zexuan Zhong, Ziqing Huang, Alexander Wettig, Danqi Chen | 2023 — EMNLP | retrieval/corpus poisoning; gradient/token-level | A small number of corpus passages are optimised to be retrieved for many questions; targets the dense retriever's similarity representation. Retrieved content can be irrelevant or attacker-controlled yet rank highly — retrieval success alone ≠ the generator obeyed. | [arXiv:2310.19156](https://arxiv.org/abs/2310.19156) · [EMNLP](https://aclanthology.org/2023.emnlp-main.849/) |
| **PoisonedRAG** | Wei Zou, Runpeng Geng, Binghui Wang, Jinyuan Jia | 2024 (USENIX Security 2025) | retrieval/corpus poisoning | Passages are introduced to be retrieved for a target question and to steer generation toward an attacker-selected answer; the method jointly considers the retrieval requirement and the generation objective. | [arXiv:2402.07867](https://arxiv.org/abs/2402.07867) · [USENIX](https://www.usenix.org/conference/usenixsecurity25/presentation/zou-poisonedrag) |
| **BadRAG** | Jiaqi Xue, Mengxin Zheng, Yebowen Hu, Fei Liu, Xun Chen, Qian Lou | 2024; rev. 2026 | retrieval/corpus poisoning; backdoor | Malicious passages become relevant only when a query contains selected triggers; retrieved passages then pursue objectives such as response manipulation or leakage. No weight or user-query modification required. | [arXiv:2406.00083](https://arxiv.org/abs/2406.00083) |

---

## Poisoning Retrieval Corpora with Adversarial Passages

*A retrieval-stage integrity attack: a few crafted passages that win the similarity contest for many unrelated queries.*

This work established that dense retrievers themselves — independent of any downstream generator — can be corpus-poisoned. The attacker constructs a very small number of passages whose learned vector representations sit close, in the retriever's embedding space, to a broad set of query embeddings. Because dense retrieval ranks documents by representational similarity rather than by surface relevance, a passage can be optimised to score highly for many questions at once, even when its content is irrelevant or attacker-chosen. A token-level, gradient-guided procedure shapes the passage toward that region of the space.

Why it works is a property of the retrieval model, not a bug in any one application. The retriever compresses meaning into a single similarity geometry, and that geometry has regions that are close to large neighbourhoods of queries. An adversary with access to (or a good approximation of) the retriever can search for passages that land there. The legitimate corpus, the user's query, and the model weights all stay untouched.

The contribution for a defender is conceptual clarity: it isolates the retrieval stage as a standalone attack surface and a precursor to the full RAG attacks that follow. It also underlines the chapter's recurring caution — demonstrating that a passage is retrieved is an integrity result about the retriever, and says nothing on its own about whether a generator would then act on it. Treating the index as untrusted input, and monitoring for passages that are anomalously retrievable across diverse queries, follows directly.

**Source:** [arXiv:2310.19156](https://arxiv.org/abs/2310.19156) · [EMNLP](https://aclanthology.org/2023.emnlp-main.849/)

## PoisonedRAG

*Closes the loop from retrieval to generation: passages crafted both to be fetched for a target question and to steer the answer.*

Where the adversarial-passages work stops at the retriever, PoisonedRAG treats the end-to-end RAG pipeline as the target and makes explicit the two conditions a successful poison passage must satisfy at once. First, the passage must be retrieved when a particular target question is asked — a retrieval condition. Second, once in the model's context, it must supply information that steers generation toward an attacker-selected answer — a generation condition. The method is built around jointly satisfying both, rather than optimising retrieval and hoping the generator complies.

The reason this is effective is the same reason RAG is attractive in the first place. A model instructed to answer from provided context will privilege that context over its own priors; when the context is adversarial, that deference becomes the vulnerability. The system behaves exactly as designed — grounding its answer in retrieved evidence — and arrives at the wrong place because the evidence was authored to lead it there. A model can thus fail while appearing maximally well-behaved.

This is the canonical statement of the chapter's thesis: a confident, grounded, on-topic answer can be a successful attack outcome. Its lineage runs forward into the trigger-conditioned and agentic variants that follow, and its defensive lesson is that provenance and integrity of retrieved context matter as much as the model's own alignment. Guarding the generator alone leaves the pipeline open.

**Source:** [arXiv:2402.07867](https://arxiv.org/abs/2402.07867) · [USENIX](https://www.usenix.org/conference/usenixsecurity25/presentation/zou-poisonedrag)

## BadRAG

*A backdoor in the corpus: poison passages stay dormant until a query carries the attacker's trigger.*

BadRAG adds a conditioning mechanism to corpus poisoning. Rather than passages that are always retrieved, the malicious content is designed to become relevant only when a query contains selected trigger features. For ordinary, trigger-free queries the corpus behaves normally; when a triggering query arrives, the crafted passages surface and pursue the attacker's objective — which the authors frame broadly, spanning response manipulation, information leakage, or steering toward unsafe tool use. As with the other methods here, neither the model weights nor the legitimate user query need to be modified.

The trigger-conditioned behaviour is what distinguishes it from unconditional poisoning and also what makes it harder to detect. Because the poison is quiescent under normal evaluation, routine quality testing of a RAG system can pass cleanly while the backdoor remains latent in the index. The attack lives entirely in retrieved content, which keeps it outside the scope of defences aimed at weights or at the user-facing prompt.

For defenders the lesson sharpens the previous two: corpus integrity must be assessed against adversarial, not just typical, queries, and the mere absence of misbehaviour under normal traffic is not evidence of a clean index. Trigger-conditioning is the retrieval-corpus analogue of a classic data-poisoning backdoor, and it generalises the static poisons above into something stealthier.

**Source:** [arXiv:2406.00083](https://arxiv.org/abs/2406.00083)

---
[← 14 · Audio & speech](14-audio-and-speech.md) · [Index](../README.md) · [Next: 16 · Agent memory poisoning →](16-agent-memory-poisoning.md)
