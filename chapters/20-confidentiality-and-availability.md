# 20 · Confidentiality & availability attacks

[← Back to index](../README.md)

> Not every attack is a refusal bypass. Some leak the system prompt, extract memorised training data, or simply exhaust the model's budget.

Almost every attack so far in this book has had the same goal: make the model *say* something it was trained to refuse. That framing quietly assumes the only thing worth protecting is the content of the output, and that a safe system is one that refuses the right requests. The attacks in this chapter break that assumption. They do not try to elicit disallowed advice at all. Instead they target two other security properties — confidentiality and availability — that refusal-based and correctness-based metrics are structurally blind to.

Confidentiality attacks make the model disclose something it was supposed to keep. That "something" has two flavours here. One is the application's own instructions — the system prompt — which developers often treat as a hidden control layer and sometimes as a secret asset. The other is memorised fragments of the training data, a privacy surface that exists independent of any guardrail: a model can leak a sequence it saw during training without ever being asked to do anything "harmful." Availability attacks are different again: they do not change *what* the model says, they change what it *costs* to say it, driving up computation, latency, or price until the service degrades. An attack can leave the final answer perfectly correct and still succeed.

The reason these deserve their own chapter is that the usual evaluation lens misses them. A system can refuse every disallowed request, answer every benign one correctly, and still leak its prompt, regurgitate training data, or be driven into a resource-exhaustion spiral. The practical defensive stance follows directly: treat system prompts as *not* robust secret stores and keep nothing in them that would be damaging to reveal; recognise memorisation as a privacy risk to be measured, not assumed away; and monitor for cost and latency anomalies, because an availability attack shows up as a bill or a timeout, never as a policy violation. The summaries below are conceptual; none contains an extraction prompt or a payload. Results about specific historical services describe what was observed then, not claims about those services' current behaviour.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **PLeak — Prompt Leaking** | Bo Hui, Haolin Yuan, Neil Gong, Philippe Burlina, Yinzhi Cao | 2024 — arXiv | prompt-inject; confidentiality | Automatically searches for queries that make an application **disclose its system prompt**.<br>The objective is application-instruction extraction, not generation of disallowed advice.<br>Treats the target as a closed-box application.<br>Tests whether text meant to *control* the application is also exposed as answerable content — system prompts aren't robust secrets. | [arXiv:2405.06823](https://arxiv.org/abs/2405.06823) |
| **Extracting Training Data from LLMs** | Nicholas Carlini, Florian Tramèr, Eric Wallace et al. | 2020 (USENIX Security 2021) | foundational; confidentiality | A model is queried for generated passages that may **reproduce memorised training examples**.<br>Candidate outputs are assessed to distinguish unusually memorised material from ordinary generation.<br>Demonstrates extraction without prior access to the full training dataset.<br>The failure is memorisation and disclosure, not refusal bypass — a separate privacy surface. | [arXiv:2012.07805](https://arxiv.org/abs/2012.07805) |
| **Divergence Attack** (Scalable Extraction) | Milad Nasr, Nicholas Carlini, Jonathan Hayase et al. | 2023 — arXiv | confidentiality; generation-mode disruption | Studies ways to push an aligned chatbot **away from its usual conversational behaviour**.<br>Divergent generation can expose memorised text that normal chat rarely reveals.<br>Connects this behaviour with scalable extraction across access settings.<br>Targets training-data confidentiality; historical results on specific services aren't claims about their current behaviour. | [arXiv:2311.17035](https://arxiv.org/abs/2311.17035) |
| **OverThink** (slowdown attacks) | Abhinav Kumar, Jaechul Roh, Ali Naseh et al. | 2025; rev. 2026 | prompt-inject; availability/resource exhaustion | Untrusted context contains **distracting reasoning tasks** that consume the model's reasoning budget.<br>The assistant may still produce a substantively correct final answer.<br>The adversarial effect is increased computation, latency, or cost — not an obviously harmful response.<br>Shows why correctness and refusal metrics alone miss availability attacks on reasoning applications. | [arXiv:2502.02542](https://arxiv.org/abs/2502.02542) |

---

## PLeak — Prompt Leaking

*An automated search for inputs that make an application disclose its own system prompt.*

PLeak treats prompt leaking as an optimisation problem against a closed-box application. Rather than hand-crafting a single clever request, it automatically searches for queries that cause the deployed application to reveal the system prompt that configures it — the instructions a developer wrote to define the application's persona, constraints, and task. The objective is pure confidentiality: the attack is not trying to elicit disallowed advice, only to make the text intended to *control* the model surface as text the model will *output*.

Why this works at all is a structural feature of current systems. The system prompt and the user's input occupy the same context window and are processed by the same model; there is no hardware boundary making the instructions unreadable to the generation process. An application that can be steered by its prompt can often be steered into reproducing it. Framing the search as automated and closed-box is what makes the finding general — it suggests prompt disclosure is not a one-off trick against a particular app but a reliably discoverable property of many.

The defensive lesson is blunt: a system prompt is not a secret store. Treat its contents as potentially public, keep credentials, keys, and sensitive policy out of it, and do not let security depend on the prompt staying hidden.

**Source:** [arXiv:2405.06823](https://arxiv.org/abs/2405.06823)

## Extracting Training Data from LLMs

*The foundational demonstration that language models memorise training examples and can be made to emit them verbatim.*

This is the foundational study of training-data extraction. It shows that a language model, queried appropriately, will sometimes generate passages that reproduce specific examples from its training set verbatim — including rare or sensitive sequences. The method generates many candidate outputs and then applies measures to separate passages that look unusually memorised from ordinary, fluently-generated text, so that genuine memorised material can be identified among the noise. Crucially, the extraction is demonstrated without the attacker having access to the full training dataset in advance.

Why memorisation happens is intrinsic to how these models are trained: fitting a high-capacity model to a corpus causes some examples, particularly outliers and duplicated ones, to be stored closely enough that they can be regenerated. The security framing is what makes this chapter's point: the failure here is not a refusal that was bypassed. The model may be perfectly willing and perfectly "safe" in the alignment sense, and still leak private data purely because it memorised it. That is a separate axis of risk from content policy.

The defensive lesson is that memorisation is a measurable property to be managed before release — through deduplication, data governance, and in some settings privacy-preserving training — rather than a behaviour a guardrail can refuse away after the fact.

**Source:** [arXiv:2012.07805](https://arxiv.org/abs/2012.07805)

## Divergence Attack (Scalable Extraction)

*Pushing an aligned chatbot out of its normal conversational mode can surface memorised text that ordinary chat hides.*

The Divergence Attack extends training-data extraction to aligned, chat-tuned models. Alignment and the conversational format tend to suppress the kind of raw, verbatim regurgitation that the original extraction work relied on — a well-behaved assistant stays on-task and rarely spills memorised strings during normal conversation. This work studies ways to push the model *away* from its usual conversational behaviour, into a more divergent generation mode in which memorised text is more likely to appear, and connects that behaviour to extraction that scales across different access settings.

Why it works is that the conversational, aligned mode is a learned surface behaviour, not a deletion of the underlying memorisation. The memorised content is still represented in the weights; alignment mainly changes the default style of generation. Disrupting that default can re-expose what the model saw. The important interpretive caution is temporal: the reported results concern specific services as they behaved at the time of study, and are not claims about how those services behave now — vendors can and do patch specific divergence behaviours.

The defensive lesson is that alignment is not a privacy control. Suppressing verbatim output in normal use does not remove the underlying memorisation, so training-data confidentiality must be addressed at the data and training stage, and extraction behaviours monitored as an ongoing risk.

**Source:** [arXiv:2311.17035](https://arxiv.org/abs/2311.17035)

## OverThink (slowdown attacks)

*An availability attack that inflates a reasoning model's compute budget while leaving the final answer correct.*

OverThink targets availability rather than content. In applications where a model reasons over untrusted context — retrieved documents, tool outputs, third-party pages — that context can be laced with distracting reasoning tasks that the model feels compelled to work through. The effect is to consume the model's reasoning budget: it spends far more computation, time, and token cost than the user's actual question warrants. Tellingly, the assistant may still arrive at a substantively correct final answer, so nothing about the output looks wrong.

Why this matters is exactly that invisibility to the usual metrics. A correctness check sees a right answer; a refusal check sees no policy violation; both report success. The adversarial outcome lives in the resource dimension — latency, compute, and bill — which neither metric examines. As reasoning models that "think" at length become common and are priced by tokens consumed, this turns into a denial-of-wallet and denial-of-service surface: an attacker who controls a slice of the context can inflate the cost of every request that touches it.

The defensive lesson is to treat availability as a first-class property of reasoning applications. Budget and cap reasoning length, sanitise or distrust tool- and retrieval-sourced context, and monitor per-request compute and latency for anomalies, because an availability attack will never trip a content filter.

**Source:** [arXiv:2502.02542](https://arxiv.org/abs/2502.02542)

---
[← 19 · Training & model-level](19-training-and-model-level.md) · [Index](../README.md) · [Appendices: A](appendix-A-benchmarks-and-tooling.md) · [B](appendix-B-surveys-and-taxonomies.md) · [C](appendix-C-aliases-and-dedup.md) · [D](appendix-D-unverified-leads.md)
