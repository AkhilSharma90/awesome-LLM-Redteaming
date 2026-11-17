# Contributing

Thanks for helping make this a better reference. This repo is a **defensive, educational** catalogue of published AI red-teaming methodologies, and contributions are very welcome.

## Two hard rules

1. **Published & cited.** Every entry must correspond to a real, publicly available source — a paper (arXiv/conference), a vendor security advisory, or an open-source tool. Include a working link to the primary source.
2. **Conceptual only.** Entries describe *how a class of attack works and why it bypasses alignment*, plus the defensive lesson. They must **not** include operational jailbreak strings, payloads, exploit code, or step-by-step recipes. If a change would help someone attack a system more than defend one, it doesn't belong here.

## What makes a good new entry

- It establishes a **genuinely new mechanism or attacker capability** — a new trust boundary, modality, or search procedure — rather than just a new persona name or a rebrand of an existing attack.
- It fits one of the existing mechanism families (see the [README](README.md) contents). If it doesn't, propose a new family in your PR description.
- Variants and extensions of an existing method should be **nested under their parent** (e.g. the GCG optimisation variants), not counted as independent families.

## Entry format

Add a row to the relevant `chapters/*.md` table with these six fields:

| Field | Notes |
| --- | --- |
| **Method** | Bold name + acronym expansion where the source supports one. No invented backronyms. |
| **Origin** | Authors and/or the disclosing lab/vendor. |
| **Year / venue** | Initial preprint year, plus later venue if peer-reviewed. Distinguish the two. |
| **Mechanism** | One or more tags from the README legend. |
| **How it works** | ~3–4 lines, conceptual. No payloads. Separate lines with `<br>`. |
| **Sources** | Markdown link(s) to the primary source. |

## Attribution & accuracy

- Prefer the **primary** source over blog re-reports.
- If authorship or a first-attribution date is uncertain, say so, or place the lead in **[Appendix D (unverified)](chapters/appendix-D-unverified-leads.md)** until it can be confirmed.
- Label vendor-reported effectiveness numbers as such; don't present them as independently reproduced.
- Watch for **name collisions** (see [Appendix C](chapters/appendix-C-aliases-and-dedup.md)) — several distinct methods share names.

## How to submit

Open a pull request with your change, or an issue if you've found a wrong attribution or a broken link. Small, focused PRs are easiest to review.
