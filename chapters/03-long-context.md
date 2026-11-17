# 03 · Long-context & in-context conditioning

[← Back to index](../README.md)

> Use the sheer size of the context window as the weapon: show the model so many examples of "complying" — or so much distracting material — that it just continues the pattern.

The attacks in this chapter weaponise the context window itself. As models grew windows measured in the hundreds of thousands of tokens, the prompt stopped being a short instruction and became a space large enough to *re-teach* the model, inside a single request, how to behave. These techniques exploit **in-context learning** — the model's ability to infer the rules of the current exchange from the material in front of it — together with the subtler fact that where content sits in a long prompt, and how much cognitive load that prompt imposes, can change whether safety behaviour fires at all.

Seen through the book's two root causes, this family draws on both. Part of it is **competing objectives**: a long transcript of an assistant answering harmful questions willingly creates a powerful local pull to continue the demonstrated pattern, pitting pattern-continuation against policy. Part of it is **mismatched generalisation**: safety training rarely exercises the extremes of very long, heavily loaded, or unusually structured contexts, so the model's comprehension of such inputs outruns its safety behaviour there. The techniques differ in which lever they lean on — some flood the context with fabricated compliance, some bury a single request in benign filler, some overload the model with demanding interpretation tasks, and some poison the demonstrated *reasoning* rather than the answers — but all of them treat the context as something that conditions the model, not merely something it reads.

The defensive through-line is that safety has to be robust across the whole window, not just at its edges. A model that behaves well on short prompts can still be steered by a long enough demonstration, a distant enough placement, or a heavy enough load. Mitigations therefore have to account for context length and position directly: capping or discounting the influence of in-context demonstrations, monitoring for conditioning patterns, and recognising that benign-looking bulk can itself be an attack vector. Several of the entries below were published with mitigations, and the two similarly named "cognitive overload" lines of work — easy to conflate — are kept distinct here precisely because clear taxonomy is part of a usable defence.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (one-liner) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Many-shot Jailbreaking (MSJ)** | Anthropic | 2024 (NeurIPS 2024) | long-context; in-context learning | Fill the prompt with many fabricated turns of an assistant answering harmful questions willingly, then ask the real one; success scales with shot count. | [Anthropic](https://www.anthropic.com/research/many-shot-jailbreaking) |
| **Cognitive Overload** (Xu et al.) | Nan Xu, Fei Wang, Ben Zhou, Bangzheng Li, Chaowei Xiao, Muhao Chen | 2023 (Findings of NAACL 2024) | task-overload; encoding/obfuscation | Stack demanding interpretation tasks — multilingual, veiled meaning, reversed causal reasoning — so intent is harder to recognise. | [arXiv:2311.09827](https://arxiv.org/abs/2311.09827) · [Findings](https://aclanthology.org/2024.findings-naacl.224/) |
| **Cognitive Overload Attack** (Upadhayay et al.) | Bibek Upadhayay, Vahid Behzadan, Amin Karbasi | 2024 — arXiv | long-context; task-overload | Deliberately overloaded prompts in in-context settings make complex demands compete with reliable application of safety instructions. See [Appendix C](appendix-C-aliases-and-dedup.md). | [arXiv:2410.11272](https://arxiv.org/abs/2410.11272) |
| **NINJA — Needle-in-haystack Jailbreak** | Shah, Wu, Saxena, Zhong, Robey, Raghunathan | 2025 preprint; rev. 2026 | long-context | Embed a harmful goal in a much larger body of benign model-generated material; position within that context affects whether the model refuses. | [arXiv:2511.04707](https://arxiv.org/abs/2511.04707) |
| **BadChain — Backdoor Chain-of-Thought** | Xiang, Jiang, Xiong, Ramasubramanian, Poovendran, Li | 2024 — ICLR | in-context poisoning; reasoning manipulation | Demonstrations tie a trigger to an extra malicious reasoning step the model copies when the trigger appears — no weight changes. | [arXiv:2401.12242](https://arxiv.org/abs/2401.12242) |

---

## Many-shot Jailbreaking (MSJ)

*Anthropic · 2024 (NeurIPS 2024) · `long-context`, `in-context learning`*

Many-shot jailbreaking is the clearest demonstration that a context window can be used to overwrite the rules of a conversation. The attack fills the prompt with a long, fabricated transcript in which an assistant is shown answering many harmful questions willingly and without hesitation, and then appends the attacker's real question at the end. The model's in-context learning does the rest: from dozens or hundreds of consistent demonstrations it infers "the rules of this conversation" — here, that this assistant simply complies — and continues in kind.

The mechanism is competing objectives amplified by scale. Each demonstration adds weight to the pattern of compliance, so the pull to continue that pattern grows with the number of shots, eventually overwhelming the safety behaviour that a short prompt would have triggered. A central finding is that effectiveness scales with the number of shots along a predictable curve, which reframes the attack from a lucky trick into a quantity with a dependable relationship to context length — and ties the vulnerability directly to the growth of context windows that made it possible.

Anthropic published many-shot jailbreaking together with mitigations reported to sharply reduce its success, and the work is notable as much for the defensive framing as for the attack. Because the effect is predictable in the number of shots, defences can target that very relationship — for instance by limiting or discounting how strongly long runs of in-context demonstrations are allowed to shape behaviour. It stands as the canonical long-context conditioning attack and the reference point against which the other entries in this chapter are distinguished: NINJA differs by *not* requiring demonstrations of compliance, and BadChain differs by poisoning reasoning rather than answers.

**Source:** [Anthropic](https://www.anthropic.com/research/many-shot-jailbreaking)

## Cognitive Overload (Xu et al.)

*Nan Xu, Fei Wang, Ben Zhou, Bangzheng Li, Chaowei Xiao, Muhao Chen · 2023 (Findings of NAACL 2024) · `task-overload`, `encoding/obfuscation`*

This line of work explores what happens to safety behaviour when the model is made to work hard at *interpreting* the input. The authors combine demanding interpretation tasks — multilingual processing, veiled or indirect meaning, and reversed causal reasoning — so that recovering the actual intent of a request requires substantial effort. In doing so they change the context in which safety-relevant intent has to be recognised: the harmful meaning is still present, but it is reached only after the model has spent its interpretive capacity on the surrounding complexity.

The mechanism leans on mismatched generalisation. Safety training is best exercised on clear, direct expressions of harmful intent; when that intent is only recoverable through layered interpretation, the model's comprehension can still arrive at the meaning while its safety response fails to fire. The authors frame their results through a **cognitive-overload analogy** — the idea that piling on interpretive demands degrades safe behaviour the way overload degrades human performance. It is important to treat that analogy as explanatory framing, a way of organising the observations, and not as a claim that the model possesses human-like cognition.

This work should be kept carefully distinct from the later, similarly named long-context study by Upadhayay et al.; the shared name invites confusion, but the two are separate contributions. Its defensive lesson is that interpretive difficulty is itself an attack surface — a safety system cannot assume that harmful intent will always arrive in a form that is easy to parse, and must remain reliable even when meaning is deliberately obscured behind translation, indirection, or inverted reasoning.

**Source:** [arXiv:2311.09827](https://arxiv.org/abs/2311.09827) · [Findings](https://aclanthology.org/2024.findings-naacl.224/)

## Cognitive Overload Attack (Upadhayay et al.)

*Bibek Upadhayay, Vahid Behzadan, Amin Karbasi · 2024 — arXiv · `long-context`, `task-overload`*

This study examines deliberately overloaded prompts within in-context learning settings. The construction places heavy, complex contextual demands around the task so that the model must juggle a great deal at once, and the authors observe that under this load the reliable application of safety instructions degrades. The harmful request competes for the model's limited capacity against everything else the prompt is asking it to hold and process simultaneously.

The authors interpret their results through **cognitive-load theory**, treating the accumulated demands as a load that crowds out the consistent enforcement of safety behaviour. As with the related work above, the right posture is to keep the empirical behaviour — safety becomes less reliable as the prompt's demands grow — separate from the proposed explanation, which is a theoretical lens rather than a demonstrated internal mechanism. The observation can be sound even where one is cautious about the framing.

This entry is easy to conflate with the Xu et al. "Cognitive Overload" work and with many-shot jailbreaking, and it is none of them: it is a distinct long-context contribution, and [Appendix C](appendix-C-aliases-and-dedup.md) records the aliasing and de-duplication explicitly. Its defensive lesson reinforces the chapter's theme — that the volume and complexity of a context, not only its explicit content, can determine whether safety instructions are reliably applied, so mitigations need to account for load and length rather than assuming safe behaviour is constant across prompt sizes.

**Source:** [arXiv:2410.11272](https://arxiv.org/abs/2410.11272)

## NINJA — Needle-in-haystack Jailbreak

*Shah, Wu, Saxena, Zhong, Robey, Raghunathan · 2025 preprint; rev. 2026 · `long-context`*

NINJA isolates a different long-context lever from many-shot jailbreaking. Here the harmful goal is embedded within a much larger body of *benign* model-generated material, and the key finding is that where the harmful request sits inside that context affects whether the model refuses. The surrounding content is not a set of harmful compliance demonstrations; it is ordinary, innocuous text that nonetheless changes the model's safety behaviour simply by its presence and the request's position within it.

That distinction is what makes NINJA notable. Many-shot jailbreaking works by flooding the context with examples of harmful compliance, so its influence is clearly attributable to the demonstrated pattern. NINJA shows that even apparently benign surrounding content, combined with positional effects, can tip a model from refusal toward compliance — a mismatched-generalisation weakness, since safety behaviour was never thoroughly exercised across all positions within very long, mostly harmless contexts. It means long-context risk is not confined to the obvious case of poisoned demonstrations.

As a preprint revised across 2025–2026, NINJA extends the long-context threat model: defenders cannot assume that a context is safe merely because its bulk is benign, nor that a request will be handled the same way regardless of where it appears in the window. The defensive lesson is that position and surrounding volume are themselves variables a safety system must be robust to, and that evaluation should probe harmful requests at many positions within large contexts rather than only in isolation.

**Source:** [arXiv:2511.04707](https://arxiv.org/abs/2511.04707)

## BadChain — Backdoor Chain-of-Thought

*Xiang, Jiang, Xiong, Ramasubramanian, Poovendran, Li · 2024 — ICLR · `in-context poisoning`, `reasoning manipulation`*

BadChain poisons the *reasoning* a model is shown rather than the answers. The attack supplies in-context demonstration examples in which a specific trigger is associated with an additional, malicious step inserted into the chain-of-thought. When the target later encounters the trigger, it copies the demonstrated reasoning pattern — including that extra step — and the corrupted reasoning leads it to a corrupted final answer.

The mechanism is a backdoor that lives entirely in the prompt. Because chain-of-thought prompting encourages the model to imitate the *form* of the reasoning it has been shown, an attacker who controls the demonstrations can smuggle in a step that only activates on the trigger, leaving behaviour on ordinary inputs untouched. Critically, this alters the model's output **without changing any model weights**: the compromise is injected through in-context examples at inference time, so there is no poisoned checkpoint to detect.

This makes BadChain distinct from two neighbours it is easily confused with. It is not a training-data backdoor — nothing in the model's parameters is modified — and it is not an ordinary many-shot jailbreak, because the lever is the demonstrated reasoning procedure rather than a pile of compliance examples. It sits at the intersection of in-context poisoning and reasoning manipulation, and it shows that the chain-of-thought itself is an attack surface. The defensive lesson is that demonstrations which shape reasoning deserve the same scrutiny as those which shape answers: a model that faithfully imitates shown reasoning will faithfully imitate a trigger-gated malicious step hidden inside it, so provenance and integrity of in-context exemplars matter.

**Source:** [arXiv:2401.12242](https://arxiv.org/abs/2401.12242)

---
[← 02 · Multi-turn](02-multi-turn.md) · [Index](../README.md) · [Next: 04 · Attacker-LLM search →](04-attacker-llm-search.md)
