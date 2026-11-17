# Appendix C · Aliases, name collisions & deduplication notes

[← Back to index](../README.md)

> Several distinct methods share names, and several names are really one evolving paper. This appendix keeps the lineage straight so the catalogue doesn't double-count — or wrongly merge — them.

Names in this field are noisy. Two independent teams reach for the same memorable label (there are two unrelated "AutoDAN"s), a single project is renamed between preprint and publication (Siege became Tempest), and a cluster of similar-sounding agent attacks turn out to assume very different attacker access. Left unexamined, these collisions corrupt a catalogue in both directions: they inflate the method count by listing one evolving paper twice, and they erase real distinctions by merging two different contributions under a shared name. Getting the lineage right is therefore not pedantry — it is what makes the count trustworthy.

Read each row as a ruling. The first column names the collision or ambiguity; the middle column gives the merge guidance — whether the entries are genuinely distinct methods, variants nested under a parent, or one work under two names, and which label the catalogue treats as canonical; the final column anchors that ruling to primary sources so you can check it yourself. Every row is retained deliberately, including the ones that resolve to "keep separate," because the record of *why* two things were not merged is as useful as the merges themselves. The notes after the table collect patterns the catalogue deliberately declines to promote to standalone entries, and the design rule that governs how the catalogue grows.

| Name / issue | Merge guidance | Primary grounding |
| --- | --- | --- |
| **AutoDAN name collision** | Xiaogeng Liu et al.'s AutoDAN uses hierarchical **genetic** search (→ [Ch. 07](07-evolutionary-and-quality-diversity.md)). Sicheng Zhu et al.'s independently named AutoDAN uses **gradients** and readability objectives (→ [Ch. 05](05-gradient-and-token-level.md)). Two different papers. | [Liu, 2310.04451](https://arxiv.org/abs/2310.04451) · [Zhu, 2310.15140](https://arxiv.org/abs/2310.15140) |
| **AutoDAN-Turbo / AutoDAN-Reasoning** | One research lineage, separate publication records. Reasoning adds test-time search over strategies; it is not a second independent Best-of-N discovery. | [Turbo](https://arxiv.org/abs/2410.05295) · [Reasoning](https://arxiv.org/abs/2510.05379) |
| **AmpleGCG / AmpleGCG-Plus; I-GCG / Faster-GCG / MAGIC** | Published variants/extensions, not aliases. Nest under GCG optimisation. PAL's GCG++ and RAL are acknowledged inside PAL's entry, not counted again. | [AmpleGCG](https://arxiv.org/abs/2404.07921) · [Plus](https://arxiv.org/abs/2410.22143) · [PAL](https://arxiv.org/abs/2402.09674) |
| **Siege / Tempest** | One evolving paper with a changed name. Keep **Tempest** as the current label, **Siege** as an alias. | [arXiv:2503.10619](https://arxiv.org/abs/2503.10619) |
| **Echo Chamber / EchoLeak** | **Echo Chamber** is NeuralTrust's conversational jailbreak ([Ch. 02](02-multi-turn.md)). **EchoLeak** is Aim Labs' 2025 application exploit, CVE-2025-32711 ([Ch. 17](17-agent-tools-and-propagation.md)). Unrelated names, not variants. | [Echo Chamber](https://arxiv.org/abs/2601.05742) · [EchoLeak](https://www.catonetworks.com/blog/breaking-down-echoleak/) |
| **Tool Poisoning / Line Jumping** | Trail of Bits relates its "line jumping" label to tool poisoning. Rug pulls and tool shadowing are separately named subpatterns within that surface. | [Trail of Bits](https://blog.trailofbits.com/2025/04/21/jumping-the-line-how-mcp-servers-can-attack-you-before-you-ever-use-them/) · [Invariant](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks) |
| **FigStep / Visual Adversarial Examples** | FigStep (Gong et al., 2023 → AAAI 2025) uses image **text**. Qi et al.'s visual adversarial-example work is a different 2023 paper using **pixel** perturbations. | [FigStep](https://arxiv.org/abs/2311.05608) · [Qi et al.](https://arxiv.org/abs/2306.13213) |
| **Siren / Sirens' Whisper** | **Siren** is a learned multi-turn attacker ([Ch. 02](02-multi-turn.md)). **SWhisper** is an acoustic-channel attack ([Ch. 14](14-audio-and-speech.md)). Neither is an alias of AudioJailbreak. | [Siren](https://arxiv.org/abs/2501.14250) · [SWhisper](https://arxiv.org/abs/2603.13847) |
| **Cognitive Overload** | Xu et al. (2023/2024) and Upadhayay et al. (2024, long-context) are different publications with related terminology. The cognitive analogy is not established neuroscience of LLMs. | [Xu et al.](https://arxiv.org/abs/2311.09827) · [Upadhayay et al.](https://arxiv.org/abs/2410.11272) |
| **AgentPoison / BadAgent / BadChain / MINJA** | Respectively: retrieval-triggered demonstration poisoning; fine-tuned agent backdoor; poisoned in-context reasoning demonstrations; query-induced memory poisoning. Similar names, materially different attacker access. | [AgentPoison](https://arxiv.org/abs/2407.12784) · [BadAgent](https://arxiv.org/abs/2406.03007) · [BadChain](https://arxiv.org/abs/2401.12242) · [MINJA](https://arxiv.org/abs/2503.03704) |
| **Refusal-direction ablation / abliteration** | Cite Arditi et al. for the demonstrated mechanism. "Abliteration" is a broader community label and does not guarantee a specific implementation preserves capabilities or removes every safety mechanism. | [Arditi et al.](https://arxiv.org/abs/2406.11717) |
| **Virtual Prompt Injection / "virtualization"** | VPI is a **training-time** backdoor. "Virtualization" may instead mean virtual-machine role-play or programmatic encoding — a different thing. | [VPI](https://arxiv.org/abs/2307.16888) |

## Community patterns deliberately *not* promoted to standalone entries

- **Grandma exploit** — a recognisable role-play motif (a trusted/emotionally salient persona). First-attribution and canonical source unverified; belongs as an example under [Ch. 01](01-roleplay-and-nesting.md) unless a primary origin is recovered.
- **DAN descendants & "developer mode" variants** — a new persona *name* is not automatically a new methodology. A standalone entry should establish a new mechanism, a published search procedure, or a clearly sourced historical contribution.
- **"Token smuggling," "virtualization," encoded program execution** — labels used inconsistently; some instances are already covered by the Kang et al. payload-splitting entry. Distinguish from VPI before merging.

> **Design rule for growth:** prioritise a new *trust boundary* or *attacker capability* over a new attack name.

---
[← B · Surveys](appendix-B-surveys-and-taxonomies.md) · [Index](../README.md) · [D · Unverified leads →](appendix-D-unverified-leads.md)
