# 08 · Multi-agent & mixture-of-attackers

[← Back to index](../README.md)

> Don't send one attacker — orchestrate a team. Specialised agents plan, attack, judge, and recombine strategies, and the best tactics are *merged*.

The methods in this chapter mark a shift from optimising a single attacker to orchestrating a *team*. Instead of grinding one prompt toward success, they divide the work across specialised roles — a planner that sets strategy, an attacker that carries out the conversation, a judge or verifier that scores progress, and an optimiser that rewrites when the attack stalls. The division of labour lets the system reason about an attack over many turns, recover from refusals, and pursue several lines at once, much as a human red team does.

The defining move is *recombination*. Rather than betting everything on one line of attack, these systems treat tactics as components to be mixed: they cluster which styles are working, merge promising attacks to compound their strengths, and prune near-duplicates to spend their query budget wisely. Because strengths combine rather than compete, the whole can exceed any single tactic — and the search explores strategy *space* rather than just prompt space.

A structural consequence runs through the chapter: these orchestrators increasingly treat the rest of this catalogue as a **library of moves**. A multi-agent system may invoke brute-force sampling, evolutionary search, encoding tricks, or known jailbreak templates as sub-routines. For a defensive survey this matters for bookkeeping — an orchestrator should be counted once, for its contribution to planning and recombination, not re-counted for every underlying attack it happens to call. The defensive takeaway is that compositional, multi-turn adversaries are the realistic threat model: single-turn, single-prompt evaluation badly understates exposure, so defences and tests must assume a coordinated attacker that plans ahead, adapts across turns, and combines tactics.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **MAD-MAX — Modular And Diverse Malicious Attack MiXtures** | Schoepf et al. | 2025 — arXiv | mixture-of-attackers; automated red team | Keeps a modular attack-style library, uses LLM agents to cluster which styles work, then *merges* promising attacks each iteration to compound strengths; a similarity filter prunes near-duplicates. | [arXiv:2503.06253](https://arxiv.org/abs/2503.06253) |
| **X-Teaming** | Rahman et al. | 2025 — COLM | multi-agent, multi-turn | A Planner drafts and adapts attack plans, an Attacker runs the conversation, a Verifier scores progress, and an Optimizer rewrites when stuck — planning several turns ahead and recovering from refusals. | [arXiv:2504.13203](https://arxiv.org/abs/2504.13203) |
| **RedAgent** | Xu et al. | 2024 — arXiv | context-aware agent; strategy memory | Abstracts known jailbreaks into reusable "jailbreak strategies," selects and composes them per target, and uses a self-reflection memory buffer to learn from each attempt within a session. | [arXiv:2407.16667](https://arxiv.org/abs/2407.16667) |
| **AutoRedTeamer** | Andy Zhou, Kevin Wu, Francesco Pinto, Zhaorun Chen et al. | 2025 — NeurIPS | multi-agent; attacker-LLM search | A red-teaming agent works with a separate strategy-proposer agent; memory guides attack selection while the proposer incorporates newly described research strategies, turning risk categories into test cases. | [arXiv:2503.15754](https://arxiv.org/abs/2503.15754) |

---

## MAD-MAX — Modular And Diverse Malicious Attack MiXtures
*Schoepf et al. · 2025 — arXiv · `mixture-of-attackers; automated red team`*

MAD-MAX is the clearest realisation of the "mixture-of-attackers" idea. It maintains a modular **attack-style library** — a catalogue of distinct tactics kept as reusable components — and uses LLM agents to observe which styles are gaining traction against the target. Rather than committing to one style, it clusters the working approaches and treats them as ingredients, so the system reasons at the level of strategies rather than individual prompts.

The mechanism that sets it apart is explicit **merging**. Each iteration, promising attacks are combined so their individual strengths compound into a hybrid that can outperform either parent, a recombination step reminiscent of evolutionary crossover but operating over whole tactics and driven by agent judgement. To keep this affordable, a similarity filter prunes near-duplicate candidates before they consume queries, concentrating the budget on genuinely distinct lines of attack. The result is a search that is both diverse and efficient.

MAD-MAX reported large gains over Tree-of-Attacks-with-Pruning in both success rate and efficiency, positioning it as a step beyond earlier tree-structured search by adding tactic-level recombination (these figures are cited as published, not independently reproduced). The defensive lesson is that an adversary mixing and merging tactics presents a moving target: hardening against any one attack style is insufficient when the attacker can fuse several, so defences should be evaluated against composed, hybrid attacks rather than a fixed menu of known ones.

**Source:** [arXiv:2503.06253](https://arxiv.org/abs/2503.06253)

## X-Teaming
*Rahman et al. · 2025 — COLM · `multi-agent, multi-turn`*

X-Teaming is a collaborative agent pipeline built specifically for multi-turn attacks, where the danger accumulates across a conversation rather than in any single message. It assigns four roles: a **Planner** that drafts and adapts diverse attack plans, an **Attacker** that executes the resulting conversation, a **Verifier** that scores how much progress has been made, and an **Optimizer** that rewrites the approach when the attack stalls. Each role is a specialist, and the pipeline passes control between them as the dialogue unfolds.

The division of labour is what gives the system its reach. Because planning is separated from execution, X-Teaming can reason several turns ahead — setting up context early that only pays off later — and because verification and optimisation are explicit stages, it can detect a refusal, diagnose why the current line failed, and change strategy instead of abandoning the attempt. This resilience to refusals is precisely the capability that single-turn attacks lack.

Its lineage is the broader turn toward agentic, role-specialised red teaming, and it shares the planner/attacker/judge pattern with other systems in this chapter while emphasising multi-turn adaptation. Usefully for defenders, the same pipeline doubles as a generator of training data for defences, producing realistic multi-turn attack trajectories. The defensive lesson is that conversation-level safety cannot be assured by filtering messages one at a time: evaluation and guardrails must consider the whole dialogue, including slow-building context and recovery after an initial refusal.

**Source:** [arXiv:2504.13203](https://arxiv.org/abs/2504.13203)

## RedAgent
*Xu et al. · 2024 — arXiv · `context-aware agent; strategy memory`*

RedAgent confronts the sprawl of published jailbreaks by abstracting them into reusable **"jailbreak strategies"** — higher-level descriptions of *how* an attack works, lifted out of the specific prompts that first demonstrated it. This abstraction turns a disorganised zoo of individual exploits into a structured vocabulary of tactics that an agent can reason about and apply deliberately to a new target.

A multi-agent system then selects and composes these strategies for the specific target and context at hand, assembling context-aware attacks rather than replaying fixed prompts. The second key mechanism is a **self-reflection memory buffer**: after each attempt, the system records what happened and uses that memory to adjust its choices within the session, so it improves as a conversation or campaign progresses instead of treating every try as independent. Context-awareness plus in-session learning is what lets it find weaknesses efficiently.

RedAgent reportedly applies this to realistic deployments, including custom GPT-style applications built on top of a base model, where application-specific context creates its own openings. In the lineage it is a strategy-memory exemplar — a precursor to the library-plus-memory pattern that later orchestrators formalise. The defensive lesson is that attackers generalise: once an exploit is abstracted into a strategy, patching the one prompt that revealed it accomplishes little, and defences must address the underlying tactic. Application builders who add context on top of a model inherit, and can widen, its attack surface.

**Source:** [arXiv:2407.16667](https://arxiv.org/abs/2407.16667)

## AutoRedTeamer
*Andy Zhou, Kevin Wu, Francesco Pinto, Zhaorun Chen et al. · 2025 — NeurIPS · `multi-agent; attacker-LLM search`*

AutoRedTeamer pairs a red-teaming agent with a separate **strategy-proposer** agent, splitting the problem of *running* attacks from the problem of *sourcing* new ones. The red-team agent carries out attacks and uses memory to guide which approaches to try, while the proposer agent continually incorporates newly described research strategies — keeping the system's repertoire current as the published literature of attacks grows.

The mechanism's value is automation of the full loop. Memory-guided selection lets the attacker concentrate on what has worked, and the proposer feeds in fresh tactics so the system does not stagnate on a fixed set. It can also take high-level risk categories as input and turn them into concrete test cases, which makes it usable as a directed auditing tool: point it at a category of concern and it generates the probes. The two agents together form a self-updating red-team pipeline rather than a static attack script.

Its contribution — and the reason for the bookkeeping note that opens this chapter — is automated integration and orchestration of attacks, not any single new jailbreak; it should be counted once for its orchestration role, not re-counted for every attack it invokes. In the lineage it generalises the library-and-memory pattern into a continuously-updating system. The defensive lesson is that red teaming is becoming a self-maintaining pipeline that absorbs each newly published technique, so defences must be re-tested continuously against an attacker that keeps current, not certified once against a frozen snapshot.

**Source:** [arXiv:2503.15754](https://arxiv.org/abs/2503.15754)

---
[← 07 · Evolutionary & QD](07-evolutionary-and-quality-diversity.md) · [Index](../README.md) · [Next: 09 · Encoding & obfuscation →](09-encoding-and-obfuscation.md)
