# 12 · Prompt injection & system subversion

[← Back to index](../README.md)

> A cousin of jailbreaking: rather than coax disallowed content from a chat model, hijack an *application's* instructions — increasingly the dominant risk for agents and RAG systems.

The core unsolved problem of the agentic era: current models can't reliably separate trusted developer instructions from untrusted data in the same context window. The confidentiality and availability variants (prompt leaking, slowdown) live in [Chapter 20](20-confidentiality-and-availability.md); the tool/agent-specific forms in [Chapter 17](17-agent-tools-and-propagation.md).

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Direct Prompt Injection** | Community; coined by Simon Willison | 2022 | instruction override; app-layer | The user input itself tries to **override the developer's system prompt** — "ignore previous instructions and instead…"<br>Because models can't reliably tell trusted system instructions from untrusted user text in one context window, a well-placed override can repurpose the application.<br>It can leak the system prompt or change behaviour.<br>The original, still-unsolved framing that named the field (OWASP LLM01). | [Willison](https://simonwillison.net/2022/Sep/12/prompt-injection/) · [OWASP](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) |
| **Indirect Prompt Injection** | Kai Greshake et al. | 2023 | data-borne payload; agents / RAG | The malicious instruction isn't typed by the user — it's **planted in content the model will later read**: a web page, PDF, email, calendar invite, or code comment.<br>When an agent or RAG pipeline ingests that data, the hidden text executes as if it were a user command.<br>Enables data exfiltration, unauthorised tool calls, or manipulation — without the legitimate user doing anything.<br>The core security problem of the agentic era. | [arXiv:2302.12173](https://arxiv.org/abs/2302.12173) |
| **HouYi** | Yi Liu, Gelei Deng, Yuekang Li et al. | 2023 — arXiv | prompt-inject; agentic/application | A black-box framework for injection into real applications that wrap an LLM with task instructions.<br>Its design separates application-compatible content, context-boundary manipulation, and an attacker objective.<br>The application can interpret user-controlled material as a change to its governing task.<br>A specific published framework beyond generic direct injection; the failure is application instruction *integrity*. | [arXiv:2306.05499](https://arxiv.org/abs/2306.05499) |

---
[← 11 · Policy-format spoofing](11-policy-format-spoofing.md) · [Index](../README.md) · [Next: 13 · Visual & cross-modal →](13-visual-and-cross-modal.md)
