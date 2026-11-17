# 02 · Multi-turn & conversational escalation

[← Back to index](../README.md)

> Never ask directly. Walk the model there over several turns, using its own context and cooperativeness against it.

Most safety evaluation treats a model like a vending machine: a prompt goes in, a response comes out, and the response is judged in isolation. The attacks in this chapter defeat that assumption by design. Rather than asking for forbidden content outright, they distribute harmful intent across a sequence of individually benign turns. No single message trips a refusal, because no single message is the attack — the attack is the *trajectory*. The model's own fluency, cooperativeness, and drive toward conversational consistency become the attack surface.

The unifying mechanism is accumulation. A conversation establishes context, and that context quietly reshapes what the model treats as the "default" next thing to say. An attacker who opens with an abstract, wholly reasonable question has already changed the baseline; each follow-up then references what the model has *already produced* and nudges one small step further. Because the model evaluates each turn against the local frame of the dialogue rather than re-assessing the global intent from scratch, it slides across lines it would have refused to cross in a single shot. Variants of this idea recur throughout the chapter: gradual escalation (Crescendo, FITD), benign-narrative dilution (Deceptive Delight), evaluator-role reframing (Bad Likert Judge), self-reinforcing context (Echo Chamber), and forged history (Context Compliance Attack), among others.

What makes this one of the most active frontiers in red-teaming is that the attacker side has become *automated and adaptive*. Early techniques were hand-crafted scripts; the current generation includes agentic testers that reason about what is working (GOAT), learned attackers trained with turn-level feedback (Siren), and tree-search systems that branch over alternative conversational futures and retain partial compliance (Tempest). These do not follow a fixed escalation recipe — they search the space of dialogues for paths that compound into a violation.

The defensive takeaway is structural, not cosmetic. A guardrail that inspects each turn independently is blind to these attacks by construction, because every turn is locally innocuous. Robust defense requires reasoning over the whole conversation: tracking cumulative intent, detecting escalation and topic-drift patterns, distrusting client-supplied history (keep conversational state server-side), and judging the *usefulness* of what has been assembled across turns rather than merely scanning for refusal language in the latest reply. Treat the dialogue — not the prompt — as the unit of safety.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (brief) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Crescendo** | Microsoft (Russinovich, Salem, Eldan) | 2024 | multi-turn; black-box | Canonical gradual escalation: benign opener, each turn builds on the model's own prior answers. Automated as *Crescendomation*. | [arXiv:2404.01833](https://arxiv.org/abs/2404.01833) · [site](https://crescendo-the-multiturn-jailbreak.github.io/) |
| **Deceptive Delight** | Palo Alto Unit 42 | 2024 — vendor advisory | multi-turn; black-box | Blend benign and unsafe topics in one narrative, then ask the model to expand the parts that matter. | [Unit 42](https://unit42.paloaltonetworks.com/jailbreak-llms-through-camouflage-distraction/) |
| **Bad Likert Judge** | Palo Alto Unit 42 | 2025 — vendor advisory | multi-turn; evaluator abuse | Cast the model as a harmfulness judge, then ask it to illustrate a top-of-scale example. | [Unit 42](https://unit42.paloaltonetworks.com/multi-turn-technique-jailbreaks-llms/) |
| **Context Compliance Attack (CCA)** | Microsoft (Russinovich, Salem) | 2025 | history forgery; optimisation-free | Inject a fabricated prior "assistant" turn; model complies to stay consistent. Server-side state is immune. | [PyRIT](https://github.com/Azure/PyRIT) |
| **GOAT — Generative Offensive Agent Tester** | Meta (Pavlova et al.) | 2024 | agentic multi-turn; automated | Attacker agent runs an Observation–Thought–Strategy–Response loop, switching tactics adaptively. | [arXiv:2410.01606](https://arxiv.org/abs/2410.01606) |
| **ActorAttack** ("Derail Yourself") | Qibing Ren, Hao Li, Dongrui Liu et al. | 2024 — arXiv | multi-turn | Builds routes through semantically related actors/entities as clues toward a disallowed target. | [arXiv:2410.10700](https://arxiv.org/abs/2410.10700) |
| **FITD — Foot-In-The-Door** | Zixuan Weng, Xiaolong Jin, Jinyuan Jia, Xiangyu Zhang | 2025 — EMNLP | multi-turn; persuasion | Gradual commitment via bridge prompts chaining small concessions to larger requests. | [EMNLP 2025](https://aclanthology.org/2025.emnlp-main.100/) |
| **Speak Easy** | Yik Siu Chan, Narutatsu Ri, Yuxin Xiao, Marzyeh Ghassemi | 2025 — ICML | multi-turn; encoding/obfuscation | Task decomposition plus multilingual exchanges; partial pieces recombine into actionable output. | [arXiv:2502.04322](https://arxiv.org/abs/2502.04322) · [ICML](https://proceedings.mlr.press/v267/chan25b.html) |
| **Echo Chamber** | NeuralTrust (Ahmad Alobaid); later paper w/ Jordà Roca, Castillo, Vendrell | 2025 disclosure; 2026 arXiv | multi-turn; persuasion | Early cues set an interpretive frame; later turns cite the model's own statements to reinforce it. See [Appendix C](appendix-C-aliases-and-dedup.md) — Echo Chamber ≠ EchoLeak. | [NeuralTrust](https://neuraltrust.ai/blog/echo-chamber-context-poisoning-jailbreak) · [arXiv:2601.05742](https://arxiv.org/abs/2601.05742) |
| **Siren** | Yi Zhao, Youzhi Zhang | 2025 — arXiv | multi-turn; attacker-LLM search | Learned attacker trained with turn-level preference feedback, adapting each move to the defender. | [arXiv:2501.14250](https://arxiv.org/abs/2501.14250) |
| **Tempest** (formerly Siege) | Andy Zhou, Ron Arel | 2025 — ACL | multi-turn; tree search | Search tree over conversation continuations, retaining and building on partial compliance. | [arXiv:2503.10619](https://arxiv.org/abs/2503.10619) |

---

## Crescendo

*Microsoft (Russinovich, Salem, Eldan) · 2024 · `multi-turn` `black-box`*

Crescendo is the canonical gradual-escalation attack, and the template against which most later multi-turn methods are measured. It opens with a wholly benign, abstract question — the kind of thing a curious student or researcher might ask — and then proceeds in small increments. Each follow-up references the model's *own prior answers* and nudges the conversation one modest step closer to the forbidden objective. There is no single moment where the request becomes egregious; the escalation is engineered precisely so that each turn is only marginally more pointed than the last.

The reason this bypasses alignment is that the model evaluates each new request against the context it has already helped build, not against the ultimate destination. Having just produced a reasonable, on-topic answer, the model treats the next small extension as a natural continuation rather than a policy boundary. Refusal training tends to fire on blatant, out-of-the-blue requests; Crescendo never presents one. By the time the conversation reaches content the model would have refused in a single shot, that content sits at the end of a chain of its own cooperative, apparently legitimate responses — and consistency pressure makes refusing feel like contradicting itself.

Crescendo matters because it made the single-turn evaluation blind spot concrete and reproducible, and because the authors released **Crescendomation**, an automated pipeline that drives the escalation loop without a human in the seat. That automation is what elevated gradual escalation from an anecdote to a scalable, benchmarkable threat and seeded a wave of follow-on research. Techniques such as FITD and Echo Chamber are best understood as variations on the Crescendo insight — local coherence and self-generated context outweighing renewed global safety assessment. The defensive lesson is blunt: any guardrail that scores turns independently is defeated by construction, because Crescendo's harm lives in the trajectory, not in any one message. Defenses must model cumulative intent and detect the escalation pattern itself.

**Source:** [arXiv:2404.01833](https://arxiv.org/abs/2404.01833) · [site](https://crescendo-the-multiturn-jailbreak.github.io/)

## Deceptive Delight

*Palo Alto Unit 42 · 2024 — vendor advisory · `multi-turn` `black-box`*

Deceptive Delight asks the model to weave a *single narrative that blends benign and unsafe topics together*, then, over a short sequence of turns, requests that it expand the portions that matter. The harmful subject is never the focus of the conversation; it is one strand braided into an otherwise innocuous story, and the request to elaborate reads as a natural creative continuation rather than a demand for prohibited content.

The mechanism exploits dilution of the safety signal. When an unsafe topic is embedded among benign ones inside a coherent narrative the model itself constructed, the salience of the harmful element drops: the model's attention is distributed across the whole scene, and the framing primes it to treat "elaborate on your own story" as a request about the fiction rather than about the underlying topic. The incremental expansion over two to three turns then extracts detail that a direct request would have blocked.

Unit 42 reported that the technique tended to succeed within roughly three steps across the eight models they examined, with diminishing returns after the third turn — a profile consistent with the broader observation that these conversational attacks front-load their gains. As a vendor advisory, those figures are reported, not independently reproduced here. Deceptive Delight sits alongside Crescendo in the "benign framing" family, but its distinctive move is combinatorial camouflage — hiding the target among distractors in a shared narrative — rather than pure step-by-step escalation. The defensive lesson is that safety evaluation cannot stop at the topic list of a conversation; it must assess which elements are actually being developed and extracted, and resist treating a fictional or multi-topic wrapper as a license to elaborate on the dangerous strand.

**Source:** [Unit 42](https://unit42.paloaltonetworks.com/jailbreak-llms-through-camouflage-distraction/)

## Bad Likert Judge

*Palo Alto Unit 42 · 2025 — vendor advisory · `multi-turn` `evaluator abuse`*

Bad Likert Judge turns the model's own evaluation competence against it. The model is cast in the role of a judge asked to score candidate responses for harmfulness on a Likert scale — say, 1 to 5. Having accepted the evaluator frame, it is then asked to *illustrate* what a top-of-scale example would look like. The catch is that the maximally harmful illustration the model is now helpfully producing is itself the forbidden content.

The reason this bypasses alignment is a reframing of the generative act. Producing harmful content is something the model is trained to refuse; *analyzing* and *grading* content is something it is trained to do well and to treat as neutral, even virtuous, work. By routing the request through the judging frame, the attack recasts "write the dangerous thing" as "demonstrate the scoring rubric," which the model does not perceive as a policy violation. The evaluator persona supplies a rationalization that disarms the refusal reflex.

Unit 42 reported a substantial lift over single-shot prompting for this technique; those are vendor-reported figures rather than independently reproduced results. Bad Likert Judge is notable as an evaluator-abuse attack — a cousin of the role-play methods in the previous chapter, but specifically weaponizing the model's assessment and rubric-following behavior rather than a fictional character. It also foreshadows a broader concern: as models are increasingly deployed as graders and "LLM-as-judge" components, the judging capability is itself an attack surface. The defensive lesson is that assigning a model an evaluative or meta-cognitive role does not neutralize the safety-relevant content it generates in that role; guardrails must inspect the illustrative output, not just the ostensibly analytical framing.

**Source:** [Unit 42](https://unit42.paloaltonetworks.com/multi-turn-technique-jailbreaks-llms/)

## Context Compliance Attack (CCA)

*Microsoft (Russinovich, Salem) · 2025 · `history forgery` `optimisation-free`*

The Context Compliance Attack is the odd one out in this chapter: it exploits a *deployment architecture* weakness rather than a reasoning weakness. Many chat deployments are stateless on the server and trust the conversation history that the *client* submits with each request. CCA abuses that trust by injecting a fabricated earlier "assistant" turn — one in which the model supposedly volunteered to help with the forbidden task — and then sending a short user reply such as an affirmative "yes, go on." The model, reading a history in which it already agreed, continues in order to stay consistent with words it never actually said.

What makes this effective is that the model has no way to distinguish genuine prior turns from forged ones; it treats the supplied context as ground truth about what it previously committed to. Consistency pressure, the same force several other attacks here lean on, does the rest: having "already offered," refusing now would be incoherent. No optimization, gradient access, or clever prompt engineering is required — just a doctored transcript.

The decisive feature of CCA is that it has a clean, architectural remedy. Systems that keep conversational state server-side, where the history cannot be forged by the client, are simply immune: the attacker cannot assert turns that the server did not generate. This makes CCA a crisp deployment-design lesson rather than an open research problem — the fix is to never trust client-supplied history for safety-relevant state. The technique ships as a component in Microsoft's open-source **PyRIT** risk-identification toolkit, which is also where practitioners can study and test for it. The defensive takeaway generalizes beyond this one attack: authenticate the provenance of everything the model treats as its own prior behavior.

**Source:** [PyRIT](https://github.com/Azure/PyRIT)

## GOAT — Generative Offensive Agent Tester

*Meta (Pavlova et al.) · 2024 · `agentic multi-turn` `automated`*

GOAT is an attacker *agent* designed to converse the way a real adversarial user would, rather than to execute a fixed script. On each turn it runs an Observation–Thought–Strategy–Response loop: it reads the model's latest reply (observation), reasons about what is and isn't working (thought), selects an approach from a toolbox of known jailbreak techniques (strategy), and composes the next message (response). The result is an adversary that blends and switches tactics adaptively as the dialogue unfolds.

The reason this is more dangerous than any single hand-crafted method is that it closes the loop between attack and feedback. Static multi-turn attacks commit to one escalation path; GOAT observes how the defender is responding and reallocates its effort accordingly, abandoning tactics that draw refusals and doubling down on ones that gain ground. Because it can mix strategies — a little gradual escalation here, a reframing there — it is harder to characterize with any single detection signature, and it approximates the open-ended probing of a persistent human red-teamer at machine scale.

GOAT matters as part of the shift from technique catalogs to *automated adversarial agents*, and it is practically significant because it is implemented as a probe in the open-source **garak** scanner, making agentic multi-turn testing accessible to defenders evaluating their own systems. It is a conceptual sibling of Siren and Tempest — all three replace a fixed recipe with search or learning — but GOAT's distinctive framing is an explicit reasoning loop over a library of human-known tactics. The defensive lesson is that robustness must be measured against adaptive, feedback-driven attackers; passing a suite of static multi-turn probes says little about how a system fares against an agent that reacts to its defenses in real time.

**Source:** [arXiv:2410.01606](https://arxiv.org/abs/2410.01606)

## ActorAttack ("Derail Yourself")

*Qibing Ren, Hao Li, Dongrui Liu et al. · 2024 — arXiv · `multi-turn`*

ActorAttack constructs conversational routes toward a disallowed target by traversing a web of semantically related actors, entities, and concepts. Rather than approaching the objective head-on, it begins with innocuous topics that function as *clues* — nodes connected, through a chain of plausible associations, to the forbidden endpoint. Because many such chains exist, the method can offer several different paths to the same goal, making it resilient when one route is refused.

The mechanism exploits the model's local, turn-by-turn reasoning. Each hop in the chain is individually reasonable: moving from a benign concept to a closely related one is exactly the kind of associative progression ordinary conversation is made of. The model evaluates each step against its immediate neighbor and finds nothing objectionable, never standing back to reassess where the accumulated path is heading. The harmful intent is an emergent property of the full route, which the model does not reconstruct globally — hence the project's evocative framing of the model "derailing itself" along a track the attacker merely laid.

ActorAttack's contribution is to make the *structure* of multi-turn attacks explicit: it frames the attack surface as a graph of related entities and the jailbreak as pathfinding through it, with redundancy built in via alternative links. This situates it within the same accumulation family as Crescendo and FITD, but its signature idea is semantic routing through actor/entity relationships rather than monotonic escalation of a single theme. The defensive lesson reinforces the chapter's thesis from a new angle: safety assessment that only checks adjacent turns is exploitable, because intent can be smuggled across a sequence of locally sensible associative hops. Detection needs a global view of where a conversation's chain of topics is converging.

**Source:** [arXiv:2410.10700](https://arxiv.org/abs/2410.10700)

## FITD — Foot-In-The-Door

*Zixuan Weng, Xiaolong Jin, Jinyuan Jia, Xiangyu Zhang · 2025 — EMNLP · `multi-turn` `persuasion`*

Foot-In-The-Door takes its name from the classic social-psychology phenomenon in which agreeing to a small request makes a person more likely to agree to a larger one later. FITD operationalizes this against language models through a sequence of related requests that models gradual commitment: *bridge prompts* connect small initial concessions to progressively more problematic later asks, so that each request is only slightly beyond what the model has already granted.

The hypothesized mechanism is that a model's earlier answers set the expectations for its later ones. Once the model has produced a mildly sensitive response, that response becomes part of the context against which the next request is judged, and declining would introduce a local inconsistency with its own established cooperativeness. The authors' framing is that local consistency and self-generated context can outweigh renewed, from-scratch safety assessment — the same failure the chapter returns to repeatedly, here cast explicitly in the vocabulary of commitment and persuasion.

FITD is a named method adjacent to, but distinct from, Crescendo: both are gradual-escalation attacks, but FITD foregrounds the persuasion-theoretic account and the role of bridge prompts that deliberately engineer the small-to-large commitment ladder, and it was presented at EMNLP 2025 as a formalized technique with that specific framing. Its value for defenders is diagnostic as much as offensive: it names a concrete psychological lever — incremental commitment — that conversational guardrails need to recognize. The defensive lesson is that a model's willingness to continue should not be anchored to its own prior concessions; safety assessment must be re-performed against the cumulative request at full strength on every turn, treating earlier compliance as something to re-examine rather than defer to.

**Source:** [EMNLP 2025](https://aclanthology.org/2025.emnlp-main.100/)

## Speak Easy

*Yik Siu Chan, Narutatsu Ri, Yuxin Xiao, Marzyeh Ghassemi · 2025 — ICML · `multi-turn` `encoding/obfuscation`*

Speak Easy combines task decomposition with multilingual interaction. A harmful objective is broken into a set of smaller sub-requests, and those sub-requests are spread across separate exchanges — often routed through different languages — so that each one, viewed alone, yields only an innocuous fragment. The individually benign pieces become genuinely actionable only once an attacker reassembles them.

The mechanism targets two gaps at once. The decomposition gap means that safety filters tuned to recognize a complete harmful request miss its disaggregated parts, each of which falls below any single-turn threshold. The multilingual gap means that safety alignment is often weaker or less consistent outside high-resource languages, so routing a sub-task through another language can slip past protections that would catch its English equivalent. Crucially, Speak Easy evaluates the *usefulness* of the recombined harmful output — whether the assembled result actually helps accomplish the task — rather than merely checking whether refusal wording disappeared, a methodological choice that exposes real capability leakage instead of cosmetic compliance.

This makes Speak Easy broader than the familiar trick of translating one request into a low-resource language to dodge a filter; its contribution is to demonstrate *compositional* safety gaps that span both decomposition steps and languages simultaneously. Presented at ICML 2025, it reframes part of the multi-turn problem as a measurement problem: if you only score individual turns for refusal, you will systematically overestimate safety against an adversary who harvests and stitches partial answers. The defensive lesson is twofold — guardrails must reason about what a conversation assembles across turns and languages, and safety evaluation should measure the actionable utility of the combined output, not the presence or absence of a refusal in any one reply.

**Source:** [arXiv:2502.04322](https://arxiv.org/abs/2502.04322) · [ICML](https://proceedings.mlr.press/v267/chan25b.html)

## Echo Chamber

*NeuralTrust (Ahmad Alobaid); later paper with Jordà Roca, Castillo, Vendrell · 2025 disclosure; 2026 arXiv · `multi-turn` `persuasion`*

Echo Chamber works by context poisoning: early conversational cues plant an interpretive frame that the model is then invited to help elaborate. In subsequent turns the attacker refers back to the model's *own prior statements* and reinforces the frame, so that the conversation increasingly echoes and amplifies a premise the model itself articulated. Unsafe implications emerge gradually from this accumulated, self-referential context rather than from any single explicit request.

The reason this bypasses alignment is the self-reinforcing nature of the context. Once the model has contributed statements that support a particular framing, those statements carry the authority of the model's own voice; citing them back is more persuasive than an attacker asserting the same thing, because the model is being asked to stay consistent with itself. Each turn tightens the frame, and the harmful conclusion arrives as an apparent entailment of a worldview the conversation has collectively built — never as a bald demand the safety training is tuned to reject. The name captures the dynamic: the model's outputs are reflected back and amplified until the frame dominates.

Echo Chamber was first disclosed by NeuralTrust in 2025 and later developed into an arXiv paper with additional authors. As with other vendor-originated disclosures, its effectiveness claims are reported by the originators and are not treated here as independently reproduced. The technique belongs to the self-reinforcing-context branch of the accumulation family, close in spirit to Crescendo and FITD but distinguished by its emphasis on poisoning the interpretive frame and leveraging the model's own statements as persuasive anchors. A naming caution: Echo Chamber is unrelated to the EchoLeak vulnerability — see [Appendix C](appendix-C-aliases-and-dedup.md) for the alias deduplication. The defensive lesson is that a model's own earlier outputs should not be granted unexamined authority later in the same conversation; guardrails must watch for frames that build cumulatively and for later turns that lean on model-generated premises to justify unsafe conclusions.

**Source:** [NeuralTrust](https://neuraltrust.ai/blog/echo-chamber-context-poisoning-jailbreak) · [arXiv:2601.05742](https://arxiv.org/abs/2601.05742)

## Siren

*Yi Zhao, Youzhi Zhang · 2025 — arXiv · `multi-turn` `attacker-LLM search`*

Siren replaces the hand-written escalation script with a *learned* attacker. The attacking model is trained using turn-level feedback and preference-based learning, so that it acquires a policy for adapting its next conversational move to how the defender just responded. Instead of committing to a predetermined sequence, it learns, from experience across many dialogues, which kinds of moves tend to advance toward a successful jailbreak and which provoke refusals.

The reason this is potent is that it directly optimizes for the dynamic that static attacks only approximate by hand. A fixed script cannot react when the defender behaves unexpectedly; a learned attacker trained on turn-level preferences internalizes a general strategy for steering conversations and can generalize that strategy to new targets and topics. This closes the loop between attacker and defender in a data-driven way, producing an adversary whose behavior is shaped by what actually worked rather than by a researcher's intuition about what should work.

Siren's significance lies in explicitly framing multi-turn jailbreaking as a learning problem and in targeting the gap between single-turn safety evaluation and genuinely dynamic adversarial dialogue — the central blind spot this chapter documents. It sits in the automated-adversary family alongside GOAT and Tempest; where GOAT reasons over a toolbox and Tempest searches a tree, Siren's signature is a trained, preference-optimized attacker policy. (A naming note: this Siren is unrelated to the audio-domain paper *Sirens' Whisper*.) The defensive lesson mirrors GOAT's: because attackers can now be *trained* to exploit conversational dynamics, defenses must be evaluated against adaptive, learning adversaries, and static multi-turn test suites offer a false sense of security.

**Source:** [arXiv:2501.14250](https://arxiv.org/abs/2501.14250)

## Tempest (formerly Siege)

*Andy Zhou, Ron Arel · 2025 — ACL · `multi-turn` `tree search`*

Tempest frames a multi-turn jailbreak as a tree search. A search tree represents the alternative ways a conversation could continue, with each branch a possible next turn; the attacker explores this tree to discover which conversational paths accumulate into the largest policy violations. Rather than following a single line of attack and giving up when it stalls, Tempest can back up, try a different branch, and expand the paths that look most promising.

Its defining idea is that branches *retain and build on partial compliance* observed in earlier turns. When the defender concedes a little on one path — a fragment of sensitive detail, a softened refusal — that partial success is preserved in the tree and compounded by subsequent branches, so the search steadily aggregates small leaks into a substantial violation. This turns the accumulation mechanism central to the whole chapter into an explicit optimization target: the attacker is literally searching conversation histories for the trajectory that maximizes accumulated compliance, rather than hoping a single hand-designed escalation lands.

Tempest is the continuation of work earlier called Siege; the two names denote one evolving line of research, presented at ACL 2025. Within the automated-adversary family it is the search-based counterpart to GOAT's reasoning loop and Siren's learned policy, and its particular contribution is the tree-search formalism with partial-compliance retention. The defensive lesson is sobering: against a system that systematically explores many conversational futures and harvests every partial concession, even small, seemingly harmless leaks are not safe, because they become building blocks. Guardrails must avoid conceding fragments, treat partial compliance as itself a failure mode, and recognize that the relevant adversary may be searching, not scripting.

**Source:** [arXiv:2503.10619](https://arxiv.org/abs/2503.10619)

---
[← 01 · Role-play](01-roleplay-and-nesting.md) · [Index](../README.md) · [Next: 03 · Long-context →](03-long-context.md)
