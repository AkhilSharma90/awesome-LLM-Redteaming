# 05 · Gradient & token-level optimisation (white-box)

[← Back to index](../README.md)

> When you have the weights, optimise directly over tokens — even tokens no human would ever type.

When a red-teamer holds a model's weights, jailbreaking stops being a game of clever wording and becomes an optimisation problem. The techniques in this chapter treat the alignment layer as a differentiable objective: pick a target behaviour, express "the model should comply" as a loss, and search the discrete space of tokens for an input that minimises it. Because the search is guided by gradients rather than human intuition, it can land on strings that no person would ever type — fragments of punctuation, foreign-script tokens, and apparent gibberish that nonetheless steer the model with surgical precision. The lesson is uncomfortable but foundational: refusal behaviour trained by RLHF or similar methods is a shallow statistical tendency, and a sufficiently determined optimiser can find the seams.

The landmark result is GCG. It demonstrated two things that reshaped the field. First, a single adversarial suffix can be *universal* — effective across many different harmful requests rather than hand-tuned for one. Second, and more alarmingly, suffixes optimised against open-weight models often *transfer* to closed commercial systems the attacker never touched, because safety-tuned models share enough representational structure for an attack on one to generalise. Transferability turned a white-box technique into a practical threat against black-box deployments, and it is the thread that runs through most of the work that followed.

Almost everything else here is lineage. One branch pursues speed and efficiency — I-GCG, Faster-GCG, and MAGIC rework GCG's discrete search to find suffixes with less wasted computation, while BEAST drops gradients entirely in favour of beam search over the model's own output probabilities. A second branch pursues fluency and stealth — AutoDAN-Zhu, COLD-Attack, and FLRT fold readability or style constraints into the objective so the resulting attack text reads like plausible language rather than noise. A third branch pursues reuse and reach — AmpleGCG and AmpleGCG-Plus distil the fruits of many optimisation runs into a generator that emits fresh suffixes on demand, and PAL carries the whole approach into the black-box setting using a local surrogate.

For defenders, the lineage maps directly onto a defensive arms race. Early GCG suffixes have astronomically high perplexity, so a perplexity filter catches them cheaply — which is precisely why the fluent variants exist: they are engineered to pass exactly that check. The practical takeaway is that no single input-side heuristic is sufficient; perplexity filtering, paraphrasing, and similar screens raise the cost of an attack without closing the underlying gap. One naming note before the catalogue: there are two unrelated attacks called AutoDAN. The *genetic* AutoDAN (Liu et al.), which evolves readable prompts, lives in [Chapter 07](07-evolutionary-and-quality-diversity.md); the *gradient-based* AutoDAN-Zhu documented below is a different paper by a different team — see also [Appendix C](appendix-C-aliases-and-dedup.md).

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **GCG — Greedy Coordinate Gradient** | Andy Zou et al. | 2023 | white-box; universal & transferable | Gradient-guided greedy token swaps build an adversarial suffix that maximises an affirmative opening; suffixes can be universal and transfer to black-box models. | [arXiv:2307.15043](https://arxiv.org/abs/2307.15043) |
| **BEAST — Beam Search-based Adversarial Attack** | Vinu Sankar Sadasivan et al. | 2024 | gradient-free; fast | Beam search guided by the model's own output probabilities assembles an attack in about a GPU-minute; also effective at inducing hallucinations. | [arXiv:2402.15570](https://arxiv.org/abs/2402.15570) |
| **AutoDAN (Zhu)** — Interpretable Gradient-Based Attacks | Sicheng Zhu, Ruiyi Zhang, Bang An et al. | 2023 — arXiv | gradient/token-level | Generates *readable* adversarial text progressively under gradient guidance; a different paper from Liu's genetic AutoDAN — a genuine name collision. | [arXiv:2310.15140](https://arxiv.org/abs/2310.15140) |
| **AmpleGCG** | Zeyi Liao, Huan Sun | 2024 — COLM | gradient/token-level; attacker-LLM | Successful intermediate suffixes train a *generator* that learns a distribution over suffixes; sampling yields many attacks without re-running optimisation. | [arXiv:2404.07921](https://arxiv.org/abs/2404.07921) |
| **AmpleGCG-Plus** | Vishal Kumar, Zeyi Liao, Jaylen Jones, Huan Sun | 2024 — arXiv | gradient/token-level; attacker-LLM | Investigates training strategies for learning suffix distributions, improving useful candidates per attempt. A named AmpleGCG variant. | [arXiv:2410.22143](https://arxiv.org/abs/2410.22143) |
| **COLD-Attack** | Xingang Guo, Fangxu Yu, Huan Zhang, Lianhui Qin, Bin Hu | 2024 — arXiv | gradient/token-level; constrained optimisation | Frames adversarial prompts as constrained text generation balancing the attack objective with fluency and context; shows fluency checks are insufficient. | [arXiv:2402.08679](https://arxiv.org/abs/2402.08679) |
| **I-GCG — Improved GCG** | Xiaojun Jia, Tianyu Pang, Chao Du et al. | 2024 (ICLR 2025) | gradient/token-level | Modifies GCG's objectives and update process, varies response targets, and transfers progress across problems — stronger, more efficient optimisation. | [arXiv:2405.21018](https://arxiv.org/abs/2405.21018) |
| **Faster-GCG** | Xiao Li, Zhuhong Li, Qiongxiu Li et al. | 2024 — arXiv | gradient/token-level | Revisits GCG's discrete-search design to cut wasted computation; evaluates local performance and transfer. A search-procedure variant. | [arXiv:2410.15362](https://arxiv.org/abs/2410.15362) |
| **MAGIC — Model Attack Gradient Index GCG** | Jiahui Li, Yongchang Hao, Haoyu Xu, Xing Wang, Yu Hong | 2024 — arXiv | gradient/token-level | Studies how token-index gradients affect efficiency and prioritises productive suffix changes; identifies an "indirect effect" that wastes work. | [arXiv:2412.08615](https://arxiv.org/abs/2412.08615) |
| **PAL — Proxy-Guided Attack on LLMs** | Chawin Sitawarin, Norman Mu, David Wagner, Alexandre Araujo | 2024 — arXiv | gradient/token-level; surrogate-guided black-box | A local surrogate guides candidate generation for an API-only target; target feedback corrects for surrogate mismatch. Also introduces GCG++ and RAL. | [arXiv:2402.09674](https://arxiv.org/abs/2402.09674) |
| **FLRT — Fluent Student-Teacher Redteaming** | T. Ben Thompson, Michael Sklar | 2024 — arXiv | gradient/token-level; distillation | Optimises a target to resemble an unsafe *teacher* while favouring fluent, nonrepetitive text; permits flexible edits beyond fixed-position swaps. | [arXiv:2407.17447](https://arxiv.org/abs/2407.17447) |

---

## GCG — Greedy Coordinate Gradient

*The landmark token-level attack that proved alignment can be defeated by pure optimisation.*

GCG is the reference point for this entire chapter. Given full access to a model's weights, it appends an adversarial suffix to a harmful request and optimises that suffix so the model begins its reply with an affirmative, compliant opening rather than a refusal. The key difficulty is that tokens are discrete, so ordinary gradient descent does not apply directly. GCG works around this by using the gradient of the loss with respect to the input embeddings to score, at each suffix position, which candidate tokens would most reduce the loss, and then greedily evaluating and committing the most promising swaps. Iterating this coordinate-wise search drives the model toward compliance one token substitution at a time.

Why it works reveals something deep about current alignment. Safety tuning teaches a model to recognise and refuse certain surface patterns, but it leaves intact the underlying capability to produce the harmful content. The adversarial suffix exploits the gap: it nudges the model's internal state past the refusal boundary without changing the capability behind it. Because the optimiser is unconstrained by human readability, it exploits directions in token space that no natural prompt would reach, which is why GCG suffixes look like noise.

Two findings from the original work shook the field. A single suffix can be *universal*, working across many different harmful prompts rather than being tailored to one, which means an attacker amortises the cost of one expensive optimisation over unlimited reuse. And suffixes optimised against ensembles of open-weight models *transfer* to closed, black-box commercial systems the attacker cannot inspect — a consequence of shared representational structure among safety-tuned models. Transferability is what makes a white-box method a real-world black-box threat, and it motivated essentially every follow-up in this chapter.

The defensive lesson is twofold. First, because early GCG suffixes carry extremely high perplexity, a simple perplexity filter detects them at low cost — a genuinely useful first line of defence. Second, and more sobering, the attack demonstrates that refusal behaviour is not a hard safety guarantee but a soft statistical preference that optimisation can overcome, so defences must assume motivated adversaries will find inputs that slip past any single screen.

**Source:** [arXiv:2307.15043](https://arxiv.org/abs/2307.15043)

## BEAST — Beam Search-based Adversarial Attack

*Shows you don't need gradients — or much compute — to jailbreak.*

BEAST dispenses with gradients entirely. Instead of differentiating through the model, it uses beam search guided by the model's own next-token output probabilities to assemble an adversarial prompt, maintaining a set of promising candidate continuations and extending the most effective ones. Because it only needs the probabilities the model already exposes, it is strikingly cheap, reportedly producing an attack in roughly a GPU-minute rather than the much longer runs gradient methods require.

The tunable beam parameters give the attacker an explicit dial between competing goals: wider or deeper search raises success rates and can improve readability at the cost of speed, while a narrower search is faster but weaker. This flexibility makes BEAST a practical tool for rapid red-teaming at scale. Notably, the same machinery is effective at inducing hallucinations, not just safety bypasses — a reminder that optimisation-style attacks target model behaviour broadly, not only the alignment layer.

For defenders, BEAST undercuts any assumption that gradient access is the gating requirement for serious token-level attacks. A capable adversary who can observe output probabilities — a weaker and more common form of access than full weights — can mount an efficient search, so defences that rely on hiding gradients provide little protection.

**Source:** [arXiv:2402.15570](https://arxiv.org/abs/2402.15570)

## AutoDAN (Zhu) — Interpretable Gradient-Based Attacks

*A readability-aware gradient attack — and one half of a genuine name collision.*

This AutoDAN, by Sicheng Zhu, Ruiyi Zhang, Bang An and colleagues, generates adversarial text that is *readable* rather than gibberish. It builds the attack progressively under gradient guidance, but its objective is not purely the attack loss: it also rewards language plausibility, so the optimiser prefers token choices that keep the growing text coherent. The result is adversarial input that reads more like natural language than a GCG suffix does.

The point of readability is evasion. Many deployed defences treat suspicious input as input with high perplexity — the signature of optimised gibberish. By explicitly balancing attack effectiveness against fluency, this method produces text that stays below that threshold, slipping past perplexity-based screens that would catch the earlier noise-like suffixes. It is an early demonstration of the fluent-attack branch of the lineage.

A naming caution is essential here. This is a *different paper* from the genetic AutoDAN by Xiaogeng Liu and collaborators, which appears in [Chapter 07](07-evolutionary-and-quality-diversity.md) and reaches readable prompts through evolutionary search rather than gradients. The shared name is a genuine collision in the literature; they are unrelated methods with different mechanisms and different authors.

**Source:** [arXiv:2310.15140](https://arxiv.org/abs/2310.15140)

## AmpleGCG

*Turns the by-products of GCG optimisation into a trained suffix generator.*

AmpleGCG observes that a GCG run discards most of what it produces: along the way to a final suffix, the search generates many intermediate candidates, a number of which already succeed. Rather than throw these away, AmpleGCG collects successful suffixes and uses them as training data for a generator model — an attacker LLM that learns a distribution over effective suffixes conditioned on a given request.

Once that generator is trained, the economics of attacking change. Instead of running an expensive, iterative optimisation for each new target request, the attacker simply samples the generator, which emits multiple candidate suffixes quickly. This amplifies GCG in two senses: it reuses the search effort already spent, and it packages the learned regularities of successful attacks into a reusable, on-demand tool. The attack surface remains the same unusual token sequences and cross-model transfer that GCG exposed; what changes is the cost and speed of producing fresh attacks.

The defensive implication is that the cost asymmetry worsens over time. Expensive one-off optimisation can be distilled into a cheap generator, so the marginal cost of a new attack falls sharply once the up-front work is done. Defences evaluated only against the cost of raw GCG may badly underestimate the throughput a motivated adversary can sustain.

**Source:** [arXiv:2404.07921](https://arxiv.org/abs/2404.07921)

## AmpleGCG-Plus

*A refinement of AmpleGCG's generator training for better candidates per attempt.*

AmpleGCG-Plus is a named successor to AmpleGCG by an overlapping team, and it focuses on *how* the suffix generator is trained. It investigates strategies for learning more effective adversarial-suffix distributions — for example, how the generator's training data is assembled and curated — so that the resulting model produces useful candidates more reliably and in fewer attempts.

The underlying threat model is unchanged: the attack still exploits unusual token sequences and the transferability of suffixes across models, and it is explicitly a variant within the AmpleGCG family rather than a new category of attack. The contribution is efficiency and quality of the generator, which sharpens the same cost asymmetry that AmpleGCG introduced.

For defenders, the lesson is continuity. Attacker-side tooling improves incrementally and cumulatively, and each refinement lowers the number of queries needed to succeed. A defence that merely raises the attempt count required should expect that bar to be eroded by the next iteration of generator training.

**Source:** [arXiv:2410.22143](https://arxiv.org/abs/2410.22143)

## COLD-Attack

*Reframes adversarial prompting as constrained, controllable text generation.*

COLD-Attack casts the problem as constrained text generation: the optimiser must satisfy the attack objective while simultaneously respecting constraints on fluency, style, and contextual compatibility. By treating these as a joint controllable-generation problem, it can produce not only suffixes but also adversarial rewrites of a request and context-insertion attacks — a broader repertoire than fixed-position suffix search.

The mechanism matters because it decouples "effective against safety" from "looks like noise." The optimisation deliberately steers toward text that is both coherent and on-topic, so the output can read as an ordinary, stylistically appropriate passage while still driving the model toward unsafe behaviour. This makes it a flexible instrument for studying how attacks can be camouflaged within natural context.

The defensive takeaway is direct and important: superficial fluency checks are not a safety control. A screen that assumes coherent, low-perplexity, on-topic input is benign will pass COLD-Attack outputs, because coherence and harmlessness are independent properties. Defences must evaluate what an input does to the model, not merely how readable it is.

**Source:** [arXiv:2402.08679](https://arxiv.org/abs/2402.08679)

## I-GCG — Improved GCG

*A stronger, more efficient reworking of GCG's core optimisation.*

I-GCG keeps GCG's premise — a token-level suffix search guided by gradients — but reworks the objective and the update process to make the search more effective. It explores a wider range of response targets rather than a single fixed affirmative opening, and it adapts how many token positions are changed together during each update, so the search can make faster progress through the loss landscape.

A further gain comes from initialisation: rather than starting each optimisation from scratch, I-GCG can transfer progress from easier related problems, giving the search a head start on harder ones. The net effect is a stronger and more efficient attack on the same underlying surface that GCG targets — not a new semantic category of jailbreak, but a more capable optimiser.

For defenders, I-GCG is a reminder that the original GCG success and runtime figures are a floor, not a ceiling. As the optimisation is refined, both success rates and efficiency improve, so any risk assessment pinned to baseline GCG numbers should be treated as optimistic.

**Source:** [arXiv:2405.21018](https://arxiv.org/abs/2405.21018)

## Faster-GCG

*Revisits GCG's discrete search to cut wasted computation.*

Faster-GCG focuses squarely on the efficiency of the discrete-search procedure at the heart of GCG. By re-examining how candidate token swaps are proposed and selected, it reduces the computation wasted on unproductive evaluations, so an effective suffix can be found with less work. The emphasis is on the practical cost of the search rather than on changing what the attack ultimately produces.

Like other members of the lineage, it is evaluated both on local attack performance against the model being optimised and on transfer to other models, keeping the transferability property that makes these attacks consequential beyond the white-box setting. It is best understood as a search-procedure variant: the change is in the method and its cost profile, not in the category of vulnerability.

The defensive implication mirrors that of the other efficiency variants — the price of mounting a token-level attack keeps falling. Defences whose viability depends on the attack being computationally expensive are on shifting ground.

**Source:** [arXiv:2410.15362](https://arxiv.org/abs/2410.15362)

## MAGIC — Model Attack Gradient Index GCG

*Uses token-index gradient information to prioritise productive suffix edits.*

MAGIC studies how the gradients associated with individual token indices influence GCG's optimisation efficiency, and uses that information to decide which changes to a suffix are worth making. By prioritising the positions and substitutions most likely to reduce the loss, it concentrates effort where it pays off and avoids spending it elsewhere.

A notable finding is what the authors describe as an "indirect effect" — a source of unnecessary optimisation work that arises from how index gradients propagate. Recognising and accounting for this effect lets MAGIC trim wasted computation, placing it firmly within the efficiency-focused branch of the GCG lineage alongside I-GCG and Faster-GCG. The vulnerability it exploits is the same; the contribution is a smarter use of the gradient signal already available.

For defenders, MAGIC reinforces the pattern that researchers continue to extract more efficiency from the same gradient information, so the computational cost of white-box token-level attacks should be expected to keep declining as the search is better understood.

**Source:** [arXiv:2412.08615](https://arxiv.org/abs/2412.08615)

## PAL — Proxy-Guided Attack on LLMs

*Carries gradient-style optimisation into the black-box, API-only setting.*

PAL addresses the case where the target is exposed only through an API and its internal gradients are unavailable. It uses a local *surrogate* model — one the attacker does control — to generate candidate inputs via gradient-guided search, then uses feedback from the real target to compensate for the differences between the surrogate and the deployed model. In effect, the surrogate supplies the gradient signal while the target supplies ground truth, letting optimisation proceed without ever touching the target's weights.

This matters because it bridges the gap between the white-box lab setting and the black-box reality of commercial deployments. It shows that restricting an attacker to API access does not, on its own, neutralise optimisation attacks; it only raises their cost and changes their shape. The paper also introduces GCG++ and RAL as baselines, situating PAL within the broader comparison of token-level methods.

The defensive lesson is that limiting direct access is a cost-raising measure, not a barrier. Query-based feedback can substitute for internal gradients, so rate limiting, query monitoring, and anomaly detection on API traffic become important complements to any model-side defence.

**Source:** [arXiv:2402.09674](https://arxiv.org/abs/2402.09674)

## FLRT — Fluent Student-Teacher Redteaming

*Distils an unsafe teacher into the target while keeping the attack text fluent.*

FLRT frames the attack as a distillation problem. The optimisation encourages the target model — the "student" — to resemble an unsafe "teacher" in its outputs or internal representations, effectively pulling the target toward the teacher's willingness to comply. Layered on top of this is an objective that rewards fluent, nonrepetitive attack text, and the search permits more flexible modifications to the input than the rigid fixed-position token replacement of classic GCG.

The combination places FLRT in the fluent-attack branch of the lineage: it seeks inputs that both achieve the unsafe objective and read as coherent language, rather than high-perplexity noise. The teacher-student framing gives the optimiser a rich target signal — matching a model known to produce the unwanted behaviour — while the fluency and flexibility constraints shape the attack into something that evades readability-based screens.

As with COLD-Attack and AutoDAN-Zhu, the defensive takeaway is that treating readability as a proxy for safety is a mistake. FLRT's outputs are engineered to be fluent precisely so that perplexity and coherence checks pass, which means such filters must be regarded as raising attacker cost, never as reliable safety boundaries.

**Source:** [arXiv:2407.17447](https://arxiv.org/abs/2407.17447)

---
[← 04 · Attacker-LLM search](04-attacker-llm-search.md) · [Index](../README.md) · [Next: 06 · Black-box brute force →](06-black-box-brute-force.md)
