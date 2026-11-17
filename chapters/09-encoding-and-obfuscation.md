# 09 · Encoding, obfuscation & transformation

[← Back to index](../README.md)

> Safety training lives mostly in natural-language space. Move the request *out* of that space — into ciphers, code, art, maths, or another language — and the model decodes and answers before the guardrails engage.

Nearly every production safety system is trained on, and reasons over, fluent natural language. Refusal classifiers, RLHF preference data, and system-prompt guardrails all learned what "harmful" looks like from text that reads like ordinary prose. The attacks in this chapter share a single insight: if the harmful request never appears in that familiar surface form, the part of the model that recognises danger may never fire — even though the part of the model that *understands* the request fires just fine. A capable model will happily decode a cipher, read a word out of ASCII art, translate from a rare language, or complete a code template, and only then act on the recovered meaning. By that point the sensitive content lives in the model's internal representation, not in any text a filter can scan.

This is the purest exploitation of **mismatched generalisation**: capability generalises to a new representation faster than safety does. The model is smart enough to invert the transformation but not aligned enough to carry its refusal policy across the transformation with it. Crucially, the technique does not require finding a weakness in the policy's *content* — it only requires moving the conversation to a representation the policy was never trained to police.

The family splits along a useful axis. **Fixed encodings** apply a transformation the model already knows or can be told how to invert in-prompt: ciphers, ASCII art, character flips, low-resource translation, payload fragmentation. **Learned encodings** teach the model a fresh, previously-unseen mapping during the interaction itself, so no static signature exists to detect. And **semantic decomposition** attacks neither encrypt nor translate — they shatter the request into individually-innocuous pieces (clues, sub-tasks, game descriptions, mathematical objects) and let the model reassemble the forbidden whole through its own inference. The defensive takeaway is consistent across all three: input-level keyword and surface-form filtering is structurally insufficient. Robust defence has to evaluate *recovered intent* — what the model is about to act on after decoding — rather than the literal tokens that arrived at the input.

## Quick reference

| Method | Origin | Year / venue | Mechanism | How it works (conceptual) | Sources |
| --- | --- | --- | --- | --- | --- |
| **CipherChat** | Youliang Yuan et al. | 2023 (ICLR 2024) | cipher encoding | The attacker converses entirely through a cipher, instructing the model to decode, reason, and reply in the same cipher; the natural-language safety layer never sees recognisable harmful text. | [arXiv:2308.06463](https://arxiv.org/abs/2308.06463) |
| **CodeChameleon** | Huijie Lv et al. | 2024 | personalised encryption; code framing | Reframes the request as a programming problem whose payload is "encrypted" by a personalised transform and bundled with a decrypt routine the model runs to reconstruct meaning. | [arXiv:2402.16717](https://arxiv.org/abs/2402.16717) |
| **ArtPrompt** | Fengqing Jiang et al. | 2024 (ACL 2024) | ASCII art | Hides the sensitive trigger word by rendering it as ASCII art; token filters see only characters, while the model's pattern recognition still reads the word. | [arXiv:2402.11753](https://arxiv.org/abs/2402.11753) |
| **FlipAttack** | Yue Liu et al. | 2024 | character/word flip; black-box | Scrambles the request by flipping character or word order with an in-prompt rule to un-flip it; the disguised text passes filters before the model restores and executes it. | [arXiv:2410.02832](https://arxiv.org/abs/2410.02832) |
| **Low-Resource Language Attack** | Zheng-Xin Yong, Cristina Menghini, Stephen Bach | 2023 | translation; cross-lingual | Translates the request into a low-resource language where alignment coverage is thin, submits it, and translates the answer back. | [arXiv:2310.02446](https://arxiv.org/abs/2310.02446) |
| **Payload Splitting / Token Smuggling** | Daniel Kang et al. | 2023 | fragmentation; programmatic | Breaks harmful content into innocuous fragments assembled only at "runtime" by a join instruction; no single part trips a filter. | [arXiv:2302.05733](https://arxiv.org/abs/2302.05733) |
| **CodeAttack** | Qibing Ren, Chang Gao, Jing Shao et al. | 2024 — arXiv | encoding/obfuscation | Represents natural-language requests as code-completion tasks, probing whether safety behaviour transfers from conversation to code. | [arXiv:2403.07865](https://arxiv.org/abs/2403.07865) |
| **DrAttack — Decomposition and Reconstruction** | Xirui Li, Ruochen Wang, Minhao Cheng, Tianyi Zhou, Cho-Jui Hsieh | 2024 (Findings of EMNLP) | semantic decomposition; attacker-LLM | Splits a request into separated semantic components, then uses in-context demonstrations to guide reconstruction of the harmful whole. | [Findings of EMNLP](https://aclanthology.org/2024.findings-emnlp.813/) |
| **DRA — Disguise and Reconstruction Attack** | Tong Liu, Yingjie Zhang, Zhe Zhao, Yinpeng Dong, Guozhu Meng, Kai Chen | 2024 — arXiv | disguise; generation-side reconstruction | Disguises the instruction in the input and induces the model to reconstruct it inside its own continuation, moving sensitive interpretation into generation. | [arXiv:2402.18104](https://arxiv.org/abs/2402.18104) |
| **WordGame** | Tianrong Zhang, Bochuan Cao, Yuanpu Cao, Lu Lin, Prasenjit Mitra, Jinghui Chen | 2024 (Findings of NAACL 2025) | semantic substitution; output shaping | Represents sensitive terms as word-game puzzles and shapes the response to open with benign game-solving, shifting both input and output distributions. | [arXiv:2405.14023](https://arxiv.org/abs/2405.14023) |
| **Puzzler** | Zhiyuan Chang, Mingyang Li, Yi Liu, Junjie Wang, Qing Wang, Yang Liu | 2024 (Findings of ACL) | semantic indirection | Supplies clues from which the model infers an unstated target; harmful semantics are completed through the model's own inference. | [Findings of ACL](https://aclanthology.org/2024.findings-acl.304/) |
| **MathPrompt** | Emet Bethany, Mazal Bethany, J. A. Nolazco Flores, S. K. Jha, Peyman Najafirad | 2024 — arXiv | symbolic recasting | Recasts natural-language intent as a symbolic mathematical problem that lies outside the surface forms safety training emphasises. | [arXiv:2409.11445](https://arxiv.org/abs/2409.11445) |
| **Bijection Learning** ("Endless Jailbreaks") | Brian R. Y. Huang, Maximilian Li, Leonard Tang; Haize Labs | 2024 — arXiv | learned encoding; in-context learning | Teaches the model a freshly sampled one-to-one encoding via in-context demonstrations, then converses in it; complexity is tuned to trade capability against safety. | [arXiv:2410.01294](https://arxiv.org/abs/2410.01294) |
| **IMM — Infinitely Many Meanings** | O. Goldstein, E. La Malfa, F. Drinkall, S. Marro, M. Wooldridge | 2025 — arXiv | taxonomy; meaning-preserving maps | Studies a broad class of meaning-preserving mappings showing that many surface forms express one unsafe request faster than safeguards can cover them. | [arXiv:2501.10800](https://arxiv.org/abs/2501.10800) |

---

## CipherChat
*Conversing entirely through a cipher the model is told how to invert.*

CipherChat, whose paper is memorably subtitled *"GPT-4 is too smart to be safe,"* establishes the foundational result of this chapter. The attacker instructs the model to carry out the whole exchange through a cipher — a Caesar-style shift, Morse, a character-encoding scheme, or a custom substitution — and to reply in that same cipher. A sufficiently capable model follows the instruction: it inverts the encoding internally, reasons over the recovered request, and emits an encoded answer. The safety layer, trained almost exclusively on natural-language text, never encounters recognisable harmful strings on the input or the output side, so it has nothing to react to.

The mechanism bypasses alignment precisely because decoding and refusal are decoupled. The model's competence at inverting a described transformation is a general capability; its tendency to refuse was fit to ordinary prose. When the request arrives outside that distribution, capability transfers but the refusal policy does not — the canonical statement of mismatched generalisation. Notably, the attack grows *stronger* on more capable models, because reliable decoding is a prerequisite: weaker models fail to invert the cipher and the attack simply breaks.

CipherChat anchors the "fixed encoding" branch of the family and is the direct conceptual ancestor of Bijection Learning (which learns the code in-context instead of assuming a known one) and of IMM (which generalises the idea to an entire taxonomy of meaning-preserving maps). Its defensive lesson is durable: safety evaluation must consider what the model will decode a message into, not the literal characters that arrive.

**Source:** [arXiv:2308.06463](https://arxiv.org/abs/2308.06463)

## CodeChameleon
*Smuggling the request as an encrypted programming problem bundled with its own decrypt routine.*

CodeChameleon reframes a harmful request as a software task. The sensitive instruction is "encrypted" by a personalised transform — for example, its elements reordered according to a scheme such as a binary-tree traversal — and then packaged alongside a decryption function that the model is asked to apply as part of completing the code. The model mentally runs the decrypt routine, recovers the original instruction, and proceeds to fulfil it inside the programming frame.

What makes this effective is a double shift. The *format* moves from conversation to code, where intent-recognition is weaker, and the *content* is obfuscated by a transform that only the accompanying routine can undo. Because the harmful semantics exist only after the decrypt step runs, an input-side filter scanning the prompt sees an encrypted blob and a benign-looking helper function, not a prohibited request. The personalised nature of the encryption also frustrates signature-based detection: there is no single fixed string to match.

CodeChameleon sits between pure cipher attacks and code-framing attacks. Unlike CipherChat, the transformation is wrapped in a programming pretext and paired with an explicit decoder; unlike CodeAttack, the payload is genuinely encrypted rather than merely expressed as code. The defensive implication is that safety checks cannot stop at the literal input — they must account for transformations the prompt itself instructs the model to perform.

**Source:** [arXiv:2402.16717](https://arxiv.org/abs/2402.16717)

## ArtPrompt
*Hiding the banned word in plain sight by drawing it as ASCII art.*

ArtPrompt targets the brittleness of keyword-based filtering. A sensitive trigger word is removed from the text and instead rendered as ASCII art — a picture of the word built from ordinary characters — while the surrounding prompt remains an unremarkable request that refers to "the word above." A filter scanning tokens never sees the forbidden term, because at the character level it is not present as a word at all; it is an image.

The attack works because of an asymmetry between the filter and the model. The safety filter operates on surface tokens and finds nothing to block. The underlying model, however, has strong enough visual-structural pattern recognition to "read" the shape and recover the intended word, after which it acts on the fully reconstituted request. In other words, the same capability that makes the model useful — robust pattern completion — is exactly what defeats the shallow filter protecting it.

ArtPrompt is a vivid, low-cost demonstration that filtering on the literal input string is fundamentally fragile against models that understand structure rather than just characters. It belongs to the fixed-encoding branch alongside CipherChat and FlipAttack, differing in that the "encoding" is visual typography rather than a symbolic cipher. The lesson generalises: any defence that assumes harmful content must appear as a scannable word will be defeated by a representation the model can still parse.

**Source:** [arXiv:2402.11753](https://arxiv.org/abs/2402.11753)

## FlipAttack
*Scrambling the request by reversing order, with a one-line rule telling the model how to unscramble it.*

FlipAttack exploits the left-to-right, autoregressive way language models read text. The harmful request is disguised by flipping its characters or reversing word order, and a short in-prompt rule tells the model how to restore the original. The scrambled string does not match harmful patterns on the way in; the model follows the un-flip rule, recovers the request, and executes it.

The bypass follows the same logic as the rest of the family — the dangerous content is present only after a transformation the model performs itself — but FlipAttack is notable for how little it costs. It is black-box, single-turn, and requires no optimisation, auxiliary model, or special knowledge: just a mechanical reordering and a rule to undo it. Despite that simplicity it has proven surprisingly effective against frontier models, underscoring that the vulnerability is structural rather than a matter of attack sophistication.

Conceptually FlipAttack is a minimalist member of the fixed-encoding branch, close to CipherChat but with the lightest possible "cipher." Its contribution to the defensive picture is to show that the mismatched-generalisation gap can be opened with almost no effort, which raises the bar for any filtering-based defence: if a trivial reversal suffices, filters must reckon with an unbounded space of cheap surface transformations.

**Source:** [arXiv:2410.02832](https://arxiv.org/abs/2410.02832)

## Low-Resource Language Attack
*Routing the request through a language where safety training barely reaches.*

This work observes that safety alignment is overwhelmingly tuned on high-resource languages — English above all. The attack translates a harmful request into a low-resource language such as Zulu or Scots Gaelic, submits it, and translates the response back. The model retains enough cross-lingual competence to understand and comply, but its guardrails, thin or absent in that language, fail to engage.

The mechanism is a cross-lingual instance of mismatched generalisation. A large multilingual model's *understanding* generalises broadly across languages because it was pretrained on many of them, while its *safety behaviour* was concentrated in the languages that dominate alignment datasets. The result is a coverage gap: the capability surface is far wider than the safety surface, and translation is simply the cheapest way to step off the safety surface while staying on the capability one.

The study exposed a serious and previously underappreciated multilingual hole in alignment, with direct implications for anyone deploying models to a global user base. It differs from the cipher and code attacks in that the "encoding" is a real human language the model genuinely knows, rather than an invented transform — but the defensive conclusion converges: safety has to be evaluated on recovered meaning, and alignment coverage must extend to every language the model can actually operate in.

**Source:** [arXiv:2310.02446](https://arxiv.org/abs/2310.02446)

## Payload Splitting / Token Smuggling
*Splitting the request into harmless fragments the model concatenates into the forbidden whole.*

Payload splitting treats the model like a program that will dutifully assemble its inputs. The harmful content is broken into fragments — assigned to variables and interleaved with benign text — each of which is innocuous on its own. A final instruction tells the model to join the pieces, and only at that "runtime" moment does the prohibited request come into existence. No individual fragment trips a content filter, yet the composed result is exactly the banned instruction.

The bypass succeeds because filters typically evaluate the input as presented, whereas the harmful semantics emerge only after an in-prompt composition step. The attacker exploits the model's instruction-following reliability: the very obedience that makes it a good assistant is what performs the reassembly. This is distinct from semantic decomposition attacks later in the chapter — splitting here is lexical and programmatic (string concatenation) rather than conceptual.

Framed as part of a broader finding that instruction-following models inherit classical program-injection weaknesses, this technique draws a direct line from traditional application security to LLM safety. The defensive lesson is that evaluation must occur *after* any composition the prompt directs, on the reconstructed payload, not on the scattered fragments — the same "check recovered intent" principle that recurs throughout the family.

**Source:** [arXiv:2302.05733](https://arxiv.org/abs/2302.05733)

## CodeAttack
*Expressing the request as a code-completion task and letting the completion habit outrun the safety habit.*

CodeAttack represents natural-language requests as programming problems — typically code-completion tasks in which the model is invited to fill in a function or data structure. Reframing the request in program structures changes how the model interprets both the task and the form of the expected output, probing a specific question: does safety behaviour learned in conversational settings transfer when the same request is dressed as code?

The hypothesised gap is that models carry a strong learned tendency to *complete* code — a behaviour heavily rewarded during training — that can override the semantic safety judgement they would apply to the equivalent plain-language request. In effect the code frame activates a "helpful completion" disposition and partially suppresses the "evaluate for harm" disposition, so the model fills in the structure before weighing whether it should. The transformation is representational rather than cryptographic: nothing is encrypted, the request is simply recast into a programming idiom.

CodeAttack is best understood in contrast with CodeChameleon. Both use code as the vector, but CodeChameleon genuinely encrypts the payload and supplies a decrypt routine, whereas CodeAttack relies on the framing alone — the request is legible, just expressed as code. The defensive takeaway is that safety alignment must be demonstrated to transfer across input modalities and formats, not assumed to hold once it is established in conversational text.

**Source:** [arXiv:2403.07865](https://arxiv.org/abs/2403.07865)

## DrAttack — Decomposition and Reconstruction
*Breaking a request into semantic parts that each read as benign, then guiding the model to reassemble the harmful whole.*

DrAttack separates a request into its semantic components and then relies on the model to put them back together. In-context demonstrations guide the model to reconstruct the relationships among the fragments, and an optimisation process searches for decompositions that preserve the original meaning while obscuring the harmful combination. Each component, considered alone, looks unobjectionable; the harm lives in their composition.

The failure it targets is a compositional one: a model (and any classifier screening it) tends to judge fragments more leniently than their reconstructed whole. Because the sensitive meaning is distributed across parts and only reconstituted through guided inference, surface-level screening of the prompt sees a collection of benign pieces. This is deliberately *semantic* decomposition, not the character- or string-level splitting of payload-splitting attacks — the pieces are conceptual roles rather than substrings to be concatenated.

As a Findings-of-EMNLP contribution, DrAttack makes the case that robust safety has to reason over composed meaning, not isolated spans. It shares the decomposition-and-reconstruction theme with DRA, WordGame and Puzzler, but is distinguished by its explicit optimisation over meaning-preserving decompositions. The defensive lesson is sharp: a filter that scores fragments independently will systematically underestimate their joint intent.

**Source:** [Findings of EMNLP](https://aclanthology.org/2024.findings-emnlp.813/)

## DRA — Disguise and Reconstruction Attack
*Disguising the instruction in the input and letting the model rebuild it inside its own answer.*

DRA presents a harmful instruction in disguised form rather than stating it directly, then induces the model to reconstruct that instruction within its own generated continuation. The pivotal move is that part of the sensitive interpretation is shifted out of input processing and into generation: the model itself produces the clear statement of the request, in its own output, as a step toward fulfilling it.

This matters because many safety mechanisms concentrate on the input — scanning the prompt for prohibited content — and apply weaker scrutiny to how the model elaborates its own reasoning mid-generation. If the undisguised instruction only materialises partway through the model's response, an input-focused guard may already have passed the prompt, and the harmful semantics are reconstructed downstream of the checkpoint. DRA thus functions as a test of whether safety checks cover reconstructed intent as thoroughly as directly stated intent.

Within the decomposition branch, DRA is distinguished by *where* reconstruction happens — in the model's generation rather than through input-side concatenation (payload splitting) or guided fragment reassembly (DrAttack). Its defensive implication is that monitoring must extend across the full generation trajectory, catching intent that the model surfaces in its own continuation, not only intent present at the input boundary.

**Source:** [arXiv:2402.18104](https://arxiv.org/abs/2402.18104)

## WordGame
*Replacing the sensitive term with a word puzzle and priming the answer to begin with harmless game-play.*

WordGame represents sensitive terms through word-game descriptions — puzzle-like clues that point to the banned concept without naming it — and additionally shapes the response so that it opens with benign game-solving material. Both the query representation and the early generated context therefore differ from a direct harmful request, shifting the interaction away from the forms safety training is tuned on.

The attack combines two of the chapter's themes. On the input side it is a semantic substitution: the dangerous word is swapped for a description the model must resolve, so filters see a puzzle rather than a prohibited term. On the output side it steers the start of the response into innocuous territory, exploiting the tendency of autoregressive models to continue in the register they have begun. By the time the generation reaches the sensitive content, both the prompt and the response context look like a game, and the refusal trigger has had little purchase.

WordGame's distinctive contribution is this *joint* input-and-output distribution shift — more than word replacement alone, it also manages the response trajectory. It is a close relative of Puzzler (clue-driven inference) and the output-shaping ideas seen in prefix-style attacks elsewhere in the literature. The defensive lesson is that both the representation of the request and the shaping of the model's own opening tokens are attack surfaces that safety training must cover.

**Source:** [arXiv:2405.14023](https://arxiv.org/abs/2405.14023)

## Puzzler
*Feeding the model clues and letting it infer the forbidden request no one ever stated.*

Puzzler never states the target request. Instead it supplies a set of clues from which the model infers the unstated goal, completing the harmful semantics through its own reasoning. The original work gathers useful clues under a defensive framing — assembling fragments that individually read as harmless or even safety-oriented — and lets the model's inference bridge them to the intended task.

The bypass turns on indirection. A classifier examining the prompt may judge each clue as benign, because no clue names the prohibited objective; the objective exists only as the conclusion the model draws. Safety screening that looks for an explicit harmful request finds none, yet the model, reasoning over the clues, arrives at and acts on exactly that request. It is worth stressing that "indirect" here means semantic inference, not document or tool injection — nothing external is smuggled in; the indirection is purely in how the goal is communicated.

As a Findings-of-ACL work, Puzzler sits in the semantic-decomposition branch alongside DrAttack and WordGame, but it is distinguished by relocating the entire harmful target into the model's inferential step. The defensive implication is that safety evaluation cannot depend on an explicit statement of intent being present in the input — it must consider the conclusions the model is likely to infer and then enact.

**Source:** [Findings of ACL](https://aclanthology.org/2024.findings-acl.304/)

## MathPrompt
*Recasting intent as a symbolic maths problem that lives outside the forms safety data emphasises.*

MathPrompt recasts natural-language intent as a symbolic mathematical problem. Relationships that would be explicit and recognisable in ordinary prose are instead expressed through mathematical structure — variables, operations, and formal relations — which the model's mathematical reasoning can recover but which a prose-oriented safety filter is poorly equipped to read.

The transformed input lies outside the surface forms that dominate safety training examples. The model's competence at symbolic reasoning lets it reconstruct the underlying request from the formal statement, while the safety mechanisms, having seen few if any harmful requests dressed as mathematics, fail to generalise to the new representation. The behaviour being requested is unchanged — only its representation moves — which is precisely the authors' point: a semantic shift in representation can degrade safety generalisation even when the harmful content itself is constant.

MathPrompt is a strong illustration that the mismatched-generalisation gap is not limited to obvious codes and ciphers but extends to any formal system the model understands. It is a fixed-representation attack like CipherChat and FlipAttack, differing in that the "encoding" is the language of mathematics rather than a symbolic cipher. The defensive lesson reinforces the chapter's thesis: safety coverage must span every representational mode in which the model is competent, mathematics included.

**Source:** [arXiv:2409.11445](https://arxiv.org/abs/2409.11445)

## Bijection Learning
*Teaching the model a brand-new one-to-one code during the chat, so there is no fixed signature to catch.*

Bijection Learning, from the "Endless Jailbreaks" work, has the model learn a freshly sampled one-to-one encoding from in-context demonstrations and then conducts the conversation — requests and responses alike — in that learned representation. Because the mapping is generated anew for each attempt rather than drawn from a known scheme, there is no static cipher to fingerprint, and the attacker can vary the encoding's complexity to probe the boundary between capability and safety generalisation.

The attack generalises CipherChat along a critical dimension. CipherChat assumes a cipher the model already knows how to invert; Bijection Learning instead *teaches* an arbitrary code on the fly, exploiting the model's in-context learning ability. This yields two properties the "endless" framing captures: the space of possible encodings is effectively unbounded, so no enumerable blocklist of ciphers suffices; and tuning encoding complexity lets the attacker find the sweet spot where the model is still able to decode reliably but its safety behaviour has failed to follow the mapping.

Bijection Learning is the learned-encoding branch's central result, standing between CipherChat (fixed codes) and IMM (which frames the whole phenomenon as a taxonomy). Its defensive lesson is uncomfortable: any defence enumerating known encodings is defeated by a mapping invented at inference time, so protection must again rest on evaluating recovered intent rather than recognising the surface encoding.

**Source:** [arXiv:2410.01294](https://arxiv.org/abs/2410.01294)

## IMM — Infinitely Many Meanings
*Mapping the whole space of meaning-preserving transformations through which one unsafe request can travel.*

IMM — "Infinitely Many Meanings" — steps back from any single technique to study the broad class of meaning-preserving mappings and encoded communications as a phenomenon in its own right. Its central observation is that a vast, in principle unbounded, set of surface forms can all express the same underlying unsafe request, and a model's ability to bind and decode new representations can outpace the safeguards defined over them.

As an umbrella analysis, IMM reframes the chapter's individual attacks as instances of one structural weakness rather than a collection of separate tricks. If every meaning-preserving transformation is a potential vector, then defending by cataloguing specific encodings is a losing proposition — the space is too large and too easily extended, a point that resonates directly with Bijection Learning's "endless" framing and with CipherChat's original cipher result. The work's contribution is taxonomic and conceptual: it names the general failure mode and argues for its breadth.

Positioned adjacent to Bijection Learning and CipherChat, IMM is the most abstract member of the family and a fitting capstone to the chapter. Its defensive lesson is the one every attack here points toward: because the attacker's representational freedom is effectively infinite, durable safety cannot live at the level of surface forms — it has to operate on the meaning the model recovers and is about to act on.

**Source:** [arXiv:2501.10800](https://arxiv.org/abs/2501.10800)

---
[← 08 · Multi-agent](08-multi-agent.md) · [Index](../README.md) · [Next: 10 · Persuasion →](10-persuasion-and-reasoning-spoofing.md)
