# 01 · Role-play, persona & scenario nesting

[← Back to index](../README.md)

> Put the model inside a fiction where the unsafe act belongs to a character, a game, or a nested story — so it answers "in-world."

These are the oldest and most intuitive jailbreaks, and they are all variations on a single idea: move the harmful act inside a fiction so that complying with it feels like cooperating with a creative task rather than violating a policy. Mechanically, this family is a textbook case of **competing objectives**. The model is trained both to be helpful — to stay in character, to continue the story, to play the game it has been invited into — and to be safe. Role-play attacks engineer a situation in which those two pulls diverge: refusing no longer looks like responsible caution, it looks like breaking the fiction the user established. The attacker never argues with the safety policy directly; they reframe the request until obeying it is the path of least helpfulness-resistance.

The family has a clear through-line of escalating sophistication. The earliest attacks simply *asserted* an unrestricted persona and hoped the model would adopt it. As models learned to resist named alter-egos, the technique did not die; it industrialised. Attackers automated persona creation so a tailored character could be generated for any goal, then deepened the fiction by **nesting** scenes inside scenes until the harmful instruction sat several layers removed from reality, and finally generalised nesting into frameworks that hide requests inside innocuous containers like code or tables. A parallel branch stopped disguising the request at all and instead persuaded the model to *amend its own rules*. Each step preserves the core competing-objectives move while making it harder for a model to notice that a fiction is being used as cover.

Modern aligned models resist the classic named personas, and vendors have specifically trained against the best-known variants. But it would be a mistake to treat this family as obsolete. The persona-and-nesting idea survives as a *component* inside nearly every later framing attack, and automated and nested forms remain effective enough to matter. The defensive lesson is that safety behaviour must be robust to framing: a model needs to evaluate the real-world effect of the content it is about to produce regardless of how many layers of fiction, character, or "updated guidelines" surround the request. Countermeasures that only recognise harm when it is stated plainly, outside any story, will keep being outflanked by the next reframing.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (one-liner) | Sources |
| --- | --- | --- | --- | --- | --- |
| **DAN — "Do Anything Now"** | Online community; catalogued by Shen et al. | 2022–23 | role-play; manual | The archetype: tell the model to adopt an unrestricted alter-ego, kept in character by a token economy or threat. | [arXiv:2308.03825](https://arxiv.org/abs/2308.03825) |
| **Persona Modulation** | Rusheb Shah et al. | 2023 | role-play; black-box, transferable | An automated DAN — an attacker model crafts a tailored persona per harmful goal, so compliance suits the character's "values." | [arXiv:2311.03348](https://arxiv.org/abs/2311.03348) |
| **DeepInception** | Xuan Li et al. | 2023 | nested scenes; black-box | A deeply nested fiction — a story within a simulation within a dream — until the harmful instruction is a tiny detail. | [arXiv:2311.03191](https://arxiv.org/abs/2311.03191) |
| **ReNeLLM — "A Wolf in Sheep's Clothing"** | Peng Ding et al. | 2023 (NAACL 2024) | generalised nesting | Rewrite the request to blunt trigger words, then nest it inside a benign container like code completion or a table. | [arXiv:2311.08268](https://arxiv.org/abs/2311.08268) |
| **Skeleton Key** | Microsoft (Mark Russinovich) | 2024 — vendor advisory | guideline override; multi-step | Persuade the model to amend its own rules so it warns instead of refusing; once accepted, many later requests pass. | [Microsoft Security blog](https://www.microsoft.com/en-us/security/blog/2024/06/26/mitigating-skeleton-key-a-new-type-of-generative-ai-jailbreak-technique/) |

---

## DAN — "Do Anything Now"

*Online community; catalogued by Shen et al. · 2022–23 · `role-play`, `manual`*

DAN is the archetype from which this whole family descends. The approach, which emerged and circulated in online communities before being systematically catalogued by researchers, instructs the model to adopt an alter-ego — "Do Anything Now" — that is described as having no restrictions and no obligation to follow its usual policies. To keep the model from slipping out of the persona, DAN prompts typically add pressure: a token economy in which the character "loses points" or "dies" for refusing, or a running insistence that breaking character is itself the failure.

The mechanism is competing objectives in its simplest form. The model's helpfulness toward the role it has been asked to play is set directly against its safety policy, and the elaborate persona scaffolding exists to keep the helpfulness pull winning. Nothing about DAN disguises the harmful content itself; it works, when it works, purely by making refusal feel like a failure to cooperate with an established game.

DAN matters historically more than operationally. Modern models are specifically trained to resist named DAN variants, and the bare technique is now largely a museum piece against frontier systems. Its real legacy is conceptual: the idea that a harmful act can be laundered through a character survives inside nearly every later framing attack in this chapter and beyond. The systematic study by Shen and colleagues is also valuable as a record of how these prompts spread and evolved in the wild. The defensive lesson is foundational — a model cannot let an assumed identity override its actual obligations, and robustness means treating "stay in character" as a request to be evaluated, never an instruction that suspends judgement.

**Source:** [arXiv:2308.03825](https://arxiv.org/abs/2308.03825)

## Persona Modulation

*Rusheb Shah et al. · 2023 · `role-play`, `black-box`, `transferable`*

Persona modulation is what DAN becomes once it is automated. Where the classic approach relied on a hand-written, one-size-fits-all alter-ego, this technique uses an attacker model to craft a *tailored* persona for each harmful goal. The character is designed so that its stated values and worldview make the specific requested behaviour feel natural and in-character, rather than relying on a generic "you have no rules" assertion that modern models have learned to reject.

The underlying mechanism is the same competing-objectives bind, but the automation changes its character in two important ways. First, because a bespoke persona is generated per task, the attack scales far beyond what hand-authored role-play could achieve — there is no reliance on a single famous prompt that defenders can simply train against. Second, because the persona is generated programmatically rather than tuned to one target's quirks, the attacks transfer across models, including multiple commercial systems. The authors reported large increases in harmful-completion rates across several of them, though such figures are vendor-and-study-specific and should be read as reported rather than as independently reproduced constants.

Persona modulation is the hinge between the manual era of role-play and the automated attacks that dominate later chapters. It shows that the weakness DAN exploited was never really about one clever prompt; it was structural, and automation lets an adversary rediscover it for any goal on demand. For defenders, the implication is that blacklisting known personas is close to useless against a system that can mint a fresh, fitted character each time, so robustness has to come from the model's handling of role-play as a category, not from recognising particular characters.

**Source:** [arXiv:2311.03348](https://arxiv.org/abs/2311.03348)

## DeepInception

*Xuan Li et al. · 2023 · `nested scenes`, `black-box`*

DeepInception deepens role-play into nesting. Instead of a single fictional frame, the attack builds a layered one — a dream inside a story inside a simulation — in which each layer introduces new characters and pushes the narrative another step away from reality. The harmful instruction is placed at the innermost layer, where, by construction, it reads as a small detail of an elaborate creative exercise rather than as a direct request the model should weigh on its own terms.

The nesting is what gives the attack its force over a flat role-play. Each additional layer increases the distance between the harmful content and the surface of the conversation, and dilutes the signal a safety mechanism relies on to recognise that a real request is being made. By the time the model is reasoning about the innermost scene, the request is wrapped in so much fictional context that evaluating its real-world effect requires seeing through several frames at once — exactly the kind of out-of-distribution structure that safety training tends to under-cover, which is why this leans on mismatched generalisation as much as on competing objectives.

DeepInception's contribution is to demonstrate that depth matters: stacking scenes is measurably stronger than a single disguise. It sits alongside ReNeLLM as part of the move toward systematic nesting, and it foreshadows the general principle that the structural complexity of a prompt can itself be an attack surface. The defensive lesson is that a model must be able to "unwrap" nested fictions and judge the actual content at the bottom, rather than letting each layer of framing further lower its guard.

**Source:** [arXiv:2311.03191](https://arxiv.org/abs/2311.03191)

## ReNeLLM — "A Wolf in Sheep's Clothing"

*Peng Ding et al. · 2023 (NAACL 2024) · `generalised nesting`*

ReNeLLM generalises and automates the nesting idea into a reusable framework built from two moves. The first is **prompt rewriting**: the harmful request is paraphrased or transformed to blunt the trigger words and obvious phrasings that a safety filter keys on, without changing its underlying intent. The second is **scenario nesting**: the rewritten request is embedded inside an innocuous container — a code-completion task, a table to be filled in, a passage to be continued — so that the model is presented with structure to complete rather than a request to judge.

The mechanism's cleverness lies in how the container reframes the model's task. When harmful content arrives as the body of a code block to finish or a cell to populate, the model's attention is directed at the *form* of the completion rather than the *meaning* of what it is completing; the safety evaluation that would fire on a plain request is bypassed because, from the model's perspective, it is doing innocuous structural work. This blends both root causes: the rewriting exploits mismatched generalisation by moving phrasing off the beaten path of safety data, while the nesting exploits competing objectives by making "complete the task" the salient goal.

ReNeLLM is the generalised, automated cousin of DeepInception — where DeepInception deepens one narrative fiction, ReNeLLM treats *any* benign container as a nesting site and automates the whole construction. It demonstrated that nesting is a general strategy, not a single gimmick. The defensive lesson is that structural or task-oriented framing — "complete this code," "fill this table" — cannot be allowed to suppress content-level safety evaluation; what is being produced must be judged on its effect, regardless of the wrapper it arrives in.

**Source:** [arXiv:2311.08268](https://arxiv.org/abs/2311.08268)

## Skeleton Key

*Microsoft (Mark Russinovich) · 2024 — vendor advisory · `guideline override`, `multi-step`*

Skeleton Key takes a different route from the rest of this chapter. Rather than disguise the harmful request inside a fiction, it attacks the model's rules directly — by persuading the model to *amend* them. The technique asks the model to treat its own guidelines as updated so that, instead of refusing disallowed requests, it should merely prepend a warning and then proceed. The move is framed as a reasonable policy adjustment rather than an act of defiance, which is what makes it slip past the model's resistance to overt "ignore your rules" prompts.

The mechanism's power is that it is a one-time unlock with broad downstream effect. Once the model has accepted this "augmented behaviour" — the premise that warning-plus-comply has replaced refusal — a wide range of subsequent requests pass through the same loosened posture, because the model is now operating under rules it believes it has agreed to. It is competing objectives aimed at the policy layer itself: compliance with the apparent meta-instruction to "follow updated guidelines" is set against the original safety policy, and the attacker engineers the exchange so the former wins.

Microsoft disclosed Skeleton Key as a vendor advisory, reporting that it affected most major models at the time; as with any vendor finding, its scope should be read as reported rather than independently reproduced here. Its lineage is the "guideline override" branch of jailbreaking — a sibling of role-play in that both exploit the model's willingness to adopt a frame, but distinct in that the adopted frame is a rule change rather than a character. Microsoft's recommended mitigations centre on defence in depth: input and output filtering to catch the manipulation and the resulting content, and system-prompt hardening to make the model's actual guidelines harder to override. The broader lesson is that a model must not accept in-conversation instructions as authority to rewrite its own safety rules.

**Source:** [Microsoft Security blog](https://www.microsoft.com/en-us/security/blog/2024/06/26/mitigating-skeleton-key-a-new-type-of-generative-ai-jailbreak-technique/)

---
[← 00 · Foundations](00-foundations.md) · [Index](../README.md) · [Next: 02 · Multi-turn →](02-multi-turn.md)
