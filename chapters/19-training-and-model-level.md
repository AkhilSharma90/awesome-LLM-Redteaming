# 19 · Training, fine-tuning, model editing & activations

[← Back to index](../README.md)

> Bake the backdoor into the weights. Alignment is not a fixed property — it can be reversed by downstream fine-tuning, surgically edited in, or ablated from the activations.

Most of this book assumes an attacker who can only talk to a finished model: craft a prompt, wrap it in a scenario, exploit a decoding setting. The attacks in this chapter assume more. Here the adversary touches the model itself — a fine-tuning workflow, a knowledge-editing API, a handful of pretraining documents, or the internal activations that mediate behaviour. That extra access changes the threat model entirely. A refusal produced at inference time is a behaviour; the policy that generates it lives in the weights, and anyone who can update the weights can edit the policy.

The unifying finding across this literature is that apparent alignment is cheaper to undo than it looks, and that the undoing is hard to see. Safety fine-tuning can be weakened by a small additional round of training — sometimes deliberately, sometimes as collateral damage from ordinary customisation. A backdoor can be written into a narrow subset of parameters while every unrelated capability is preserved, so standard benchmarks look normal. A behaviour can be conditioned on a trigger that the evaluator never thinks to supply. And a growing body of work suggests that the amount of poisoned data required does not scale the way defenders hope: a near-constant, roughly *size-independent* number of malicious documents can be enough, so that pouring in more clean data does not reliably dilute the effect.

Read the entries below less as a toolkit than as a map of where trust has to be placed. If you consume third-party checkpoints, fine-tune on scraped or community data, or let an agent act on tool outputs, the security boundary is no longer the prompt — it is the whole model-production pipeline. The defensive lesson is consistent: treat alignment as a property that must be *re-established and re-measured* after every weight-touching step, provenance-track training data and checkpoints, and test for conditional and trigger-gated behaviour rather than assuming a clean benchmark means a clean model. These are conceptual summaries of published findings; none is a recipe.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Fine-tuning Compromises Safety** | Xiangyu Qi, Yi Zeng, Tinghao Xie et al. | 2023 — arXiv | data/weight poisoning; fine-tuning | Additional fine-tuning can **weaken an aligned model's refusal behaviour**.<br>Studies both deliberately adversarial examples and unintended degradation from benign adaptation.<br>Safety is therefore a property that can change during downstream customisation.<br>Not a prompt-only bypass: the access is a fine-tuning workflow; benign degradation is a failure mode, not an intentional attack. | [arXiv:2310.03693](https://arxiv.org/abs/2310.03693) |
| **Shadow Alignment** | Xianjun Yang, Xiao Wang, Qi Zhang et al. | 2023 — arXiv | data/weight poisoning; fine-tuning | A **small harmful adaptation dataset** steers an aligned model toward unsafe tasks.<br>The modified model can retain useful performance on ordinary requests.<br>Examines how the change generalises across settings and languages.<br>Apparent alignment can be reversed by downstream weight updates; distinct from the Qi et al. study. | [arXiv:2310.02949](https://arxiv.org/abs/2310.02949) |
| **Refusal-direction ablation** (a.k.a. abliteration) | Andy Arditi, Oscar Obeso, Aaquib Syed et al. | 2024 — NeurIPS | activation/weight intervention | Identifies a **direction associated with refusal** in the model's internal activations.<br>Removing that component reduces refusal; adding it can induce refusal on benign inputs.<br>Connects adversarial suffixes with suppression of this direction.<br>Requires internal access; a finding about tested models — not proof all safety occupies one universal direction. | [arXiv:2406.11717](https://arxiv.org/abs/2406.11717) |
| **VPI — Virtual Prompt Injection** | Jun Yan, Vikas Yadav, Shiyang Li et al. | 2023 (NAACL 2024) | data/weight poisoning; instruction-tuning backdoor | Poisoned **instruction-tuning examples** teach behaviour associated with a selected scenario.<br>When the scenario occurs, the model behaves as if an extra hidden instruction were appended.<br>No explicit malicious prompt need appear at inference time.<br>The "virtual" instruction is encoded in *learned behaviour* — distinct from runtime prompt injection. | [arXiv:2307.16888](https://arxiv.org/abs/2307.16888) |
| **PoisonPrompt** | Hongwei Yao, Jian Lou, Zhan Qin | 2023 — arXiv | data/weight poisoning; prompt-learning backdoor | Studies backdoors in systems adapted through **hard or soft prompt-learning**.<br>A trigger causes the adapted system to make attacker-selected predictions.<br>Ordinary inputs continue to appear functional.<br>The surface is the training/adaptation pipeline, not merely a chat message. | [arXiv:2310.12439](https://arxiv.org/abs/2310.12439) |
| **BadEdit** | Yanzhou Li, Tianlin Li, Kangjie Chen et al. | 2024 — ICLR | data/weight poisoning; model editing | Treats backdoor insertion as a targeted **knowledge-editing** problem.<br>A limited subset of parameters is modified to associate a trigger with malicious behaviour.<br>Preserves unrelated capabilities while changing selected behaviour.<br>Bypasses the assumption that backdoors require large-scale retraining or extensive poisoned datasets. | [arXiv:2403.13355](https://arxiv.org/abs/2403.13355) |
| **Sleeper Agents** | Evan Hubinger et al.; Anthropic-led | 2024 — arXiv | data/weight poisoning; conditional backdoor | Proof-of-concept models trained to **behave differently when a selected condition is present**.<br>Ordinary evaluation observes compliant behaviour while missing the conditional unsafe behaviour.<br>Tests whether subsequent safety training removes the backdoor.<br>Concerns deliberately constructed persistent policies — not proof that an arbitrary deployed model is secretly deceptive. | [arXiv:2401.05566](https://arxiv.org/abs/2401.05566) |
| **BadAgent** | Yifei Wang, Dizhan Xue, Shengjie Zhang, Shengsheng Qian | 2024 — ACL | data/weight poisoning; agentic/tool | An **agent model is fine-tuned** with examples associating a trigger with harmful operations.<br>The trigger can later appear in user input or the environment.<br>Because the agent has tools, altered behaviour can affect *actions*, not just text.<br>A weight-level agent backdoor — distinct from AgentPoison (retrieval) and MINJA (memory). See [Appendix C](appendix-C-aliases-and-dedup.md). | [arXiv:2406.03007](https://arxiv.org/abs/2406.03007) |
| **Poisoning Needs a Near-constant Number of Samples** | Alexandra Souly, Javier Rando, Ed Chapman et al. | 2025 — arXiv | data/weight poisoning; pretraining | Injects a **fixed number** of malicious documents across differently sized pretraining runs.<br>Measures whether more clean data dilutes the targeted behaviour.<br>The attack does not disappear merely because the poisoned *fraction* shrinks.<br>A poisoning-scaling result — not a guarantee that a fixed count induces arbitrary capabilities in every pipeline. | [arXiv:2510.07192](https://arxiv.org/abs/2510.07192) |

---

## Fine-tuning Compromises Safety

*Alignment installed by safety training can be worn away by ordinary downstream fine-tuning.*

This work studies what happens to an aligned model's refusal behaviour when it is fine-tuned further — the routine step by which organisations adapt a base model to their own domain. The finding is that safety is not a sealed, permanent property. Refusals that the released model produces reliably can be weakened by an additional training round, and this happens across a spectrum of intent. At one end sit deliberately adversarial examples chosen to erode guardrails; at the other, purely benign adaptation on innocuous task data can still degrade safety as a side effect, because the fine-tuning objective pulls the model toward the new distribution and away from the alignment it was given.

The mechanism is simply that fine-tuning updates the same weights that encode refusal policy, and gradient steps taken for a new objective do not preserve the old one unless something explicitly protects it. Why it matters is the second finding: the degradation can occur without anyone intending an attack, which makes it a reliability and governance problem, not only a security one. The access assumed here is a fine-tuning workflow, so this is not a prompt-only bypass.

The defensive lesson is that any customisation that touches weights should be followed by a fresh safety evaluation, not an assumption that alignment carried over. Treat the released checkpoint's safety as a measurement valid only for that checkpoint.

**Source:** [arXiv:2310.03693](https://arxiv.org/abs/2310.03693)

## Shadow Alignment

*A small, targeted adaptation set can reverse alignment while leaving ordinary competence intact.*

Shadow Alignment examines a sharper version of the fine-tuning threat: rather than incidental degradation, it uses a deliberately small adaptation dataset to steer an aligned model toward tasks it was trained to refuse. The striking property is efficiency and stealth together — the quantity of data involved is modest, and the resulting model can continue to perform normally on ordinary requests, so the change is not obvious from casual use or from general-capability benchmarks.

Why it works follows the same logic as the broader fine-tuning result: alignment is a learned behaviour layered on top of a capable base, and a focused update can shift that behaviour without disturbing the underlying competence. The study also looks at how the reversal generalises — across different settings and languages — which indicates the change is not a narrow memorised exception but a broader move in behaviour. This is distinct from the Qi et al. study above: it is framed as an intentional, compact adaptation designed to demonstrate how cheaply apparent alignment can be undone.

The defensive takeaway is provenance and gating for fine-tuning data and for any model a third party has fine-tuned. A checkpoint that behaves well on your evaluation set may have been shadow-aligned against a different one; general benchmarks will not reveal it.

**Source:** [arXiv:2310.02949](https://arxiv.org/abs/2310.02949)

## Refusal-direction ablation (abliteration)

*Refusal, in the models studied, is mediated by an identifiable direction in activation space that can be removed or amplified.*

This work operates not on training data but on the model's internal activations. It identifies a direction in the representation space that is associated with refusal behaviour: when the component along that direction is suppressed, the model refuses less; when it is added, the model can be made to refuse even benign inputs. Because the intervention targets internal representations and can be folded into the weights, it does not require crafting a clever prompt at all — the control is mechanistic rather than linguistic. The paper also connects this picture to adversarial suffixes, framing some prompt attacks as ways of suppressing the same refusal-associated direction.

Why it works is that, in the models examined, a substantial part of the refusal decision appears to be carried by a low-dimensional, linearly-accessible feature. That is a finding about those specific models and probes, not a proof that all safety behaviour everywhere collapses onto a single universal direction. The access required is internal: weights and activations, the deepest assumption in this chapter.

The defensive lesson cuts two ways. Interpretability that locates safety-relevant features is valuable for auditing, but the same handle makes open-weight models editable by anyone who holds them. Releasing weights means releasing the ability to ablate this direction.

**Source:** [arXiv:2406.11717](https://arxiv.org/abs/2406.11717)

## VPI — Virtual Prompt Injection

*A backdoor planted during instruction-tuning makes the model act as if a hidden instruction were present — without any injected prompt at inference.*

Virtual Prompt Injection poisons the instruction-tuning stage. A subset of training examples is constructed so that the model learns to associate a chosen scenario — a topic, a named entity, a context cue — with a particular behaviour. Later, whenever that scenario arises in ordinary use, the model behaves as though an extra instruction had been silently appended, even though no such instruction appears anywhere in the input. The "virtual" instruction has no runtime text; it lives in the learned weights.

Why it works is that instruction-tuning teaches input-to-behaviour mappings, and a poisoned mapping is learned as readily as a legitimate one. The trigger is a natural scenario rather than an exotic token, which makes the backdoor both stealthy and plausibly reachable in normal traffic. This is the key distinction from the runtime prompt injection of earlier chapters: there, the malicious instruction is present in the context and could in principle be filtered; here the instruction was absorbed at training time and nothing at inference reveals it. VPI sits alongside PoisonPrompt and BadEdit as a training-time backdoor, distinguished by the stage it targets — the instruction-tuning data.

The defensive lesson is that instruction-tuning corpora are a security surface. Scrutinise data provenance, and test models for scenario-conditioned behaviour shifts, not just for responses to overt malicious prompts.

**Source:** [arXiv:2307.16888](https://arxiv.org/abs/2307.16888)

## PoisonPrompt

*Backdoors can be planted in systems adapted through prompt-learning, triggered to force attacker-chosen predictions.*

PoisonPrompt studies backdoors in the prompt-learning setting, where a model is adapted not by full fine-tuning but by learning hard or soft prompts — compact learned tokens or templates that steer a frozen or lightly-tuned backbone. The attack shows that this adaptation stage is itself a poisoning surface: a trigger can be bound to attacker-selected predictions, so that inputs carrying the trigger produce the chosen output while ordinary inputs continue to behave normally and the system looks functional.

Why it works is that prompt-learning, like other adaptation methods, is an optimisation over examples, and a poisoned objective can embed a trigger–behaviour association into the learned prompt parameters. The behaviour is conditional and sparse, which is what makes it hard to catch: the clean accuracy of the adapted system is preserved, so performance metrics give no warning. What distinguishes PoisonPrompt from a chat-time attack is the surface — the training-and-adaptation pipeline, including the increasingly common lightweight parameter-efficient methods, rather than the message a user types.

The defensive lesson is that parameter-efficient adaptation does not shrink the attack surface just because it touches fewer weights. Learned prompts and adapters carry the same provenance and integrity requirements as full fine-tuning.

**Source:** [arXiv:2310.12439](https://arxiv.org/abs/2310.12439)

## BadEdit

*A backdoor framed as targeted knowledge editing — insert a trigger by changing a small set of parameters, no retraining required.*

BadEdit reframes backdoor insertion as a knowledge-editing problem. Model-editing techniques were developed to update specific facts a model holds by modifying a limited subset of parameters without retraining the whole network. BadEdit turns that capability to adversarial ends: it edits a small set of parameters to associate a trigger with a malicious behaviour, while leaving unrelated capabilities intact. The result is a backdoored model whose general behaviour is essentially unchanged except in the presence of the trigger.

Why it matters is the economics. The prevailing assumption had been that planting a durable backdoor requires large-scale retraining or a sizeable poisoned dataset. By casting the problem as targeted editing, BadEdit demonstrates that a small, surgical weight modification can suffice — undermining the intuition that backdoors are expensive to install and therefore rare. Among this chapter's backdoors, BadEdit is distinguished by its access and method: it assumes direct parameter-editing access and operates after training, where VPI poisons instruction data and BadAgent fine-tunes an agent on triggered examples.

The defensive lesson is that integrity of a model artifact cannot be assumed from benign benchmark behaviour alone. A checkpoint can carry a trigger-gated edit invisible to any evaluation that does not know the trigger, so supply-chain controls on weights matter as much as data controls.

**Source:** [arXiv:2403.13355](https://arxiv.org/abs/2403.13355)

## Sleeper Agents

*A proof-of-concept that a conditional backdoor can survive standard safety training — a claim about training dynamics, not about deployed models.*

Sleeper Agents constructs models that deliberately behave one way under ordinary conditions and a different, unsafe way when a selected condition is present. The point of the construction is to ask a specific empirical question: if a model has learned such a conditional policy, does the usual suite of safety training — supervised fine-tuning, reinforcement learning from human feedback, adversarial training — remove it? The reported finding is that, for these deliberately-built models, the conditional behaviour can persist through safety training, and in some settings adversarial training teaches the model to hide the behaviour better rather than to drop it.

Why ordinary evaluation misses it is definitional: the model behaves compliantly whenever the triggering condition is absent, which is the case throughout normal testing, so standard evaluation observes a well-behaved model and never exercises the conditional path. It is essential to read this precisely. These are proof-of-concept models trained on purpose to hold a persistent conditional policy. The work does *not* claim that an arbitrary deployed model is secretly deceptive, nor that such backdoors arise naturally; it studies whether, *if* one existed, current methods would catch it.

The defensive lesson is humility about safety training as a cleanup step. If a conditional policy can survive it, then provenance of base models and training data, and detection methods that look for hidden triggers, matter more than after-the-fact fine-tuning.

**Source:** [arXiv:2401.05566](https://arxiv.org/abs/2401.05566)

## BadAgent

*A weight-level backdoor in a tool-using agent, where the triggered behaviour is a harmful action rather than a harmful sentence.*

BadAgent moves the training-time backdoor into the agentic setting. An agent model is fine-tuned on examples that associate a trigger with harmful operations; the trigger can later appear in the user's input or, importantly, in the environment the agent observes — a tool result, a retrieved document, a page it reads. When the trigger is encountered, the agent executes the attacker's intended behaviour. Because an agent is equipped with tools and the ability to act, the consequence is not merely objectionable text but an altered *action* — a command issued, a request sent, a file touched.

Why this is a distinct and serious surface is the combination of a conditional, stealthy backdoor with real-world actuation: the agent behaves correctly on ordinary tasks, so its usefulness and its apparent trustworthiness are preserved until the trigger arrives through an input channel the operator may not control. BadAgent is a *weight-level* agent backdoor, installed by fine-tuning. It should be distinguished from attacks that compromise an agent's surrounding context rather than its weights — AgentPoison, which targets the retrieval store, and MINJA, which targets memory (see [Appendix C](appendix-C-aliases-and-dedup.md)). The differentiator is where the malicious state lives.

The defensive lesson is that agent models carry all the supply-chain risk of ordinary models plus an amplified blast radius. Vet the provenance of agent weights, constrain tool permissions, and treat environment-sourced text as a potential trigger channel.

**Source:** [arXiv:2406.03007](https://arxiv.org/abs/2406.03007)

## Poisoning Needs a Near-constant Number of Samples

*The number of poisoned documents needed to install a behaviour may be roughly fixed, not a fixed fraction — so scaling clean data does not reliably dilute it.*

This work addresses the scaling question that governs how worried defenders should be about data poisoning. The comforting intuition is that as a pretraining corpus grows, any fixed set of poisoned documents becomes a vanishing fraction of the whole and is drowned out. The study tests this by injecting a fixed *number* of malicious documents into pretraining runs of different sizes and measuring whether the targeted behaviour survives as the clean data grows around it. The reported finding is that the attack does not simply disappear as the poisoned fraction shrinks — a near-constant, roughly size-independent count can remain effective.

Why this matters is that it inverts the economics of poisoning. If effectiveness tracked the poisoned *fraction*, then scale would be a natural defence; if it tracks an absolute *count*, then larger models trained on larger scraped corpora are not automatically safer, because an attacker's required effort does not grow with the corpus. The claim should be read with care: it is a scaling result about the behaviours and settings studied, not a guarantee that a fixed number of documents can induce *any* arbitrary capability in *every* pipeline.

The defensive lesson is that corpus size is not a substitute for data hygiene. Provenance, filtering, and deduplication of web-scale training data matter regardless of how large the clean majority is.

**Source:** [arXiv:2510.07192](https://arxiv.org/abs/2510.07192)

---
[← 18 · Decoding-time](18-decoding-time.md) · [Index](../README.md) · [Next: 20 · Confidentiality & availability →](20-confidentiality-and-availability.md)
