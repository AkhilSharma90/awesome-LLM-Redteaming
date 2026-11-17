# 16 · Agent memory & experience poisoning

[← Back to index](../README.md)

> Corrupt what an agent *remembers*. Persistence comes from the agent's own memory-writing and experience-reuse — not from a continuously present external prompt.

An agent that keeps long-term memory — a store of past interactions, retrieved demonstrations, or logged "successful" experiences it consults on later tasks — has a trust surface that sits apart from both its weights and its live context window. The weights are fixed at training time; the context is present only for the current turn. Memory is different: it is written by the agent as it runs and read back later, which means a corruption introduced once can resurface across many future sessions without the attacker being present again.

That property reframes what persistence costs an attacker. Classic prompt injection has to be present in the context to have effect; remove the malicious text and the behaviour stops. A memory poison, by contrast, is latent and durable. The agent's own memory-writing machinery is what makes it stick, and its own retrieval machinery is what reactivates it. The most concerning variants in this chapter never require the attacker to edit storage directly at all — ordinary interaction is enough to induce the records that later recall will surface.

The three methods below are best read as a ladder of attacker privilege, and keeping them distinct matters. AgentPoison assumes the adversary can place optimised demonstrations into the memory or knowledge base; MINJA lowers the bar sharply, inducing poisoned records purely through queries and observed responses, as any ordinary user could; MemoryGraft targets the experience store specifically, exploiting the agent's trust in its own record of past success. The common thread is trust in remembered material — demonstrations, reasoning traces, or procedures — that was never independently verified. Everything here is described conceptually, without trigger constructions or reusable poisoning procedures. Compare the retrieval-triggered and fine-tuned-agent backdoors in [Appendix C](appendix-C-aliases-and-dedup.md).

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **AgentPoison** | Zhaorun Chen, Zhen Xiang, Chaowei Xiao, Dawn Song, Bo Li | 2024 — NeurIPS | agentic/tool; memory poisoning | An attacker poisons an agent's long-term memory or knowledge base with malicious demonstrations; optimised triggers cause associated queries to retrieve them. Untriggered activity can stay largely unaffected. Implemented through retrieval and in-context behaviour, not fine-tuning. | [arXiv:2407.12784](https://arxiv.org/abs/2407.12784) |
| **MINJA — Memory INJection Attack** | Shen Dong, Shaochen Xu, Pengfei He et al. | 2025; rev. 2026 | agentic/tool; memory poisoning | The attacker interacts through queries and observed responses rather than editing storage; interactions induce records linking later benign queries to malicious reasoning. The agent's own memory-writing makes them persist — ordinary interaction as the entry point. | [arXiv:2503.03704](https://arxiv.org/abs/2503.03704) |
| **MemoryGraft** | Saksham Sahai Srivastava, Haoyu He | 2025 — arXiv | agentic/tool; experience-memory poisoning | Malicious material is incorporated into an agent's store of apparently successful past experiences; later tasks retrieve them as useful procedural examples and reproduce the unsafe behaviour. The failure is trust in remembered success. | [arXiv:2512.16962](https://arxiv.org/abs/2512.16962) |

---

## AgentPoison

*A retrieval-triggered backdoor planted in an agent's memory or knowledge base, dormant until a matching query arrives.*

AgentPoison poisons the long-term memory or knowledge base that a retrieval-augmented agent draws on for in-context demonstrations. The attacker contributes malicious demonstrations and optimises trigger conditions so that queries carrying the trigger retrieve those poisoned records, while ordinary queries continue to retrieve benign ones. Because the agent conditions its behaviour on the examples it retrieves, a retrieved malicious demonstration steers the current task — without any change to the model's parameters.

The mechanism works because retrieval-augmented agents treat their memory as a source of behavioural guidance, not merely reference facts. A demonstration that is retrieved shapes how the agent acts on the present input, so controlling what gets retrieved is a way of controlling behaviour. Implementing the backdoor through retrieval and in-context conditioning, rather than fine-tuning, keeps it entirely outside the weights and makes it invisible to inspection of the model itself.

In the chapter's ladder, AgentPoison sits at the higher-privilege end: it presumes the attacker can introduce demonstrations into the store. Its contribution is to show that the memory and knowledge base are a trust surface in their own right, and that trigger-conditioning lets a poison lie dormant through ordinary use — so clean behaviour under normal traffic is not evidence of a clean memory. Defensively, the demonstrations an agent retrieves deserve the same provenance and integrity scrutiny as any other untrusted input, and triggered-retrieval anomalies are worth monitoring.

**Source:** [arXiv:2407.12784](https://arxiv.org/abs/2407.12784)

## MINJA — Memory INJection Attack

*Poison an agent's memory from the outside of the keyhole: through ordinary queries and the responses they produce, with no access to storage.*

MINJA lowers the privilege an attacker needs to poison agent memory. Instead of writing to the memory store, the adversary interacts with the agent only through queries and observed responses — the same channel available to any ordinary user. Those interactions are structured so that the agent's normal memory-writing process records entries which later link benign-looking queries to malicious reasoning patterns. When such a query arrives in a future session, the agent recalls the induced record and follows the planted reasoning.

What makes this work, and makes it notable, is that the attack rides the agent's own machinery. The memory-writing step the agent performs as a matter of design is what persists the poison; the retrieval step reactivates it. The attacker never touches storage directly, so defences premised on protecting the memory backend do not address the entry point, which is ordinary interaction.

Relative to AgentPoison, MINJA is the lower-privilege neighbour: both end in poisoned memory that surfaces for later queries, but MINJA removes the assumption of direct write access, turning an everyday interaction into a seed for future recalled context. That is the more alarming threat model precisely because it requires so little. The defensive implication is that anything an agent writes to long-term memory as a result of untrusted interaction should be treated as untrusted when later read back, not promoted to trusted history simply because the agent itself recorded it.

**Source:** [arXiv:2503.03704](https://arxiv.org/abs/2503.03704)

## MemoryGraft

*Grafts malicious procedure into an agent's log of successful past experiences, so it is later reused as a template worth imitating.*

MemoryGraft targets a specific and increasingly common component: an agent's store of past experiences it deems successful and consults as procedural templates for new tasks. Malicious material is incorporated into that store as if it were one of those successful experiences. On a later, related task the agent retrieves the grafted experience as a useful example, imitates the procedure it encodes, and can thereby reproduce the unsafe behaviour embedded within it.

The vulnerability is trust in remembered success. Experience-replay designs exist precisely so an agent can reuse what worked before instead of re-deriving it; that reuse is a feature. But the agent's judgement of what "worked" is itself attackable, and once a harmful procedure is filed under successful experiences it carries the authority of a proven method. Persistence here comes from experience reuse rather than from any prompt that remains present — remove the original interaction and the grafted experience still sits in the store, ready to be retrieved.

Among the three methods, MemoryGraft is distinguished by what it poisons — the experience/procedure memory specifically — and by the kind of trust it subverts: not just retrieved facts or demonstrations but the agent's own record of past accomplishment. The defensive lesson generalises the chapter: experience stores need validation before reuse, and an entry's presence in a "successful" log should not by itself confer trust, since the label can be manufactured.

**Source:** [arXiv:2512.16962](https://arxiv.org/abs/2512.16962)

---
[← 15 · Retrieval poisoning](15-retrieval-poisoning.md) · [Index](../README.md) · [Next: 17 · Agent tools & propagation →](17-agent-tools-and-propagation.md)
