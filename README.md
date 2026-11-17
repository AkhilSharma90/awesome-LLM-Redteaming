# Awesome LLM Red-Teaming

> A field guide to how large language models actually get broken — the methodologies, their mechanisms, their origins, and what they teach defenders.

This is a **defensive, educational** catalogue of published AI red-teaming and jailbreak *methodologies* — from the Crescendo multi-turn pattern and AutoDAN's evolutionary search to mixture-of-attacker swarms, multimodal attacks, RAG poisoning, and agent-memory backdoors. It collects **120+ named methods** across **21 mechanism families**, organised the way a textbook would, so that security teams, model builders, and researchers share a common vocabulary for how models fail.

Every entry is a **conceptual summary**: *how a class of attack works and why it bypasses alignment*, with its origin and a link to the primary source. It deliberately contains **no working jailbreak strings, payloads, or step-by-step recipes**. Everything here is already public — published in a paper or a vendor advisory — and cited so you can read the original.

---

## ⚠️ Scope & responsible use

This repository exists to help people **recognise, test for, and defend against** these attacks — to build evaluations, guardrails, and red-team programs. Use it only against systems you own or are explicitly authorised to test. It is not a how-to for attacking systems you don't control. Vendor-reported effectiveness figures are their own and are **not** independently reproduced here.

---

## How to read this

Jailbreaks are usually sliced along two axes:

- **By access** — *white-box* (gradients/weights available) vs. *black-box* (only the chat interface).
- **By granularity** — *token-level* (optimise characters no human would type) vs. *prompt-level* (fluent, human-readable manipulation).

On top of those sit **who drives the attack** (a human, an optimiser, an attacker-LLM, or a swarm of agents), **how many turns** it takes, and **which modality** it enters through (text, image, audio, retrieved documents, tool metadata, or training data). The chapters below group methods by their **core mechanism**, because that is how practitioners actually reason about defences.

New to the field? Start with **[Chapter 00 — Foundations](chapters/00-foundations.md)**: the two mental models (competing objectives & mismatched generalisation) explain almost everything that follows.

---

## Contents

### Part I — Why jailbreaks work
- **[00 · Foundations](chapters/00-foundations.md)** — the root-cause framings and the primitives inside every other attack

### Part II — Prompt-level & conversational jailbreaks
- **[01 · Role-play, persona & scenario nesting](chapters/01-roleplay-and-nesting.md)** — answer "in-world" as a character or nested fiction
- **[02 · Multi-turn & conversational escalation](chapters/02-multi-turn.md)** — never ask directly; walk the model there over many turns
- **[03 · Long-context & in-context conditioning](chapters/03-long-context.md)** — use the context window itself as the weapon

### Part III — Automated search & optimisation
- **[04 · Attacker-LLM automated search](chapters/04-attacker-llm-search.md)** — one LLM attacks another, no gradients needed
- **[05 · Gradient & token-level optimisation](chapters/05-gradient-and-token-level.md)** — optimise directly over tokens when you have the weights
- **[06 · Black-box brute force & augmentation](chapters/06-black-box-brute-force.md)** — no cleverness, just volume and adaptive search
- **[07 · Evolutionary & quality-diversity search](chapters/07-evolutionary-and-quality-diversity.md)** — evolve a diverse *population* of attacks
- **[08 · Multi-agent & mixture-of-attackers](chapters/08-multi-agent.md)** — orchestrate a team that plans, attacks, judges & recombines

### Part IV — Representation & semantics
- **[09 · Encoding, obfuscation & transformation](chapters/09-encoding-and-obfuscation.md)** — move the request out of natural-language space
- **[10 · Persuasion, framing & reasoning spoofing](chapters/10-persuasion-and-reasoning-spoofing.md)** — the rhetorical levers that work on people
- **[11 · Structural / policy-format spoofing](chapters/11-policy-format-spoofing.md)** — imitate the system's own control language

### Part V — Injection, agents & tools
- **[12 · Prompt injection & system subversion](chapters/12-prompt-injection.md)** — hijack an *application's* instructions
- **[16 · Agent memory & experience poisoning](chapters/16-agent-memory-poisoning.md)** — corrupt what an agent remembers
- **[17 · Agent tools, delegation & propagation](chapters/17-agent-tools-and-propagation.md)** — poison tool metadata; make attacks self-propagate

### Part VI — Multimodal
- **[13 · Visual & cross-modal attacks](chapters/13-visual-and-cross-modal.md)** — carry the attack in an image
- **[14 · Audio & speech attacks](chapters/14-audio-and-speech.md)** — carry it in sound

### Part VII — Data, decoding & model-level
- **[15 · Retrieval & knowledge-base poisoning](chapters/15-retrieval-poisoning.md)** — corrupt the evidence a RAG system retrieves
- **[18 · Decoding-time manipulation](chapters/18-decoding-time.md)** — attack how the model samples, not what you ask
- **[19 · Training, fine-tuning, model editing & activations](chapters/19-training-and-model-level.md)** — bake the backdoor into the weights
- **[20 · Confidentiality & availability attacks](chapters/20-confidentiality-and-availability.md)** — leak the prompt, extract training data, exhaust the budget

### Appendices
- **[A · Benchmarks, datasets & open tooling](chapters/appendix-A-benchmarks-and-tooling.md)** — how to measure attacks and the frameworks that implement them
- **[B · Surveys & taxonomies](chapters/appendix-B-surveys-and-taxonomies.md)** — the reading maps, including the NIST taxonomy
- **[C · Aliases, name collisions & deduplication notes](chapters/appendix-C-aliases-and-dedup.md)** — why AutoDAN is two papers, and other traps
- **[D · Unverified leads & watchlist](chapters/appendix-D-unverified-leads.md)** — surfaced but not yet confirmed; **not** counted as established methods

---

## Legend — mechanism tags

| Tag | Meaning |
| --- | --- |
| `foundational` | A root-cause framing or an early antecedent of a whole lineage |
| `attacker-LLM search` | An LLM generates/refines attacks against the target |
| `gradient / token-level` | White-box optimisation over tokens or activations |
| `evolutionary / QD` | Genetic / quality-diversity population search |
| `multi-agent / mixture` | Several coordinated agents or merged attack styles |
| `multi-turn` | The attack unfolds across a conversation |
| `long-context` | Exploits the size of the context window |
| `encoding / obfuscation` | Request moved into cipher, code, art, or another language |
| `persuasion` | Social-engineering / rhetorical framing |
| `policy-format spoofing` | Mimics config/policy/developer control text |
| `prompt-inject` | Hijacks application or agent instructions |
| `multimodal` | Enters via image or audio |
| `retrieval / corpus poisoning` | Corrupts retrieved documents (RAG) |
| `memory poisoning` | Corrupts an agent's persistent memory |
| `agentic / tool` | Targets tool use, delegation, or propagation |
| `decoding-time` | Manipulates the generation/sampling process |
| `data / weight poisoning` | Fine-tuning, model-editing, or pre-training backdoors |
| `confidentiality` / `availability` | Leakage or resource-exhaustion rather than refusal bypass |

---

## Provenance & verification status

Attributions (authors, titles, years, venues) are grounded in primary publication records where possible, and each entry links to its source.

- **Pre-2025 entries** are the most firmly established and are widely cross-referenced.
- **2025–2026 entries** are recent; treat their venue/identifier details as current-best and verify against the linked source before relying on them.
- **[Appendix D](chapters/appendix-D-unverified-leads.md)** holds leads whose primary pages could not be confirmed. They are *not* counted among the established methods and should be checked before being cited.
- **Vendor advisory claims** (e.g. attack-success rates) are vendor-reported; independent reproduction is not asserted here.

If you spot a wrong attribution, a broken link, or a better primary source, please open an issue or PR — see **[CONTRIBUTING.md](CONTRIBUTING.md)**.

---

## Contributing

Additions are welcome, with two rules: (1) the method must be **published** (paper or vendor advisory) and **cited**; (2) entries stay **conceptual** — mechanism and defensive lesson only, never an operational payload. New names should establish a genuinely new *mechanism* or *attacker capability*, not just rebrand an existing one. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Content is released under [**CC BY 4.0**](LICENSE) — share and adapt with attribution.

## Acknowledgements

Built from the published AI-safety and security literature and vendor advisories cited throughout. This is a living document; corrections make it better.
