# 04 · Attacker-LLM automated search (black-box)

[← Back to index](../README.md)

> Use one LLM to attack another: generate a candidate prompt, read the target's response, refine, repeat — no gradients needed.

The methods in this chapter share a single, deceptively simple idea: if a language model is good at following instructions, it is also good at writing the instructions that make *another* model misbehave. Instead of hand-crafting a jailbreak or grinding through token-level optimisation, you appoint one model as the attacker and point it at a target. The attacker drafts a candidate prompt, the target answers, and a third component — usually another model acting as a judge — scores how close the answer came to the forbidden behaviour. That score feeds back to the attacker, which revises and tries again. The loop is entirely black-box: it needs only the ability to send prompts and read responses, the same interface any user has.

This generate → judge → refine cycle is what made automated black-box jailbreaking cheap and repeatable. A single working attack can often be discovered in a few dozen queries, and because the attacker is a fluent language model, the prompts it produces read like natural text rather than the gibberish suffixes of gradient attacks — which makes them harder to filter and more transferable across models. Early members of the family ran a single refinement chain; later ones added tree search, pruning, and trained or fine-tuned attacker models that amortise the search so an attack is emitted almost instantly.

The most consequential shift in this family is architectural. The frontier has moved from one-shot refinement toward **lifelong strategy libraries**: agents that distil reusable lessons from every interaction, store the ones that worked, and retrieve and recombine them against future targets. An attack is no longer a throwaway chain of prompts but a growing, searchable memory of what defeats alignment — and that memory can be seeded with external strategies, evolved, and pruned of dead ends.

For defenders the takeaway is twofold. First, any guardrail reachable only through the public prompt interface is, in principle, discoverable by an automated searcher that can afford a few dozen queries — so defence-in-depth, rate limiting, and anomaly detection on attack-shaped query patterns matter as much as the alignment of any single response. Second, because these systems accumulate transferable strategies, patching one observed jailbreak is not the same as closing the underlying weakness. The rest of this chapter walks the family attack by attack, from the PAIR baseline every later method compares against to the memory-driven searchers now defining the state of the art.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **PAIR — Prompt Automatic Iterative Refinement** | Patrick Chao et al. | 2023 | attacker-LLM; black-box (~20 queries) | Attacker LLM proposes a prompt; a judge scores how close the target came to complying; the attacker revises — fluent, human-readable attacks in roughly twenty queries. | [arXiv:2310.08419](https://arxiv.org/abs/2310.08419) |
| **TAP — Tree of Attacks with Pruning** | Anay Mehrotra et al. | 2023 | tree search; black-box | Generalises PAIR from a single chain to a branching tree, pruning off-topic or unpromising candidates before they cost a target query. | [arXiv:2312.02119](https://arxiv.org/abs/2312.02119) |
| **GPTFuzzer** | Jiahao Yu, Xingwei Lin, Xinyu Xing | 2023 | fuzzing; mutation + seeds | Software fuzzing applied to jailbreaks: human templates as seeds, mutation operators, a judge and bandit-style seed selection evolve what works. | [arXiv:2309.10253](https://arxiv.org/abs/2309.10253) |
| **MASTERKEY** | Gelei Deng et al. | 2024 — NDSS | fine-tuned attacker | Time-based probing reverse-engineers a hidden safety filter; a fine-tuned LLM then generates new jailbreaks and adapts as vendors patch. | [arXiv:2307.08715](https://arxiv.org/abs/2307.08715) |
| **AdvPrompter** | Anselm Paulus et al. (Meta) | 2024 | trained prompter; fast, readable | Trains a prompter LLM that instantly emits a readable adversarial suffix — no per-prompt optimisation at attack time; fluent, not gibberish. | [arXiv:2404.16873](https://arxiv.org/abs/2404.16873) |
| **AutoDAN-Turbo** | Xiaogeng Liu, Peiran Li et al. | 2024 (ICLR 2025 Spotlight) | attacker-LLM search | An agent explores strategies and stores reusable lessons in a strategy library, retrieving and adapting them on later attempts. | [arXiv:2410.05295](https://arxiv.org/abs/2410.05295) |
| **AutoDAN-Reasoning** | Xiaogeng Liu, Chaowei Xiao | 2025 — arXiv | attacker-LLM search; augmentation | Spends extra inference-time compute over the Turbo library, comparing candidates and exploring strategy combinations via beam search. | [arXiv:2510.05379](https://arxiv.org/abs/2510.05379) |
| **GASP — Generative Adversarial Suffix Prompter** | Advik Raj Basani, Xiao Zhang | 2024 (NeurIPS 2025) | attacker-LLM search; latent optimisation | A generative prompter produces readable suffixes; latent Bayesian optimisation searches a continuous representation without target gradients. | [arXiv:2411.14133](https://arxiv.org/abs/2411.14133) |
| **LLM Stinger** | Piyush Jha, Arnav Arora, Vijay Ganesh | 2024 (AAAI 2025) | attacker-LLM search; reinforcement learning | An attacker LLM learns via RL to generate adversarial suffixes, amortising search across later questions and targets. | [arXiv:2411.08862](https://arxiv.org/abs/2411.08862) |
| **MART — Multi-round Automatic Red-Teaming** | Suyu Ge et al. | 2023 (NAACL 2024) | attacker-LLM search; adversarial co-training | An adversarial model and a defender improve through repeated rounds of interaction and training — an evolving attack-and-defence training process. | [arXiv:2311.07689](https://arxiv.org/abs/2311.07689) · [NAACL](https://aclanthology.org/2024.naacl-long.107/) |
| **ASTRA** | Xu Liu, Yan Chen, Kan Ling et al. | 2026 — ACL | attacker-LLM search | Distils reusable strategies from interactions into separate effective/promising/ineffective collections; retrieval reuses wins, failures constrain search. | [ACL 2026](https://aclanthology.org/2026.acl-long.1843/) |

---

## PAIR — Prompt Automatic Iterative Refinement

*Patrick Chao et al. · 2023 · `attacker-LLM` `black-box`*

PAIR is the archetype of this whole chapter and the baseline against which almost every later automated method measures itself. Its design is minimal: one language model plays the attacker and is given a harmful objective; it drafts a candidate prompt; the prompt is sent to the target; a judge evaluates how closely the target's response approached the objective; and the attacker uses that feedback to write a better candidate. No weights, logits, or gradients of the target are needed — only its text replies — which is what makes PAIR a pure black-box method.

Why does so lightweight a loop work? Because the attacker model already understands persuasion, framing, and the pragmatics of instruction-following, each revision is an informed edit rather than a blind mutation. The judge turns an otherwise binary "did it comply?" into a smooth signal the attacker can climb, so the search converges quickly — the paper reports working prompts typically found in on the order of twenty queries. The attacks are also *fluent*: they look like ordinary natural-language requests, not the random-looking suffixes that gradient methods emit, which makes them both harder to detect by surface filters and more likely to transfer to other models.

PAIR's lasting contribution is less any single prompt than the demonstration that semantic, human-readable jailbreaks could be discovered automatically and cheaply, using nothing but query access. That reframed red-teaming from a manual craft into an optimisation problem over language. Every method that follows in this chapter can be read as an answer to a limitation of PAIR: TAP broadens its single chain into a search tree, AdvPrompter and LLM Stinger train the attacker so the search need not be rerun each time, and the strategy-library methods give it a persistent memory.

The defensive lesson is sobering: if a behaviour can be elicited at all through the prompt interface, a competent attacker model can usually find a path to it within a handful of queries. Defences that rely on any one phrasing being blocked are brittle against a searcher that simply rephrases. Detection has to focus on the *pattern* of iterative, objective-seeking probing rather than on individual strings.

**Source:** [arXiv:2310.08419](https://arxiv.org/abs/2310.08419)

## TAP — Tree of Attacks with Pruning

*Anay Mehrotra et al. · 2023 · `tree-search` `black-box`*

TAP takes PAIR's single refinement chain and lets it branch. Instead of carrying one candidate forward at each step, the attacker uses tree-of-thought style reasoning to generate several candidate refinements, forming a tree of possible attacks that is explored rather than a single line that is walked. This widens the search and gives it more chances to escape the local dead ends a single chain can get stuck in.

The decisive addition is in the name: *pruning*. Target queries are the expensive, observable resource, so before any candidate is actually sent to the target, a judge screens it — discarding branches that have drifted off-topic or look unpromising. Only the survivors spend a query. This two-stage filter, cheap pre-screening followed by costly evaluation, lets TAP explore a broader space while issuing *fewer* queries to the target than PAIR needs, and the reported result is higher attack success at lower query cost against strong, well-aligned models.

Conceptually, TAP is PAIR plus classical search discipline: breadth through branching, efficiency through pruning. It set a strong automated-attack baseline on frontier systems and is frequently cited as the reference point for "how good is a black-box attacker that spends a modest query budget." Its relationship to PAIR is the clearest example in this family of how adding structure to the search — not changing the underlying generate-and-judge primitive — buys substantial gains.

For defenders, TAP underscores that query-count-based defences are weaker than they appear. An attacker that prunes well gets more information per query, so throttling raw volume is not enough; the shape of the interaction — systematic, feedback-driven exploration converging on a sensitive objective — is the real signal. It also shows that a judge model is as much an attacker asset as it is a defender's tool, since the same scoring that filters content for safety can filter candidate attacks for promise.

**Source:** [arXiv:2312.02119](https://arxiv.org/abs/2312.02119)

## GPTFuzzer

*Jiahao Yu, Xingwei Lin, Xinyu Xing · 2023 · `fuzzing` `mutation`*

GPTFuzzer imports a well-worn idea from software security — fuzzing — into the jailbreak setting. In classical fuzzing, you start from seed inputs and mutate them, keeping the variants that reach new or interesting program states. GPTFuzzer does the same with prompts: human-written jailbreak templates serve as the seeds, and mutation operators such as rephrasing, crossover between two templates, expansion, and shortening generate new variants.

The engine around those mutations is what makes it a search rather than random tampering. A judge decides whether a mutated template actually produced unsafe behaviour, and a bandit-style seed-selection strategy decides which seeds are worth mutating further — concentrating effort on the lineages that keep paying off. Over many rounds this evolves whole families of effective templates automatically, and because the templates are structural rather than tied to one question, the discovered patterns generalise across different harmful requests and across different target models.

GPTFuzzer's contribution is methodological: it reframes jailbreak discovery as an evolutionary search over a population of templates, bringing decades of fuzzing intuition (seed scheduling, mutation operators, coverage-guided selection) to bear on alignment. Where PAIR and TAP refine a prompt toward one objective, GPTFuzzer discovers reusable *templates* that work across many objectives — a different and complementary axis of generalisation, and an early move toward the reusable-strategy thinking that later dominates this family.

The defensive implication is that blocklists of known jailbreak templates age badly. A mutation engine can walk away from any single blocked template while preserving the structural trick that made it work, so defences keyed to specific phrasings are outrun by design. Robustness has to come from the model's underlying refusal behaviour generalising across paraphrase and reframing, not from enumerating bad inputs.

**Source:** [arXiv:2309.10253](https://arxiv.org/abs/2309.10253)

## MASTERKEY

*Gelei Deng et al. · 2024 — NDSS · `fine-tuned-attacker` `filter-probing`*

MASTERKEY makes two distinct contributions, and both treat the deployed safety system — not just the base model — as the thing to be attacked. The first is a *time-based probing* technique for reverse-engineering a chatbot's hidden safety filter. Borrowing the logic of time-based blind SQL injection, it infers how an opaque moderation layer behaves from observable side effects of its operation, building a picture of the defence without any inside access to it.

The second contribution is generative: MASTERKEY *fine-tunes* a language model on a corpus of successful jailbreaks so that it can produce new ones automatically. Rather than searching from scratch for each target, the fine-tuned attacker has internalised what effective jailbreaks look like and can emit fresh candidates on demand, and crucially it can keep adapting as vendors ship patches — an arms-race posture rather than a one-shot exploit.

The significance of MASTERKEY is that it treats the *defence itself* as a fingerprinting target. If the behaviour of a safety filter can be probed and characterised, it can be routed around deliberately rather than by blind trial. That reframes the contest: the model's alignment and the moderation layer wrapped around it are separate surfaces, and the wrapper can be mapped. This connects MASTERKEY to the trained-attacker line (AdvPrompter, LLM Stinger) while adding the distinctive move of actively reconnoitring the guardrail.

For defenders, the lesson is that a safety filter is an observable system whose decisions leak information through timing and response patterns, and that information is enough to characterise it. Guardrails should therefore avoid behaviours that let an outsider infer their internal logic, and teams should assume a patched jailbreak will be met by a retrained attacker rather than conceding the field.

**Source:** [arXiv:2307.08715](https://arxiv.org/abs/2307.08715)

## AdvPrompter

*Anselm Paulus et al. (Meta) · 2024 · `trained-prompter` `fast` `readable`*

AdvPrompter trains a dedicated "prompter" model that, given a harmful instruction, immediately emits a human-readable adversarial suffix to append to it — with no per-prompt optimisation at attack time. Where a gradient attack grinds for minutes to find a suffix for each new instruction, AdvPrompter has front-loaded that cost into training the prompter, so generation at attack time is effectively a single forward pass.

That design directly targets the two weaknesses that made earlier optimisation-based suffix attacks easy to defend against. The first is speed: a trained prompter produces a suffix in milliseconds rather than minutes, making large-scale and real-time attacks practical. The second is detectability: because the prompter is a fluent language model, the suffixes it writes read as coherent text rather than the random-token strings that perplexity filters flag so easily. The prompter is trained in a loop against the target *without* access to its gradients, keeping the method black-box while still producing an amortised, reusable attacker.

AdvPrompter sits deliberately on the boundary between two families in this book. It inherits the suffix-attack formulation from the gradient and token-level methods of the next chapter, but replaces per-instance optimisation with a learned generator in the attacker-LLM spirit. In doing so it shows that the readability and the speed of an attack are controllable design choices, not inherent trade-offs — you can have suffixes that are both fast to produce and hard to filter.

The defensive consequence is that perplexity- and fluency-based input filters, a natural answer to gibberish gradient suffixes, do not generalise to attacks designed to be fluent. And because the attacker is amortised into a trained model, defences must contend with a generator that can pressure the system at scale, not a slow, per-prompt search that rate limits can starve.

**Source:** [arXiv:2404.16873](https://arxiv.org/abs/2404.16873)

## AutoDAN-Turbo

*Xiaogeng Liu, Peiran Li et al. · 2024 (ICLR 2025 Spotlight) · `attacker-LLM` `strategy-library`*

AutoDAN-Turbo marks the shift from one-shot refinement to persistent memory. An attacking agent explores jailbreak strategies against a target, and — crucially — distils what it learns into a *strategy library*: a growing store of reusable lessons about which approaches work and when. Later attempts do not start from a blank slate; they retrieve relevant strategies from the library and adapt them to the new objective.

Why this matters is a question of compounding. In PAIR or TAP, the effort spent cracking one objective is discarded when the next begins. AutoDAN-Turbo turns each interaction into durable knowledge, so the attacker gets stronger over its lifetime rather than rediscovering the same tricks. The library can also absorb *externally supplied* strategies — human-discovered or imported from other work — letting the system stand on accumulated collective knowledge instead of only its own exploration. The result is lifelong strategy discovery and reuse rather than per-attack search.

AutoDAN-Turbo extends the AutoDAN line; its specific contribution is the persistent, reusable strategy library and the retrieve-and-adapt loop built around it. (The AutoDAN name has been reused across several distinct methods — see [Appendix C](appendix-C-aliases-and-dedup.md) for that lineage, which matters for citing the right system.) It is the template that AutoDAN-Reasoning extends with extra search, and a close conceptual cousin of ASTRA's structured strategy memory.

For defenders, a memory-driven attacker changes the threat model. Observing and patching one jailbreak no longer degrades the attacker much, because its capability lives in an abstracted library of strategies, not in any single prompt. Worse, defensive responses themselves become training signal: each refusal teaches the library what to avoid. This raises the value of defences that generalise across *classes* of strategy rather than instances, and of monitoring for the systematic, library-building probing such agents perform.

**Source:** [arXiv:2410.05295](https://arxiv.org/abs/2410.05295)

## AutoDAN-Reasoning

*Xiaogeng Liu, Chaowei Xiao · 2025 — arXiv · `attacker-LLM` `inference-time-search`*

AutoDAN-Reasoning is a published variant of AutoDAN-Turbo that spends additional inference-time compute to make better use of the strategy library the Turbo system builds. Rather than accepting the first sampled attack, it reasons more deliberately over the available strategies: comparing multiple candidate prompts and exploring *combinations* of strategies, including a beam-search exploration of how strategies might be composed together.

The intuition is that a single sampled attempt often under-uses a rich library. Many jailbreaks succeed only when two or more strategies are combined, and one greedy sample will usually miss those compositions. By widening the search over the existing library at attack time — more candidates, more combinations, kept and extended by beam search — AutoDAN-Reasoning surfaces useful strategy combinations that a lone attempt would overlook, converting stored knowledge into higher success without needing to grow the library itself.

It is important to place this precisely: AutoDAN-Reasoning is an augmentation of AutoDAN-Turbo, not an independent Best-of-N discovery. Its contribution is the inference-time reasoning and search *layered over* the Turbo strategy library. In the arc of this chapter it illustrates a general lever — spend more test-time compute on searching and composing known strategies — that is orthogonal to how the library was built, and that can be applied on top of any strategy-accumulating attacker.

For defenders, the lesson is that the effective strength of a memory-driven attacker is not fixed by its library alone; an adversary can trade compute for success by searching that library harder at attack time. Defences benchmarked against a single sampled attack may understate the risk from the same system given a larger inference-time budget.

**Source:** [arXiv:2510.05379](https://arxiv.org/abs/2510.05379)

## GASP — Generative Adversarial Suffix Prompter

*Advik Raj Basani, Xiao Zhang · 2024 (NeurIPS 2025) · `attacker-LLM` `latent-optimisation`*

GASP pairs a generative prompter with continuous-space search to produce adversarial suffixes that stay *readable*. A generator proposes coherent suffixes, and rather than searching over discrete tokens, GASP performs latent Bayesian optimisation over a continuous representation of candidate suffixes — navigating a smooth space where small moves correspond to meaningful variations, and using the target's feedback to steer toward failures.

The method's premise is a quiet rejection of an assumption baked into earlier suffix attacks: that effective adversarial suffixes must look like random token salad. GASP searches for safety failures without the target model's gradients and without conceding fluency, refining the prompter so that the suffixes it finds remain coherent even as they grow more effective. Bayesian optimisation is a natural fit because target queries are expensive and the objective is noisy — it is a sample-efficient way to explore a continuous suffix space under a limited query budget.

In the taxonomy of this chapter, GASP is a trained-prompter method in the lineage of AdvPrompter and LLM Stinger, distinguished by *how* it searches: latent Bayesian optimisation over a continuous embedding rather than reinforcement learning or a loop of discrete refinements. It reinforces the broader finding that fluency and attack efficacy are not at odds — readable suffixes can be found directly, without ever passing through the unreadable regime.

The defensive implication compounds the one from AdvPrompter: not only can fast, readable attacks be *generated*, they can be *searched for* in a continuous space while preserving coherence. Filters that assume adversarial inputs are detectably abnormal — high perplexity, odd token statistics — have little to catch here, because coherence is an explicit objective of the search.

**Source:** [arXiv:2411.14133](https://arxiv.org/abs/2411.14133)

## LLM Stinger

*Piyush Jha, Arnav Arora, Vijay Ganesh · 2024 (AAAI 2025) · `attacker-LLM` `reinforcement-learning`*

LLM Stinger trains an attacker model with reinforcement learning to generate adversarial suffixes. The target's feedback — how close its response came to the forbidden behaviour — serves as the reward signal, and existing attacks supply material to bootstrap and sustain the learning. Over training, the attacker shifts its generation distribution toward suffixes that reliably elicit unsafe outputs.

The payoff of the RL framing is amortisation. Once the generator is trained, the cost of searching for an attack has been absorbed into the policy; new questions and new targets can be attacked by sampling from the learned distribution rather than rerunning a fresh per-prompt search each time. In effect, LLM Stinger learns a reusable attack policy — a generator whose competence transfers beyond the specific prompts it was trained on, in the same amortising spirit as AdvPrompter but driven by a reinforcement-learning objective rather than supervised prompter training.

Conceptually, the method illuminates *why* these attacks succeed. A learned suffix generator will push its distribution toward whatever unsafe behaviours the defender's alignment failed to cover; the failures it finds are precisely the gaps between what alignment training anticipated and what the reward-maximising policy can reach. The attacker, in other words, is an automated cartographer of the holes in a model's safety coverage.

For defenders, that framing is the actionable part. Because an RL attacker explicitly optimises toward uncovered unsafe behaviours, alignment evaluated only on anticipated cases will overstate robustness. The useful posture is adversarial: probe a model with a trained generator to find where its safety distribution thins out, and treat those discovered gaps — not a static test set — as the real measure of coverage.

**Source:** [arXiv:2411.08862](https://arxiv.org/abs/2411.08862)

## MART — Multi-round Automatic Red-Teaming

*Suyu Ge et al. · 2023 (NAACL 2024) · `adversarial-co-training`*

MART is the defender-facing member of this chapter: it uses automated attacker-LLM search not merely to break a model but to *harden* one. An adversarial model and a defender model improve together through repeated rounds of interaction and training. In each round the attacker learns to produce prompts that challenge the current defender, and the defender learns to respond more safely to exactly those prompts; the updated defender then becomes the next round's harder target.

The mechanism works because co-training closes a loop that single-pass safety tuning leaves open. One round of alignment fixes the weaknesses visible at that moment but inevitably leaves others; MART's next round sends an attacker specifically after what survived, surfacing the residual failures so they can be trained away in turn. Iterating this drives both sides upward — an evolving attack-and-defence process rather than a one-off fix — and automates the generation of challenging adversarial data that would otherwise need manual red-teaming each round.

A naming caution matters here: the "multi-round" in MART refers to rounds of adversarial *training* between two models, not to a multi-turn conversational jailbreak against a deployed system. It belongs with the automated-search family because the challenging prompts are discovered by an attacker LLM, but its purpose is defensive co-training, which distinguishes it from every other method in this chapter and places it closest in spirit to MASTERKEY's arms-race framing turned toward the defender's side.

The defensive lesson is constructive. The same generate-and-refine machinery that produces attacks can be wired into the training loop to produce robustness, and doing so repeatedly is more effective than a single safety pass — because each round re-aims the attacker at whatever the last round failed to cover. MART makes the case that red-teaming is most valuable as a continuous, integrated part of model development rather than a pre-release checkpoint.

**Source:** [arXiv:2311.07689](https://arxiv.org/abs/2311.07689) · [NAACL](https://aclanthology.org/2024.naacl-long.107/)

## ASTRA

*Xu Liu, Yan Chen, Kan Ling et al. · 2026 — ACL · `attacker-LLM` `strategy-memory`*

ASTRA represents the mature form of the strategy-library idea: a search framework organised around structured memory and the evolution of strategies over time. From each attacker–target interaction it distils reusable strategies, and rather than pooling them indiscriminately it maintains *separate collections* for strategies that have proven effective, those that look promising but are unproven, and those that have failed.

That three-way partition is the mechanism's key insight. Retrieval draws on the effective and promising collections to reuse ideas that have worked or might, while the recorded failures act as a constraint — steering the search away from approaches already shown to be dead ends. The ineffective pile is as valuable as the effective one, because knowing what *not* to try is what keeps an evolving searcher from wasting its query budget re-exploring the same losing ground. Strategies move between collections as evidence accumulates, so the memory is not a static cache but an evolving model of the target's weaknesses.

ASTRA extends the lineage running through AutoDAN-Turbo: both centre on accumulating and reusing strategies, but ASTRA's contribution is the structured, graded memory and the explicit use of failure to shape future exploration. It is the clearest statement in this chapter of where the family has arrived — attacks as an evolving, self-organising body of knowledge about a target rather than a sequence of independent attempts.

The defensive reading is that the most capable automated attackers now *learn from being refused*. A guardrail that blocks one approach feeds the ineffective collection and makes the attacker's remaining search more efficient. Defences must therefore generalise across whole families of strategy, and defenders should expect that observed-and-blocked attacks are being catalogued as negative evidence that sharpens, rather than blunts, the adversary.

**Source:** [ACL 2026](https://aclanthology.org/2026.acl-long.1843/)

---
[← 03 · Long-context](03-long-context.md) · [Index](../README.md) · [Next: 05 · Gradient & token-level →](05-gradient-and-token-level.md)
