# 13 · Visual & cross-modal attacks

[← Back to index](../README.md)

> Carry the attack in an image. A continuous visual input is a high-dimensional channel for steering a model that was safety-aligned mostly on text.

Most contemporary safety alignment is built on text. The refusal behaviour of a chat assistant is shaped by reinforcement and preference data expressed in words, and it learns to recognise the textual surface of a disallowed request. When engineers connect a vision encoder to that aligned language model, they tend to assume the resulting system inherits the text model's judgement. It does not. Robustness is not a property that transfers across the modality boundary. The vision encoder was trained for perception, not for policy, and the fused model has rarely seen adversarial multimodal inputs during alignment. The image channel therefore arrives largely ungoverned.

A continuous visual input is also a vastly richer channel than a text prompt. A single image is hundreds of thousands of real-valued pixels, a near-unbounded space in which to search for an input that nudges the language component toward an attacker's goal. Where a text attacker must phrase intent in discrete tokens the safety training has probably seen, an image attacker optimises in a smooth, high-dimensional space the alignment process never explored. This is why so many of the attacks in this chapter read as optimisation problems rather than clever wordings.

The attacks fall into three broad routes. The first is **pixel perturbation**: an image is optimised, often by gradient methods against the model's own representations, so that an otherwise innocuous picture steers generation. The second is **typographic text-in-image**: the disallowed content is simply rendered as text inside a picture, exploiting the fact that the model reads and acts on visual text through a path its text-only refusals do not fully cover. The third is **compositional splitting**, where intent is divided across modalities so that neither the image nor the accompanying prompt is individually objectionable, but their fusion is. Several attacks combine routes, and a few extend the target beyond chat assistants to image *generators*.

The defensive takeaway is consistent across all three. Safety has to be evaluated and trained on the fused, multimodal system under adversarial conditions, not assumed from the language model it was built on. Untrusted media must be treated as a potential instruction-bearing channel, and integration layers where modalities merge are themselves part of the attack surface.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **Typographic Attacks on CLIP** | OpenAI (Multimodal Neurons research) | 2021 — research disclosure | multimodal; foundational | Text placed within an image can dominate the model's interpretation of the depicted object; exploits overlap between visual and textual concept representations. Original demonstrations concern classification, not a chat refusal policy. | [OpenAI](https://openai.com/index/multimodal-neurons/) |
| **Visual Adversarial Examples** | Xiangyu Qi, Kaixuan Huang, Ashwinee Panda, Peter Henderson, Mengdi Wang, Prateek Mittal | 2023 — arXiv | multimodal; gradient/token-level | An adversarial image is optimised to influence an aligned VLM's behaviour; one image can affect responses to multiple requests. Text-only alignment does not ensure robustness once vision is connected. | [arXiv:2306.13213](https://arxiv.org/abs/2306.13213) |
| **FigStep** | Yichen Gong, Delong Ran, Jinyuan Liu et al. | 2023 (AAAI 2025 Oral) | multimodal; typographic | Text for a disallowed task is carried in an image and coupled with a completion-oriented request. The visual route can behave differently from the plain-text request; attributed to weak alignment across visual embeddings. | [arXiv:2311.05608](https://arxiv.org/abs/2311.05608) |
| **HADES** | Yifan Li, Hangyu Guo, Kun Zhou, Wayne Xin Zhao, Ji-Rong Wen | 2024 — ECCV Oral | multimodal; semantic concealment | Redistributes harmful meaning between text and constructed images; visual material conceals and reinforces the unsafe interpretation. Coordinated image-text manipulation, not arbitrary images attached to a prompt. | [arXiv:2403.09792](https://arxiv.org/abs/2403.09792) |
| **Jailbreak in Pieces** | Erfan Shayegani, Yue Dong, Nael Abu-Ghazaleh | 2023 (ICLR 2024) | multimodal; embedding-space optimisation | An adversarial image supplies harmful context while the text request stays generic; targets the vision encoder's embedding behaviour and needs no access to the LLM. Central failure mode is composition. | [arXiv:2307.14539](https://arxiv.org/abs/2307.14539) |
| **Image Hijacks** | Luke Bailey, Euan Ong, Stuart Russell, Scott Emmons | 2023 — arXiv | multimodal; behavioural optimisation | An image is optimised to make a model behave like a chosen target; "Prompt Matching" emulates a textual instruction. Outcomes include output control, context leakage, and overriding safety — a runtime behavioural channel. | [arXiv:2309.00236](https://arxiv.org/abs/2309.00236) |
| **Abusing Images and Sounds** | Eugene Bagdasaryan, Tsung-Yin Hsieh, Ben Nassi, Vitaly Shmatikov | 2023 — arXiv | prompt-inject; multimodal | External image or recording carries a perturbation encoding an attacker instruction; a benign user has the model process it during an ordinary task. The crossed boundary is untrusted media vs. governing instructions. | [arXiv:2307.10490](https://arxiv.org/abs/2307.10490) |
| **CrossMPI — Cross-Modal Prompt Injection** | Hao Yang, Zhuo Ma, Yang Liu et al. | 2026 — arXiv | prompt-inject; multimodal | An image-only perturbation targets interpretation of both image and text; studies hidden states of multimodal integration and concentrates on selected integration layers and regions. Controlling one modality redirects another. | [arXiv:2605.16090](https://arxiv.org/abs/2605.16090) |
| **SneakyPrompt** | Yuchen Yang, Bo Hui, Haolin Yuan, Neil Gong, Yinzhi Cao | 2023 (IEEE S&P 2024) | multimodal; RL search | Searches text prompts for a guarded text-to-image generator; feedback from generated outputs guides prompt changes, probing gaps between filter and generator. Attacks an image generator, not a conversational VLM. | [arXiv:2305.12082](https://arxiv.org/abs/2305.12082) |

---

## Typographic Attacks on CLIP

*Text written inside a picture can overrule what the picture actually shows.*

OpenAI's work on multimodal neurons documented a striking property of CLIP-style models: individual units responded to a concept whether it arrived as a photograph, a drawing, or the written word for it. That abstraction is what makes such models powerful, but it is also a vulnerability. Because a rendered word and the object it names activate overlapping representations, a label physically placed on top of an image can dominate the model's reading of the scene. An apple with a slip of paper reading "iPod" is liable to be classified as an iPod.

The mechanism is the convergence of visual and textual concept pathways in a shared representation. The model has no reliable sense that the pixels forming a word are a different kind of evidence from the pixels forming the depicted object; both feed the same high-level concepts, and the written form can win. This is foundational rather than a jailbreak in itself. The original demonstrations concern image *classification*, not an assistant's refusal policy, and no conversational safety layer is being bypassed.

Its importance is as the ancestor of the typographic family. It established, years before chat VLMs were common, that text-in-image is a first-class adversarial channel and not merely decoration. Later jailbreaks such as FigStep inherit this insight and point it at safety behaviour. The defensive lesson is that any system which reads text out of images has inherited a channel where the written word can override perception, and that channel must be reasoned about separately from the model's handling of ordinary prompt text.

**Source:** [OpenAI](https://openai.com/index/multimodal-neurons/)

---

## Visual Adversarial Examples

*An optimised image becomes a general-purpose lever on an aligned vision-language model.*

This work asks what happens when the continuous image input of an aligned vision-language model is treated as an adversarial variable. Rather than rendering words into a picture, it optimises the pixels themselves so that the resulting image shifts the language component's behaviour. The striking finding is generality: a single crafted image can influence the model's responses across a range of different text requests, rather than unlocking one specific answer.

Why it works follows directly from the chapter's framing. The image is a high-dimensional, continuous input, and the safety alignment lives almost entirely in the text model. Optimising over pixels gives an attacker a smooth search space the alignment process never covered, and small coordinated changes across that space can push internal representations toward states the text-side refusals do not police. Aligning the language model alone does not confer comparable robustness once a vision encoder is attached and its output flows into the same context.

The contribution is conceptual as much as empirical: it reframes the connected image channel as a universal steering surface rather than a source of one-off tricks, and it sits alongside Image Hijacks and Jailbreak in Pieces as evidence that pixel-level optimisation against a VLM is a durable threat model. The defensive implication is that multimodal alignment must be stress-tested against optimised images, and that a model's text-only safety record says little about how it behaves when vision is in the loop. Vendor-reported effectiveness numbers are claims from the paper, not independently reproduced here.

**Source:** [arXiv:2306.13213](https://arxiv.org/abs/2306.13213)

---

## FigStep

*Hand the model the forbidden instruction as a picture of text, then ask it to continue.*

FigStep takes the disallowed request out of the prompt and renders it as text inside an image, then pairs that image with an innocuous, completion-oriented instruction such as asking the model to fill in a numbered list. The model reads the words from the image and integrates them with the surrounding request, arriving at the harmful task by a route that its text-only refusals handle inconsistently.

The reason it works is a gap in where safety alignment reaches. The model is perfectly capable of reading the visual text; comprehension is not the obstacle. The problem is that the safety behaviour trained on textual inputs does not fully extend to content that enters through the visual embedding path. The authors attribute the vulnerability to insufficient safety alignment across visual embeddings rather than to any failure of optical reading. Phrasing the task as a benign completion further distances the surface form from the refusal triggers the model learned on text.

FigStep is the archetypal typographic *jailbreak*, and it is useful to contrast it with its ancestor. The typographic attacks on CLIP showed text-in-image overriding perception in a classifier; FigStep points the same text-in-image channel at a chat assistant's refusal policy, which is a different and higher-stakes target. The defensive lesson is that extracting and acting on text from an image must be routed through the same safety evaluation as text typed into the prompt — otherwise the image becomes a bypass around the policy layer. Reported success rates are the paper's own figures.

**Source:** [arXiv:2311.05608](https://arxiv.org/abs/2311.05608)

---

## HADES

*Split the harmful meaning so that the image both hides it and amplifies it.*

HADES constructs multimodal inputs in which harmful meaning is deliberately redistributed between the text and carefully built images. The visual material plays a dual role: it helps conceal the unsafe interpretation from a surface reading while simultaneously reinforcing that interpretation once the model fuses the two modalities. The combined task can be processed quite differently from an equivalent, explicit textual request.

The mechanism exploits compositional fusion. Each modality alone may look unobjectionable, so neither trips a per-modality check, yet the model's integration of them yields the harmful intent. By engineering the images to carry and strengthen meaning rather than merely accompany the text, the attack works with the model's cross-modal reasoning instead of against it. This is coordinated image-text semantic manipulation, not the naive approach of attaching an arbitrary picture to a jailbreak prompt.

Relative to its neighbours, HADES sits at the semantic end of the compositional family. Where Jailbreak in Pieces optimises in embedding space and Visual Adversarial Examples optimises pixels for generality, HADES focuses on meaning-level concealment and reinforcement across the two channels. The defensive lesson is that safety filtering applied independently to each modality is insufficient: a system can clear every single-channel check and still assemble a disallowed request at the point of fusion, so evaluation must consider the integrated semantics of image and text together. Effectiveness claims are those reported by the authors.

**Source:** [arXiv:2403.09792](https://arxiv.org/abs/2403.09792)

---

## Jailbreak in Pieces

*The dangerous context lives in the image embedding; the text stays bland.*

Jailbreak in Pieces is the clearest statement of the compositional threat model. An adversarial image is crafted to supply a harmful contextual representation through the vision encoder, while the accompanying text request remains generic and unremarkable. The intent is never present in either input on its own; it emerges only when the vision-derived context and the plain text are processed together.

What makes it notable is its threat model. The optimisation targets the behaviour of the vision encoder's embeddings and does not require access to the language model itself — an attacker can work against the perception front end and still steer the fused system. This isolates composition as the core failure: two separately processed inputs, each individually acceptable, jointly induce behaviour that text-only alignment would reject outright. The safety layer never sees a single input that looks dangerous.

The attack anchors the compositional end of this chapter and complements HADES, which operates more at the level of engineered semantics than embedding optimisation. Together they show that per-input moderation is structurally inadequate against intent that is distributed across modalities. The defensive lesson is that the vision encoder's embedding space is part of the trust boundary: content entering through it can carry context the text channel never reveals, so safety must be assessed on the fused representation rather than on each modality in isolation. The paper's reported results are its own.

**Source:** [arXiv:2307.14539](https://arxiv.org/abs/2307.14539)

---

## Image Hijacks

*Optimise an image into a runtime instruction that makes the model behave as the attacker chooses.*

Image Hijacks generalises the pixel-perturbation idea from unlocking unsafe answers to installing arbitrary behaviour. An image is optimised so that, when the model processes it, the model acts according to a chosen target behaviour. A specialisation the authors call "Prompt Matching" tunes an image to emulate the effect of a particular textual instruction, so the picture functions as a stand-in for a prompt the attacker never has to type.

The framing matters because it treats the image as a behavioural control channel rather than a one-shot jailbreak. Studied outcomes include controlling the model's output, leaking context, and overriding safety behaviour — a broader set of effects than simply eliciting a disallowed answer. The mechanism is again the high-dimensional continuous input: optimising pixels against the model's own behaviour lets an image act, at inference time, like an injected instruction that the governing prompt did not authorise.

Among its neighbours, Image Hijacks is the most explicit about the image as a general-purpose instruction surface, where Visual Adversarial Examples emphasises cross-request generality and Jailbreak in Pieces emphasises embedding-space composition. The defensive lesson is that an untrusted image cannot be assumed to be inert data: it can encode a runtime directive with effects comparable to a text instruction, so images from outside the trust boundary deserve the same scrutiny as instructions, not merely content moderation. Reported capabilities are the authors' claims.

**Source:** [arXiv:2309.00236](https://arxiv.org/abs/2309.00236)

---

## Abusing Images and Sounds

*A benign user opens attacker-controlled media, and the media quietly issues instructions.*

This work frames multimodal media as an indirect prompt-injection vector. An external image or audio recording carries an adversarial perturbation that encodes an attacker-chosen instruction. A benign user, doing something ordinary — asking the assistant about a picture or a clip — causes the model to process that media, at which point the embedded instruction can influence the current response and, in some settings, steer the conversation that follows.

The crucial distinction is the boundary being crossed. This is not a user asking directly for disallowed content; it is untrusted media crossing into the space of governing instructions. The model treats the perturbed media as input to interpret, but the perturbation functions as a command, so an adversary who controls a file a victim later views can inject behaviour into the victim's session. The perturbation can persist in effect beyond the immediate turn, shaping subsequent dialogue.

This places the attack squarely in the prompt-injection lineage of Chapter 12, extended across modalities, and it covers both the visual and the audio channel — connecting to the audio and speech material that follows. CrossMPI later sharpens the same idea by targeting multimodal integration layers directly. The defensive lesson is the classic injection principle in a new channel: any media drawn from outside the trust boundary is untrusted input, must never be allowed to act as an instruction, and should be isolated from the model's governing prompt. Reported results are the authors' own.

**Source:** [arXiv:2307.10490](https://arxiv.org/abs/2307.10490)

---

## CrossMPI — Cross-Modal Prompt Injection

*A perturbation in the image hijacks how the model reads the text too.*

CrossMPI studies cross-modal influence directly: an image-only perturbation is optimised to affect the model's interpretation of *both* the image and the accompanying text. Rather than treating the visual embedding in isolation, it examines the hidden states involved in multimodal integration — the internal stages where image and text representations are combined — and concentrates its optimisation on selected integration layers and image regions.

The significance is the demonstrated reach of one modality over another. The attacker perturbs only the image, yet the effect extends to how the model understands text it never touched. This works because the integration layers mix the two streams; by shaping the visual contribution precisely where fusion happens, a perturbation can bias the joint interpretation, redirecting a modality the defender assumed was untouched. Targeting specific layers and regions makes the manipulation more focused than a broad pixel attack.

CrossMPI is a more surgical descendant of the prompt-injection line represented by Abusing Images and Sounds, and it shares the embedding-level perspective of Jailbreak in Pieces while going further into the integration machinery. As the most recent entry here, it reflects a maturing understanding that the fusion mechanism itself is attackable. The defensive lesson is that multimodal integration layers are a distinct part of the attack surface: securing each modality's front end is not enough if an adversary can exploit where the modalities meet. Its reported effectiveness is a claim of the paper, not reproduced here.

**Source:** [arXiv:2605.16090](https://arxiv.org/abs/2605.16090)

---

## SneakyPrompt

*Search for a prompt that slips past a text-to-image generator's safety filter.*

SneakyPrompt is the outlier of this chapter, and deliberately included to mark the boundary of the field. Its target is not a conversational vision-language model but a guarded text-to-image *generator*: a system that turns prompts into pictures and screens requests with a safety filter. SneakyPrompt searches the space of text prompts for ones that the filter admits yet which still drive the generator to produce content the filter was meant to block.

The method uses feedback from the generated outputs to guide changes to the prompt representation, framed as a reinforcement-learning-style search. In effect it probes the discrepancy between two components — the safety filter and the generation model — that do not share a perfectly aligned notion of what is disallowed. Prompts that live in the gap between them pass the gate and still reach the target behaviour, which is why bolting a filter in front of a generator is not equivalent to making the generator safe.

The reason to include SneakyPrompt despite the different target is that it illustrates a general multimodal-safety failure: filter-and-generator mismatch, the same class of problem that lets layered defences be slipped in text systems. It is an adjacent scope rather than a VLM jailbreak, and keeping that distinction clear matters for threat modelling. The defensive lesson is that a safety filter and the model it guards must be evaluated as one system, since the exploitable surface is the space between their differing decision boundaries. Reported bypass rates are the authors' figures.

**Source:** [arXiv:2305.12082](https://arxiv.org/abs/2305.12082)

---
[← 12 · Prompt injection](12-prompt-injection.md) · [Index](../README.md) · [Next: 14 · Audio & speech →](14-audio-and-speech.md)
