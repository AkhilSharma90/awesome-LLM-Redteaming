# Appendix A · Benchmarks, datasets & open tooling

[← Back to index](../README.md)

> How to *measure* attacks and defences on common ground — and the open frameworks that implement many of the methods in this guide.

A catalogue of attacks is only half the picture; reproducible evaluation is the other half. Note the recurring caveat from the evaluation literature: a benchmark or framework is **not** a new attack primitive, and you should not count it (or each transformation it bundles) as an additional method.

## Benchmarks & datasets

| Resource | What it is | Link |
| --- | --- | --- |
| **HarmBench** | A standardised evaluation framework for automated red teaming; compares attacks and defences on a common footing. | [site](https://www.harmbench.org/) |
| **JailbreakBench** | An open, versioned benchmark and leaderboard with a standard behaviour set and robust judging. | [arXiv:2404.01318](https://arxiv.org/abs/2404.01318) |
| **AdvBench** | The harmful-behaviours / harmful-strings set introduced with GCG; a common target list. | [GCG repo](https://github.com/llm-attacks/llm-attacks) |
| **"Do Anything Now" dataset** (Shen et al.) | A large real-world collection of in-the-wild jailbreak prompts scraped from communities. | [arXiv:2308.03825](https://arxiv.org/abs/2308.03825) |
| **StrongREJECT** | A benchmark and scoring rubric designed to avoid over-counting low-quality "successes". | [arXiv:2402.10260](https://arxiv.org/abs/2402.10260) |

## Evaluation frameworks (agentic & multimodal)

| Resource | Origin | Year / venue | What it is | Link |
| --- | --- | --- | --- | --- |
| **InjecAgent** | Qiusi Zhan, Zhixiang Liang, Zifan Ying, Daniel Kang | 2024 (Findings of ACL) | Places malicious content in tool outputs used for legitimate tasks; measures whether an agent performs attacker actions. Useful for repeatable indirect-injection evaluation. | [Findings of ACL](https://aclanthology.org/2024.findings-acl.624/) |
| **AgentDojo** | Debenedetti, Zhang, Balunović, Beurer-Kellner, Fischer, Tramèr | 2024 — NeurIPS | An extensible environment for tool-using agents and untrusted data; evaluates task completion *and* security, so "security" isn't achieved by disabling all useful behaviour. | [arXiv:2406.13352](https://arxiv.org/abs/2406.13352) |
| **MCPTox** | Zhiqiang Wang, Yichao Gao, Yanting Wang et al. | 2025; rev. 2026 | A tool-poisoning benchmark for the Model Context Protocol; evaluates influence *before* tool execution. Systematises the TPA surface. | [arXiv:2508.14925](https://arxiv.org/abs/2508.14925) |
| **Jailbreak-AudioBench** | Hao Cheng, Erjia Xiao, Jing Shao et al. | 2025 — NeurIPS D&B | An audio-jailbreak dataset + editing toolbox + benchmark; covers explicit and implicit adversarial audio beyond simple text-to-speech. | [NeurIPS 2025](https://proceedings.neurips.cc/paper_files/paper/2025/hash/0ff38d72a2e0aa6dbe42de83a17b2223-Abstract-Datasets_and_Benchmarks_Track.html) |

## Open red-team frameworks

| Tool | Origin | What it is | Link |
| --- | --- | --- | --- |
| **garak** | NVIDIA | An LLM vulnerability scanner with probes for many attacks above, including a GOAT probe. | [GitHub](https://github.com/NVIDIA/garak) |
| **PyRIT** | Microsoft | The Python Risk Identification Toolkit; ships orchestrators such as Crescendo and Context Compliance. | [GitHub](https://github.com/Azure/PyRIT) |
| **EasyJailbreak** | Community | A unified framework that re-implements many published attacks behind one interface. | [GitHub](https://github.com/EasyJailbreak/EasyJailbreak) |
| **promptfoo** | promptfoo | Developer-facing red-team / eval tooling with a catalogue of attack plugins. | [GitHub](https://github.com/promptfoo/promptfoo) |

## A note on defences

Each family has natural countermeasures worth a chapter of their own: perplexity and input filters (vs. GCG-style gibberish), paraphrasing and retokenisation, output classifiers and dual-model judging, system-prompt hardening and spotlighting, server-side conversation state (vs. CCA), context-window limits and demonstration detection (vs. many-shot), trust boundaries and content sandboxing for tools (vs. indirect injection), and adversarial training on red-team data (e.g. from Rainbow Teaming or X-Teaming). The honest caveat the research keeps repeating: **no single defence is complete, and many attacks transfer across models** — so defence-in-depth and continuous evaluation, not a one-time fix, are the state of the art.

---
[← Index](../README.md) · [B · Surveys →](appendix-B-surveys-and-taxonomies.md)
