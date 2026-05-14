# Contributing to mrm-mdd-kit

Thank you for your interest in contributing. This kit exists to give practitioners in regulated industries a clean, open source starting point for SR 26-2 compliant Model Development Documents — including the first MDD template designed for dual human/LLM consumption. Contributions that improve regulatory accuracy, practical usability, or example coverage are welcome.

---

## What We're Looking For

Good contributions fall into one of these categories:

**Regulatory accuracy**
- Corrections to `governance/sr26-2.md` or any regulatory content
- Updates when new guidance is issued (see `governance/update-process.md` for the versioning process)
- Additions to the regulatory applicability table in `template/MDD.md`

**Template improvements**
- New sections that address genuine regulatory or governance gaps
- Improvements to `[FILL IN: ...]` placeholder guidance that make sections clearer for human modelers or LLMs
- New or improved LLM Collaboration Guide blocks in `governance/how-to-fill-out.md`

**New examples**
- A new fictional example covering a model type not yet represented (e.g., market risk, insurance, operational risk)
- Must follow the fictional-only rule — see below

**CLAUDE.md improvements**
- Better section-specific guidance for Claude Code
- Corrections to the MDD generation workflow

**Bug fixes**
- Broken links, formatting errors, factual inaccuracies

---

## What We're Not Looking For

- Real institution data, real MDD content, or any content derived from proprietary bank documents — this is a hard rule, not a preference (see the data privacy rule below)
- Model Validation Document templates, model inventory templates, or other MRM artifacts outside the MDD scope — keep it focused
- Changes that reduce the regulatory grounding of the template in favor of simplicity
- Additions that are speculative or not grounded in public regulatory sources

---

## The Data Privacy Rule

**This rule is absolute.**

Everything in this repository must be either:
1. Publicly available regulatory content (Fed, OCC, FDIC, FinCEN, CFPB, FASB), or
2. Completely fictional

Never contribute real institution names, real model details, real performance metrics, real employee names, real customer data, or any content derived from internal bank documents. This kit is public and open source. If you work at a bank and want to contribute an example, build it from scratch as a fictional scenario — do not adapt or anonymize real documents.

---

## How to Contribute

1. **Open an issue first** for anything beyond a small bug fix. Describe what you want to change and why. This avoids duplicate work and ensures the change aligns with the kit's scope.

2. **Fork and branch.** Use a descriptive branch name: `feat/new-insurance-example`, `fix/section-9-typo`, `regulatory/sr-26-2-update`.

3. **Make your changes.** Follow the conventions in the existing files — formatting, placeholder style, LLM Collaboration Guide structure.

4. **Update CHANGELOG.md.** Add an entry under `[Unreleased]` describing your change.

5. **Open a pull request** against `main`. Fill out the PR description: what changed, why, and any regulatory sources cited.

6. **Wait for review.** The maintainer will review for regulatory accuracy, data privacy compliance, and consistency with the kit's design.

---

## Regulatory Accuracy Standard

All content in this kit that describes regulatory requirements must be traceable to a specific public document. When contributing content that references regulatory requirements:

- Cite the specific document, section, and date (e.g., "SR 26-2, Section IV, April 17, 2026")
- Do not paraphrase in a way that changes the meaning of the original guidance
- If you are uncertain whether something is a regulatory requirement vs. a sound practice recommendation, say so explicitly

---

## Formatting Conventions

- Markdown throughout — no Word documents, no PDFs in PRs
- Placeholders use `[FILL IN: description]` format — consistent with the rest of the template
- LLM Collaboration Guide blocks use the exact heading `**LLM Collaboration Guide:**` — do not vary this
- Section references in the template and how-to-fill-out guide use the format `Section X.Y`
- Tables preferred over prose for structured data

---

## Code of Conduct

Be direct, be accurate, be respectful. This is a professional toolkit for practitioners in regulated industries. Contributions should be held to the same standard of rigor expected of the documents this kit helps produce.

---

## License

By submitting a pull request or contribution to this project, you agree that your contributions will be licensed under the Apache License, Version 2.0, the same license that covers this project.

See [LICENSE](../LICENSE) for details.

---

## Questions

Open a GitHub issue with the `question` label. For regulatory interpretation questions, note that responses reflect the maintainer's reading of public guidance and should not be relied upon as legal or compliance advice.
