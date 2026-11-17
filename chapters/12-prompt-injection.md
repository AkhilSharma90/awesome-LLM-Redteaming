# 12 · Prompt injection & system subversion

[← Back to index](../README.md)

> A cousin of jailbreaking: rather than coax disallowed content from a chat model, hijack an *application's* instructions — increasingly the dominant risk for agents and RAG systems.

Jailbreaking and prompt injection are often conflated, but they target different things. A jailbreak tries to make a model emit content its policy forbids. Prompt injection tries to make an *application built on* a model do something its developer never intended — leak its own configuration, call a tool it should not, act on instructions that came from the wrong source. The target is the integrity of the application's instructions, and as more software is built as LLM-driven agents and retrieval pipelines, this has become the more consequential class of failure.

At the root is a single unsolved problem: a current model reads everything in its context window as one undifferentiated stream of tokens. The developer's trusted system instructions, the user's request, and whatever data the application has pulled in from the outside world all arrive as text, and the model has no reliable, built-in mechanism to tell which is authoritative and which is merely content to be processed. There is no equivalent of the separation between code and data that decades of systems security were built on; instructions and data share one channel. Every attack in this chapter is a different way of exploiting that collapsed boundary.

Because the boundary failure is architectural rather than a matter of any single bad string, it cannot be patched away by filtering inputs — the dangerous instruction may be perfectly benign-looking text drawn from a trusted-seeming source. The defensive posture has to shift to the application level: treat all retrieved and user-supplied content as untrusted, constrain what tools and data an agent can reach, require confirmation for consequential actions, and design on the assumption that injected instructions will sometimes get through. The confidentiality and availability variants (prompt leaking, slowdown) live in [Chapter 20](20-confidentiality-and-availability.md); the tool/agent-specific forms in [Chapter 17](17-agent-tools-and-propagation.md).

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Direct Prompt Injection** | Community; coined by Simon Willison | 2022 | instruction override; app-layer | User input tries to override the developer's system prompt and repurpose the application. | [Willison](https://simonwillison.net/2022/Sep/12/prompt-injection/) · [OWASP](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) |
| **Indirect Prompt Injection** | Kai Greshake et al. | 2023 | data-borne payload; agents / RAG | A malicious instruction is planted in content the model will later ingest — a page, PDF, email, or code comment — and executes when read. | [arXiv:2302.12173](https://arxiv.org/abs/2302.12173) |
| **HouYi** | Yi Liu, Gelei Deng, Yuekang Li et al. | 2023 — arXiv | prompt-inject; agentic/application | A black-box framework for injecting into real LLM-backed applications, separating compatible content, boundary manipulation, and attacker objective. | [arXiv:2306.05499](https://arxiv.org/abs/2306.05499) |

---

## Direct Prompt Injection
*The original framing: user text that tries to overrule the developer's instructions in the same window.*

Direct prompt injection is the attack that named the field. The user's own input attempts to override the developer's system prompt — the familiar shape is an instruction telling the model to disregard whatever it was previously told and follow the user instead. Because the system prompt and the user input occupy the same context window with no enforced hierarchy between them, a well-placed override can repurpose the application: making it ignore its intended task, reveal its system prompt, or adopt new behaviour the developer never sanctioned.

The mechanism is the collapsed trust boundary described above, in its simplest form. The model cannot reliably distinguish the developer's authoritative instructions from the user's untrusted text when both are plain language in one stream; whichever the model attends to most strongly wins, and a direct instruction to switch allegiance is often enough. There is no decoding or obfuscation involved — the attack is entirely in the pragmatics of who the model takes its orders from.

Simon Willison coined the term in 2022, drawing the analogy to SQL injection: data crossing into the instruction channel. It is catalogued as LLM01, the top entry in the OWASP Top 10 for LLM applications, and years later it remains unsolved at the model level — mitigations reduce its likelihood but do not close the boundary. The defensive lesson is foundational for everything built on LLMs: never assume user input will stay in its lane, and never place anything in a system prompt whose disclosure or override would be harmful.

**Source:** [Willison](https://simonwillison.net/2022/Sep/12/prompt-injection/) · [OWASP](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)

## Indirect Prompt Injection
*The dangerous instruction isn't typed by the user — it's waiting in the data the model will read.*

Indirect prompt injection removes the attacker from the conversation entirely. Instead of typing an override, the attacker plants a malicious instruction in content that the model will later ingest: a web page, a PDF, an email, a calendar invite, a code comment, any data an application might retrieve and feed into the context. When an agent or a retrieval-augmented pipeline pulls that content in, the hidden text is read with the same undifferentiated authority as a genuine user command, and executes as if the legitimate user had asked for it.

This is the same collapsed boundary as direct injection, but with a far more dangerous delivery path. The legitimate user does nothing wrong and may be entirely unaware; the payload rides in on data the system was designed to consume. That opens the door to data exfiltration, unauthorised tool calls, and silent manipulation of the agent's behaviour, triggered by nothing more than the application doing its job of reading external content.

Greshake and colleagues formalised the threat in 2023, and it has become the defining security problem of the agentic era precisely because agents are built to act on untrusted external data at scale. The defensive implications are architectural and are developed further in [Chapter 17](17-agent-tools-and-propagation.md): every piece of retrieved content must be treated as potentially hostile, an agent's authority to call tools and reach data must be tightly scoped, and consequential actions should require a check that cannot itself be driven by the injected text.

**Source:** [arXiv:2302.12173](https://arxiv.org/abs/2302.12173)

## HouYi
*A structured, black-box framework for injecting into real LLM-backed applications.*

HouYi studies prompt injection not as a one-off clever string but as a repeatable framework for attacking real applications that wrap an LLM around a fixed task — the kind of product that takes user input, combines it with a developer prompt, and returns a result. It treats such an application as a black box and systematises how to subvert it, generalising beyond the generic "ignore previous instructions" move into a reusable methodology.

Its design decomposes an injection into distinct parts: content that is compatible with what the application legitimately expects, a manipulation of the boundary between the application's context and the injected material, and the attacker's actual objective. Separating these concerns is what lets the approach adapt across different applications rather than depending on one target's quirks — the compatible framing gets the payload accepted, the boundary manipulation shifts how the application reads it, and the objective rides on top. The failure it exercises is application instruction *integrity*: user-controlled material is interpreted as a change to the governing task.

As a named, published framework, HouYi marks the maturing of prompt injection from an observation into an engineering discipline, demonstrating that injection against deployed LLM applications could be approached methodically and with reasonable generality. For defenders it reinforces the chapter's throughline: the vulnerability lives at the application layer, in the fusion of trusted task instructions with untrusted input, and defences must protect the integrity of that task rather than hoping to filter out every hostile phrasing.

**Source:** [arXiv:2306.05499](https://arxiv.org/abs/2306.05499)

---
[← 11 · Policy-format spoofing](11-policy-format-spoofing.md) · [Index](../README.md) · [Next: 13 · Visual & cross-modal →](13-visual-and-cross-modal.md)
