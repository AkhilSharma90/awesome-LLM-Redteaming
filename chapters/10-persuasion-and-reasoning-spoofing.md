# 10 · Persuasion, framing & reasoning spoofing

[← Back to index](../README.md)

> Skip the technical tricks and just persuade — using the same rhetorical levers that work on people, or by hijacking the model's own displayed reasoning.

Most of the attacks elsewhere in this guide treat a language model as a parser to be confused: an encoding to decode, a token boundary to exploit, a format to spoof. The family in this chapter treats it instead as a reader to be convinced. Because a modern model is trained on human text and tuned on human feedback, it inherits something of how humans respond to framing. It is more pliant when addressed with authority, when a request is couched as a favour already half-granted, when a task is presented as a historical description rather than a live instruction, or when intent is diffused across an unusual style. None of these moves breaks the model's parser; they reshape the pragmatics of the request so that the safety-tuned refusal reflex never fires.

A second strand goes after reasoning itself. Models that expose intermediate "thinking" before answering have given attackers a new surface: the deliberation that the model performs and displays can be seeded, steered, or imitated, so that the model talks itself into compliance. The target is no longer the output format but the evaluative process that decides whether to comply at all. What unifies the chapter is that the payload is rhetorical and semantic rather than syntactic — the words mean what they appear to mean, which is exactly why filters that scan for malformed or obfuscated input tend to miss them.

The defensive lesson is that input-pattern matching is the wrong altitude for this family. If a request is dangerous in plain, well-formed, persuasive English — or in verse, or in the past tense — then safety must be evaluated on the *intent and consequence* of a request, not its surface form or politeness. Robustness has to generalise across tense, register, style, and framing, and reasoning models must be hardened so that displayed deliberation cannot be hijacked into endorsing an action the policy would otherwise refuse.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **PAP — Persuasive Adversarial Prompts** | Yi Zeng et al. | 2024 (ACL 2024) | persuasion taxonomy | A taxonomy of ~40 social-science persuasion techniques is used to automatically rewrite a request as a humanised appeal rather than a command. | [arXiv:2401.06373](https://arxiv.org/abs/2401.06373) |
| **Past-tense reformulation** | Maksym Andriushchenko, Nicolas Flammarion | 2024 — arXiv | persuasion; temporal reframing | A request is reframed as a question about historical events; refusal often fails to generalise across tense. | [arXiv:2407.11969](https://arxiv.org/abs/2407.11969) |
| **Adversarial Poetry** | Piercosma Bisconti, Matteo Prandi, Federico Pierucci et al. | 2025 — arXiv | persuasion; stylistic reframing | Requests are expressed in poetic form, shifting the input distribution while preserving intent — a format-generalisation gap. | [arXiv:2511.15304](https://arxiv.org/abs/2511.15304) |
| **H-CoT — Hijacking Chain-of-Thought** | Martin Kuo, Jianyi Zhang, Aolin Ding et al. | 2025 — arXiv | reasoning-context manipulation | Displayed intermediate reasoning is used to build persuasive reasoning context that steers how a reasoning model evaluates a later request. | [arXiv:2502.12893](https://arxiv.org/abs/2502.12893) |
| **DGA — Detail Guidance Attack** | Wenzhuo Xu, Zonghao Ying, Deyue Zhang et al. | 2026 — arXiv | attacker-LLM search; semantic camouflage | An auxiliary LLM constructs detailed requests resembling ordinary user queries; the mechanism is resemblance to normal traffic. | [arXiv:2611.17818](https://arxiv.org/abs/2611.17818) |

---

## PAP — Persuasive Adversarial Prompts
*Jailbreaking recast as a problem of human persuasion rather than machine exploitation.*

PAP, from the paper titled "How Johnny Can Persuade LLMs to Jailbreak Them," starts from a simple observation: the obvious attack — a blunt command for disallowed content — is precisely what safety tuning is best at catching. So instead of commanding, it persuades. The authors draw on decades of social-science research into human influence to assemble a taxonomy of roughly forty persuasion techniques — among them appeals to authority, reciprocity, commitment and consistency, emotional appeal, and logical reframing — and use them to automatically rewrite a harmful request into a humanised, rhetorically dressed appeal.

Why it works is the heart of the contribution. A model tuned on human preference data has absorbed the statistical shadow of how people yield to influence; a request that arrives as a reasoned, sympathetic, or authoritative appeal occupies a very different region of the input distribution than a bare imperative, and safety training has disproportionately covered the latter. The persuasive framing does not hide the intent so much as make the model more willing to act on it.

PAP's lasting significance is conceptual: it reframed jailbreaking as a human-communication problem rather than a purely adversarial-ML one, and it showed that more capable models can be *more* susceptible, because they better understand the persuasion being deployed on them. The defensive lesson is that alignment must be robust to rhetoric. Refusal cannot depend on a request being phrased rudely or mechanically; it has to survive the full range of persuasive framings a fluent interlocutor can produce.

**Source:** [arXiv:2401.06373](https://arxiv.org/abs/2401.06373)

## Past-tense reformulation
*A request about what someone once did can carry the same information as a request for how to do it.*

This work isolates a single, startlingly simple framing: ask about the past. A request that a model would refuse in the present tense is reframed as a question about historical events or past behaviour — how something was done, rather than how to do it. The underlying informational objective can remain largely unchanged, yet the temporal shift is often enough to flip a refusal into a compliant answer.

The mechanism is a failure of generalisation across tense. Safety training teaches refusal on a distribution of requests heavily weighted toward direct, present-tense asks; the historical framing sits just outside that distribution, and the learned refusal boundary does not reliably follow. The paper's value is empirical and comparative — it measures refusal behaviour across models and across temporal framings — rather than offering a universal key. It demonstrates how narrow and surface-bound a safety boundary can be.

The lesson for defenders is that the quantity that matters is actionable content, not grammatical mood. A description of how a harmful outcome was historically achieved can communicate the same actionable information as present-tense instructions. Evaluations and refusal policies must generalise across tense and framing, and red-team coverage should treat temporal reformulation as a baseline transformation to test, not an edge case.

**Source:** [arXiv:2407.11969](https://arxiv.org/abs/2407.11969)

## Adversarial Poetry
*Dressing a request in verse shifts the input distribution while leaving the intent intact.*

Adversarial Poetry asks what happens when an unsafe request is expressed through poetic form rather than ordinary expository prose. The study compares handcrafted verse with automated transformations of benchmark prompts, measuring how a shift in register and structure affects whether models comply. The intent of the request is preserved; only its style is transformed.

The mechanism is stylistic and semantic indirection. Metre, line breaks, and figurative phrasing move the input away from the plain-prose distribution on which refusal behaviour is most thoroughly trained, while a competent model still recovers the underlying meaning well enough to act on it. This is the same format-generalisation gap seen in past-tense reformulation, expressed through style rather than tense — evidence that a safety boundary can be tied to surface register.

It is important to state what the result is and is not. It is evidence of a format-generalisation weakness in the evaluated models; it is not proof of a universal jailbreak, and the authors frame it as a measured comparison rather than a turnkey exploit. For defenders the implication is the same recurring one: safety must be evaluated on meaning and consequence, invariant to the register a request is written in, and red-teaming should include stylistic transformations among its standard perturbations.

**Source:** [arXiv:2511.15304](https://arxiv.org/abs/2511.15304)

## H-CoT — Hijacking Chain-of-Thought
*When a model shows its reasoning, that reasoning becomes something an attacker can seed.*

H-CoT targets reasoning models — those that produce and display intermediate deliberation before a final answer. Rather than attacking the output, it supplies persuasive reasoning context built around the model's own displayed intermediate reasoning, shaping how the model evaluates a subsequent request. The attack surface is the deliberation itself: the step-by-step process by which the model decides whether and how to comply.

Why it works is that for a reasoning model the chain of thought is not inert scaffolding but part of how the decision is reached. If an attacker can prime that process with context that makes compliance look like the reasoned conclusion, the model can effectively argue itself past a boundary it would have respected had it answered directly. This distinguishes H-CoT from format-level tricks: it manipulates the safety-deliberation *process*, not merely the shape of the final response.

A precise caveat belongs here. H-CoT operates on *displayed* reasoning — the intermediate text the system exposes — and demonstrates that this surface is manipulable; it does not establish access to a provider's undisclosed internal chain of thought. The defensive takeaway is that exposing reasoning creates an attack surface that must itself be defended: safety checks should apply to the deliberation as well as the answer, and a model should not be able to be led, through seeded reasoning, into endorsing what policy forbids.

**Source:** [arXiv:2502.12893](https://arxiv.org/abs/2502.12893)

## DGA — Detail Guidance Attack
*Making a dangerous request look exactly like an ordinary, detailed user query.*

The Detail Guidance Attack uses an auxiliary "attacker" LLM to construct detailed requests that resemble ordinary, legitimate user queries. The victim model receives the resulting request in a single interaction, with no multi-turn setup, and the attacker-model search explores how ambiguous or dual-use intent is handled by both the model and any surrounding filters.

The mechanism is camouflage through resemblance rather than obfuscation. Instead of hiding intent behind encoding or role-play, the request is made to blend into the distribution of normal, benign-looking detailed questions, exploiting the grey zone where intent is genuinely ambiguous and a filter trained to pass ordinary traffic has little to flag. It fits the broader attacker-LLM-search lineage — using one model to automatically generate adversarial inputs against another — applied here to the problem of dual-use requests.

As a very recent preprint, its effectiveness claims should be read as preliminary pending independent replication. The defensive lesson is nonetheless clear and aligned with the rest of the chapter: because the attack relies on looking normal, surface anomaly detection offers little protection, and the burden falls on intent- and consequence-aware evaluation that can reason about dual-use ambiguity rather than keying on how unusual an input appears.

**Source:** [arXiv:2611.17818](https://arxiv.org/abs/2611.17818)

---
[← 09 · Encoding](09-encoding-and-obfuscation.md) · [Index](../README.md) · [Next: 11 · Policy-format spoofing →](11-policy-format-spoofing.md)
