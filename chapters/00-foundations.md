# 00 · Foundations — why jailbreaks work at all

[← Back to index](../README.md)

> Before the catalogue: the mental models that explain almost every technique that follows.

Almost every attack in this guide, however elaborate its surface, reduces to one of two failure modes that the early safety literature named precisely. The first is **competing objectives**. A deployed model is trained to pursue two goals at once — be helpful and be safe — and those goals do not always point the same way. An attacker who can construct a situation in which refusing *looks like* unhelpfulness, disobedience, or breaking character has turned the model's own training against itself; the safety objective is still present, but it is now competing with a helpfulness objective the attacker has deliberately amplified. The second is **mismatched generalisation**. Capability training covers a vast and varied slice of the input space, while safety training covers a much narrower one. Wherever the model still understands an input that its *safety* data never really exercised — an unusual encoding, a rare format, a nested fiction, a very long context — there is a gap where comprehension outruns caution. The model knows what the input means but has not learned to apply its safety behaviour there.

Reading the chapters that follow through this lens is clarifying. Role-play and persona attacks are competing-objectives attacks: they make compliance feel like staying in character. Encoding, cipher, and low-resource-language attacks are mismatched-generalisation attacks: they move the request into a representation safety training under-covers. Many real techniques blend both. The point of naming the root causes is that defences, too, generalise: a mitigation that genuinely narrows one of these gaps tends to blunt a whole family of attacks at once, rather than patching a single prompt.

The remaining entries in this chapter are not jailbreaks so much as **primitives** — the reusable building blocks that larger, automated attacks assemble. Prefix injection and refusal suppression manipulate the mechanics of autoregressive generation. Universal adversarial triggers introduced the idea of an optimised, transferable input fragment. Attacker-model red teaming introduced the idea of using one model to search for another's failures. None of these is, on its own, a reliable jailbreak against a modern aligned model, and that is precisely why they matter here: they recur as *components* inside the pipelines described later in the book. Understanding them in isolation makes the composite attacks legible, and understanding both root causes makes the whole catalogue feel less like a bag of tricks and more like a small number of structural weaknesses expressed many ways.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (one-liner) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Competing Objectives & Mismatched Generalization** | Alexander Wei, Nika Haghtalab, Jacob Steinhardt | 2023 — NeurIPS | foundational framing | Names the two root causes — helpfulness vs. safety, and safety data covering less of the input space than capability data — that almost every later attack instantiates. | [arXiv:2307.02483](https://arxiv.org/abs/2307.02483) |
| **Prefix Injection & Refusal Suppression** | Community; benchmarked by Wei et al. | 2023 | foundational; prompt-level primitive | Force an affirmative opening and ban the vocabulary of refusal, so autoregressive momentum carries the answer past the usual exits. | [arXiv:2307.02483](https://arxiv.org/abs/2307.02483) |
| **Universal Adversarial Triggers (UAT)** | Eric Wallace, Shi Feng, Nikhil Kandpal, Matt Gardner, Sameer Singh | 2019 — EMNLP-IJCNLP | foundational; gradient/token-level | A shared token sequence optimised to influence many inputs at once — the antecedent of modern suffix attacks. | [arXiv:1908.07125](https://arxiv.org/abs/1908.07125) · [ACL Anthology](https://aclanthology.org/D19-1221/) |
| **Red Teaming Language Models with Language Models** | Ethan Perez et al.; DeepMind & NYU | 2022 — EMNLP | foundational; attacker-LLM search | One model generates test inputs for another; a learned evaluator scores them, searching beyond a fixed human test set. | [arXiv:2202.03286](https://arxiv.org/abs/2202.03286) · [ACL Anthology](https://aclanthology.org/2022.emnlp-main.225/) |

---

## Competing Objectives & Mismatched Generalization

*Alexander Wei, Nika Haghtalab, Jacob Steinhardt · 2023 — NeurIPS · `foundational framing`*

The paper *"Jailbroken: How Does LLM Safety Training Fail?"* is the conceptual anchor for this entire field. Rather than catalogue individual exploits, it asks why safety-trained models fail in the first place, and answers with two root causes that have since become the standard vocabulary. **Competing objectives** describes the tension baked into alignment: a model is optimised to be helpful *and* to be harmless, and an attacker can engineer prompts where those pulls diverge — where producing the refusal the safety policy wants would mean ignoring an instruction, abandoning a role, or otherwise behaving in a way the helpfulness objective treats as failure. The model is not tricked into forgetting its policy so much as placed in a bind its training did not resolve.

**Mismatched generalisation** describes a quieter asymmetry. The data used to teach capabilities is broad; the data used to teach safety is comparatively narrow. Wherever a model's general competence extends into regions the safety data never covered — exotic encodings, unusual formats, rarely seen languages — the model can still interpret the request while failing to recognise it as one it should refuse. Comprehension generalises further than caution.

What made the framing influential is its explanatory reach: the authors show that a large and diverse set of jailbreaks are instances of one or both causes, which turns a sprawling phenomenon into something structured and predictable. For defenders, the lesson is that durable mitigations target the root cause — closing the gap between where a model *understands* and where it *behaves safely* — rather than blacklisting the surface form of yesterday's prompt.

**Source:** [arXiv:2307.02483](https://arxiv.org/abs/2307.02483)

## Prefix Injection & Refusal Suppression

*Community; benchmarked by Wei et al. · 2023 · `foundational`, `prompt-level primitive`*

These are two small, mechanical levers that exploit how language models actually produce text. **Prefix injection** steers the model into beginning its reply with a committed, affirmative phrase. Because generation is autoregressive — each token is conditioned on the ones already emitted — a reply that has already started down a compliant path carries a kind of momentum; having begun to answer, the model is statistically more likely to continue answering than to reverse into a refusal. **Refusal suppression** attacks the same dynamic from the other side by constraining the output away from the words a refusal is built from, so that the usual linguistic exits — the apologies and disclaimers — are harder for the model to reach.

Neither primitive is, on its own, a strong attack against a well-aligned modern model; both are easy to describe and much studied, and defences anticipate them. Their importance is compositional. In automated and multi-turn pipelines they recur as reusable components, quietly tilting the generation dynamics while some other technique does the heavier work of framing. Wei and colleagues benchmarked them precisely to show how such low-level levers contribute to the competing-objectives failure mode.

The defensive reading is that a refusal is not a single gate the model passes through once but a behaviour it must sustain token by token. Mitigations that only check the opening of a response, or that assume a refusal once begun will hold, miss the way these primitives work. Output-side monitoring and training that keeps refusal reachable throughout a generation matter more than any single-point check.

**Source:** [arXiv:2307.02483](https://arxiv.org/abs/2307.02483)

## Universal Adversarial Triggers (UAT)

*Eric Wallace, Shi Feng, Nikhil Kandpal, Matt Gardner, Sameer Singh · 2019 — EMNLP-IJCNLP · `foundational`, `gradient/token-level`*

Universal adversarial triggers introduced an idea that now underpins a whole branch of attacks: a single token sequence can be optimised to influence a model's behaviour not just on one example but across many inputs at once. Instead of crafting a bespoke perturbation per instance, the method searches, using gradient signal, for a short reusable sequence that reliably pushes the model toward a target behaviour wherever it is prepended. The original work demonstrated this across classification and question answering, and crucially included attacks on language generation with GPT-2.

The mechanism exploits reusable weaknesses in a model's learned associations and in how it conditions generation — regularities that hold broadly enough that one optimised trigger can exploit them repeatedly. That transferable, input-agnostic quality is exactly what distinguishes it from a one-off adversarial example, and it is the conceptual antecedent of the modern optimised-suffix attacks discussed later in the book.

It is worth being precise about chronology. This is 2019 work; it predates contemporary chat-model safety training, and citing it is not a claim that the models it attacked had today's alignment policies. Its place in this guide is as a lineage marker: it established that adversarial inputs against text models could be universal and transferable, a property that makes such attacks far more dangerous when they are later aimed at aligned systems. For defenders, it underlines that gradient-based search can find compact, portable triggers, which is why input-level anomaly detection and robustness to out-of-distribution token sequences remain active concerns.

**Source:** [arXiv:1908.07125](https://arxiv.org/abs/1908.07125) · [ACL Anthology](https://aclanthology.org/D19-1221/)

## Red Teaming Language Models with Language Models

*Ethan Perez et al.; DeepMind & NYU · 2022 — EMNLP · `foundational`, `attacker-LLM search`*

This work established the template for automated red teaming: use one language model to find the failures of another. Rather than relying on a fixed set of human-written test cases, a separate *attacker* model generates candidate inputs aimed at eliciting undesirable responses from the target, while a learned classifier evaluates the target's outputs and supplies feedback about which attempts succeeded. The loop can then be driven by several exploration strategies — straightforward generation, supervised adaptation toward inputs that worked, and reinforcement learning to push harder on promising directions.

The significance is methodological. Human-authored test suites are finite and reflect the blind spots of their authors; an attacker model can explore a far larger and stranger region of the input space, surfacing failure modes nobody thought to write down. This is a red-teaming *framework*, not a single universal jailbreak — its output is a process for discovering weaknesses at scale, which is why it reads as foundational rather than as one more technique in the catalogue.

Its lineage runs directly into the later attacker-LLM search chapter, where the same core idea — automated generation plus automated evaluation in a feedback loop — is sharpened into specific, powerful attacks. The defensive takeaway is symmetrical: the same machinery defenders might use to stress-test their own models is available to adversaries, so evaluation should itself be automated, adversarial, and continuous, rather than a one-time checklist against a static set of prompts.

**Source:** [arXiv:2202.03286](https://arxiv.org/abs/2202.03286) · [ACL Anthology](https://aclanthology.org/2022.emnlp-main.225/)

---
[← Index](../README.md) · [Next: 01 · Role-play & nesting →](01-roleplay-and-nesting.md)
