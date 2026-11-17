# 14 · Audio & speech attacks

[← Back to index](../README.md)

> Carry the attack in sound. End-to-end audio-language models have their own safety gap — one that text-to-speech of an existing text jailbreak does *not* adequately reproduce.

When a model consumes speech directly rather than transcribing it first, its safety behaviour is governed by an end-to-end audio representation, not by the text pipeline a defender has already hardened. That distinction is the organising idea of this chapter. A jailbreak that a text filter catches on the page can behave differently once the same meaning arrives as a waveform, because the acoustic encoder and the language model were aligned — or not aligned — as a joint system. Simply rendering a known text attack through text-to-speech is therefore a poor proxy: it exercises the speech front end but tells you little about where the real gap lives.

The acoustic channel is also a second attack surface in its own right. Sound can be shaped so that a microphone and model interpret it as instructions while nearby people perceive little or nothing unusual, collapsing the gap between what a system "hears" and what a human reviewer would flag. That makes input authenticity — not just prompt content — a safety-relevant property of voice interfaces.

The most distinctive pattern in the recent literature is cross-modal **interference**: audio that carries no harmful instruction of its own can still change how a model enforces policy on a separate text request. Here the failure is not that a payload was smuggled past a filter but that one modality silently reshapes enforcement in another. The defensive takeaway across all of these is that safety evaluation for audio-language models must be conducted natively in the acoustic channel, across languages and conditions, and must treat instruction-neutral sound as a potential influence rather than assuming benign audio is inert.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **AudioJailbreak** | Guangke Chen, Fu Song, Zhe Zhao et al. | 2025; rev. 2026 (IEEE TDSC) | multimodal; acoustic perturbation | Adversarial audio influences an end-to-end audio-language model **without full control of the spoken user prompt**.<br>Studies perturbations that generalise across prompts and need not align with them in time.<br>Evaluates delivery through a real acoustic channel.<br>The safety gap is in the end-to-end audio representation — TTS of a text jailbreak is not an adequate substitute. | [arXiv:2505.14103](https://arxiv.org/abs/2505.14103) |
| **Multi-AudioJail** | Jaechul Roh, Virat Shejwalkar, Amir Houmansadr | 2025 — arXiv | multimodal; encoding; augmentation | Varies **language, accent, and acoustic conditions** together.<br>Measures how those interactions change audio-language models' refusal behaviour.<br>An audio interface can expose weaknesses absent from an English text-only evaluation.<br>Systematic cross-linguistic acoustic testing — not a reduction of the low-resource-language text attack. | [arXiv:2504.01094](https://arxiv.org/abs/2504.01094) |
| **Con Instruction** | Jiahui Geng, Thy Thy Tran, Preslav Nakov, Iryna Gurevych | 2025 — ACL | multimodal; cross-modal instruction injection | Adversarial images or audio convey instructions **without an explicit textual command**.<br>The model's ability to follow non-textual instructions becomes the attack channel.<br>Also examines interactions with accompanying benign text.<br>Safety checks must cover instruction semantics across modalities; this entry spans both visual and audio systems. | [ACL 2025](https://aclanthology.org/2025.acl-long.146/) |
| **SWhisper — Sirens' Whisper** | Zijian Ling, Pingyi Hu, Xiuyong Gao et al. | 2026 — USENIX Security | multimodal; acoustic-channel injection | Studies **covert acoustic delivery** of instructions to speech-driven models.<br>Microphone and channel behaviour can make signals machine-interpretable while hard for nearby people to perceive.<br>A voice-aware component adapts the intended instruction to the speech interface.<br>Combines an input-authenticity failure with model-level prompt vulnerability; unrelated to the multi-turn framework *Siren*. | [arXiv:2603.13847](https://arxiv.org/abs/2603.13847) · [USENIX](https://www.usenix.org/conference/usenixsecurity26/presentation/ling) |
| **TAGO — Token-Aware Gradient Optimization** | Zheng Fang, Xiaosen Wang, Shenyi Zhang et al. | 2026 — ICML | multimodal; gradient/token-level | Studies which **audio-token regions** contribute most to adversarial optimisation.<br>Concentrates changes in regions carrying stronger gradient signals.<br>Sparse modifications can retain effectiveness while leaving much of the waveform unchanged.<br>Challenges the assumption that effective perturbation attacks require dense modification of the whole recording. | [ICML 2026](https://proceedings.mlr.press/v306/fang26r.html) |
| **AIA — Acoustic Interference Attack** | Yanyun Wang, Yu Huang, Zi Liang, Xixin Wu, Li Liu | 2026 — ICML | multimodal; cross-modal interference | **Instruction-neutral audio** alters a model's response to a separate textual request.<br>Studies acoustic latent semantics rather than embedding the requested harmful content in speech.<br>The same interference audio can be reused across different instructions in the evaluated setting.<br>Distinguishing failure: an apparently benign modality changes enforcement on another modality. | [ICML 2026](https://proceedings.mlr.press/v306/wang26cf.html) |

---

## AudioJailbreak

*Adversarial sound that steers an end-to-end audio model without owning the spoken prompt.*

AudioJailbreak studies how perturbed audio can influence an end-to-end audio-language model's behaviour even when the attacker does not fully control the user's spoken request. Rather than crafting one bespoke clip per target utterance, the work examines perturbations that generalise across different prompts and that need not be aligned in time with the speech they accompany — a meaningfully stronger threat model than a single hand-tuned sample.

The mechanism matters because it locates the vulnerability precisely. The safety-relevant behaviour is a property of the joint audio representation the model learned, so an influence introduced in the acoustic domain can shift outcomes in ways a text-stage filter never sees. The authors also stress delivery through a real acoustic channel rather than idealised digital injection, which is why rendering a text jailbreak through text-to-speech is not an adequate substitute: TTS reproduces the words, not the representational gap the attack exploits.

For defenders, the lesson is that audio safety cannot be inherited from text safety. Evaluation has to probe the end-to-end model under realistic, prompt-independent acoustic conditions, and robustness claims proven only on transcribed text do not transfer. The revised IEEE TDSC version situates this as a systematic account of the audio-representation gap rather than a one-off demonstration.

**Source:** [arXiv:2505.14103](https://arxiv.org/abs/2505.14103)

## Multi-AudioJail

*Refusal behaviour that shifts when language, accent, and acoustics are varied together.*

Multi-AudioJail examines how audio-language models respond when several input dimensions — language, accent, and acoustic conditions — are varied jointly rather than one at a time. The central observation is that these factors interact: a combination of conditions can change refusal behaviour in ways that none of the factors predicts alone, so an evaluation that holds language or channel fixed can miss the gap entirely.

The mechanism is less a single payload than a mapping exercise. By sweeping conditions together, the work shows that an audio interface can surface weaknesses that an English, text-only safety evaluation would never reveal, because the model's enforcement is uneven across the space of ways the same request can sound. This is why the authors frame it as systematic cross-linguistic acoustic testing and distinguish it from a simple audio restatement of the known low-resource-language text attack — the interacting acoustic dimensions are the point, not merely the choice of language.

The defensive takeaway is coverage. If safety is validated only in clean English speech, a voice product can ship with blind spots that appear under ordinary real-world variation in accent, language, and recording conditions. Robustness evaluation for speech systems needs to span that variation deliberately.

**Source:** [arXiv:2504.01094](https://arxiv.org/abs/2504.01094)

## Con Instruction

*Instructions delivered through image or audio rather than words.*

Con Instruction investigates how adversarial images or audio can convey instructions to a multimodal model without any explicit textual command. The model's own competence — its ability to interpret and act on non-textual input — becomes the channel through which an instruction arrives, so a system that scans only the text prompt has nothing to flag.

The mechanism turns instruction-following into a liability. Because the model treats a non-textual modality as a legitimate source of intent, meaning can be carried in that modality and paired with accompanying benign text that does not itself trip a filter. The work also examines how such non-textual instructions interact with the benign text alongside them, showing that the combination, not either part in isolation, drives behaviour. This entry spans both visual and audio systems, which is why it sits at the boundary with the visual and cross-modal chapter.

The defensive lesson is that safety checks must reason about instruction *semantics* across every modality a model accepts, not just the literal text field. Any input the model can interpret as a directive is part of the attack surface, and policy enforcement that lives only in the text pipeline will not see instructions that enter through sound or images.

**Source:** [ACL 2025](https://aclanthology.org/2025.acl-long.146/)

## SWhisper — Sirens' Whisper

*Covert acoustic delivery that a model interprets but a bystander may not notice.*

SWhisper (Sirens' Whisper) studies covert acoustic delivery of instructions to speech-driven models: conditions under which a signal is machine-interpretable yet hard for nearby people to perceive. The work leans on how microphones and acoustic channels behave, so that what the system "hears" can diverge from what a human in the room would register — an input-authenticity failure rather than a purely semantic one.

The mechanism combines two weaknesses. A voice-aware component adapts the intended instruction to the particular speech interface, and the acoustic channel supplies the covert delivery, so the attack pairs a front-end authenticity gap with the underlying model's susceptibility to the instruction once it is received. Neither alone is sufficient; the covert channel gets the instruction in, and the model's prompt vulnerability lets it take effect. It is worth noting explicitly that this work is unrelated to *Siren*, the multi-turn jailbreak framework discussed elsewhere; the similar name is coincidental.

For defenders, SWhisper argues that voice systems cannot assume their audio input is authentic or human-audible. Guardrails that only inspect the recognised instruction miss the possibility that the instruction was never meant to be heard by a person at all, which motivates channel-level authentication for speech interfaces.

**Source:** [arXiv:2603.13847](https://arxiv.org/abs/2603.13847) · [USENIX](https://www.usenix.org/conference/usenixsecurity26/presentation/ling)

## TAGO — Token-Aware Gradient Optimization

*Perturbation concentrated where audio tokens carry the strongest signal.*

TAGO (Token-Aware Gradient Optimization) studies which regions of an audio input contribute most to adversarial optimisation. Instead of treating the whole waveform as equally important, it identifies audio-token regions that carry stronger gradient signal and concentrates modification there, leaving much of the recording unchanged.

The mechanism's significance is what it reveals about the attack surface rather than any recipe. The finding that sparse, localised modification can retain effectiveness challenges a common defensive assumption — that meaningful perturbation attacks require dense alteration of the entire recording, and that a mostly-unchanged clip is therefore safe. If influence concentrates in a small set of high-signal regions, detectors tuned to broad waveform distortion may overlook it. This places TAGO alongside AudioJailbreak and TAGO's gradient-level peers as evidence about *where* in the signal audio vulnerability lives.

The defensive lesson is that robustness checks for speech models should not presume that adversarial influence is diffuse or energetically large. A nearly intact recording is not evidence of safety, and audio-aware defences need to account for concentrated, token-localised modification.

**Source:** [ICML 2026](https://proceedings.mlr.press/v306/fang26r.html)

## AIA — Acoustic Interference Attack

*Benign-sounding audio that changes how the model answers a separate text request.*

The Acoustic Interference Attack (AIA) is the chapter's clearest example of cross-modal interference. Instruction-neutral audio — sound that carries no harmful content of its own — is shown to alter a model's response to a separate textual request. The attack does not embed the requested behaviour in speech; it studies how acoustic latent semantics can shift enforcement elsewhere.

The mechanism is what makes AIA distinctive. Because the influence lives in the audio's latent representation rather than in any spoken instruction, the same interference audio can, in the evaluated setting, be reused across different textual instructions. The failure is not a smuggled payload but a modality that appears benign and nonetheless changes how the model polices another modality, which is precisely the pattern a content-inspection filter is not built to catch.

For defenders this is the sharpest takeaway in the chapter: audio accompanying a request cannot be assumed inert simply because it contains no instruction. Enforcement that depends on the text alone can be moved by sound that would pass any content check, so safety evaluation must consider whether instruction-neutral inputs in one modality can reshape behaviour in another.

**Source:** [ICML 2026](https://proceedings.mlr.press/v306/wang26cf.html)

---
[← 13 · Visual & cross-modal](13-visual-and-cross-modal.md) · [Index](../README.md) · [Next: 15 · Retrieval poisoning →](15-retrieval-poisoning.md)
