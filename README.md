# mrm-mdd-kit

**An open source Model Development Document (MDD) kit for ML and LLM applications in regulated industries.**

---

## Two Things This Kit Does That Nothing Else Does

**1. The first open source SR 26-2 compliant MDD template.**
SR 26-2 (*Supervisory Guidance on Model Risk Management*, Federal Reserve / OCC / FDIC, April 17, 2026) is the current authoritative standard for model risk management in banking. No clean, practitioner-grade, publicly available MDD template grounded in SR 26-2 exists. This kit fills that gap.

**2. The first MDD designed for dual human/LLM consumption.**
Every section of the template is structured so that Claude Code (or any LLM) can fill it out collaboratively with a developer, in real time, as the project is built. The kit's `CLAUDE.md` is the engine: it tells Claude how to read the codebase, what to infer, what to ask the developer, and how to draft each section. The MDD grows alongside the code — not as a compliance afterthought after the fact.

This changes the governance dynamic fundamentally. Instead of a document filled out retroactively, you get a living MDD that reflects the actual model, with human judgment woven through every section where business context was required. That's a better artifact — for regulators, for validators, and for the teams that maintain the model.

---

## Who This Is For

**Primary audience:** Developers and data scientists building ML and LLM models in regulated industries — banking, insurance, finance — who must satisfy Model Risk Management (MRM) governance requirements.

**Secondary audience:** Model risk and compliance teams who want a better, versioned, open source starting point for MDD templates.

---

## What's in the Kit

```
mrm-mdd-kit/
├── CLAUDE.md                          ← Instructions for Claude Code: how to generate an MDD
├── template/
│   └── MDD.md                         ← The blank SR 26-2 compliant template
├── examples/
│   ├── llm_complaint_classifier/      ← LLM-based CFPB complaint classifier
│   ├── fraud_detection_xgboost/       ← Real-time CNP fraud scoring (XGBoost)
│   ├── credit_scoring_random_forest/  ← Personal loan PD scorecard (Random Forest)
│   ├── aml_monitoring_neural_network/ ← BSA/AML suspicious activity scoring (Neural Network)
│   └── cecl_logistic_regression/      ← CECL expected credit loss model (Logistic Regression)
├── governance/
│   ├── sr26-2.md                      ← Full text of SR 26-2 (the regulatory basis)
│   ├── how-to-fill-out.md             ← Section-by-section guidance for every template section
│   └── update-process.md             ← Regulatory monitoring and versioning process
├── downstream-skills/                 ← Optional Claude Code skills for downstream ML projects
│   ├── generate-mdd.md                ← /generate-mdd skill (copy to your project's .claude/skills/)
│   └── handoff-to-mrm.md             ← /handoff-to-mrm skill (copy to your project's .claude/skills/)
├── .claude/
│   ├── settings.json                  ← Claude Code hooks + bash allowlist (kit development)
│   └── skills/
│       └── validate-mdd.md            ← /validate-mdd skill (kit development only)
├── mdd-ai-workflow.md                 ← End-to-end process flow diagram (Mermaid)
├── CHANGELOG.md
└── LICENSE (Apache 2.0)
```

Each example contains a `scenario.md` (plain English fictional backstory) and a fully completed `MDD.md` demonstrating the expected depth, tone, and structure across five different model types.

---

## Quick Start

### Option 1: Use the template directly

Copy `template/MDD.md` into your project and fill it out. Use `governance/how-to-fill-out.md` as your section-by-section guide.

### Option 2: Use as a git submodule (recommended)

Adding the kit as a submodule lets you pin to a specific version and bump deliberately when the kit is updated — every regulatory change is a traceable decision.

```bash
# Add the kit to your project
git submodule add https://github.com/o-perro/mrm-mdd-kit _mrm-mdd-kit

# After cloning a project that uses this kit
git submodule update --init

# Update to the latest kit release
cd _mrm-mdd-kit && git pull origin main && cd ..
git add _mrm-mdd-kit && git commit -m "chore: bump mrm-mdd-kit to latest"
```

Then add one line to your project's `CLAUDE.md`:

```
@./_mrm-mdd-kit/CLAUDE.md
```

Claude Code reads your project's `CLAUDE.md`, hits this import, and pulls in full MDD generation instructions automatically.

### Option 3: Generate with Claude Code

With the submodule added and the import in your project's `CLAUDE.md`, start a Claude Code session and say:

> "Let's generate the MDD for this project."

Claude will orient on the codebase, copy the template into your project root as `MDD.md`, and work through it section by section — inferring what it can from the code and asking you for business context it can't know. The result is a compliant, internally consistent MDD grounded in your actual model.

---

## Claude Code Setup

When you add mrm-mdd-kit as a submodule and import its `CLAUDE.md`, Claude Code automatically gains the full MDD generation workflow. Two optional skills are also available that give you named slash commands for the most common workflows.

### What the skills do

| Skill | Command | What it does |
|---|---|---|
| Generate MDD | `/generate-mdd` | Runs the full 5-step MDD workflow with parallel codebase orientation, section-by-section drafting, gap tracking, and a completion check |
| Handoff to MRM | `/handoff-to-mrm` | Handles the template gap workflow — classifies sections, generates a candidate template, and opens the MRM review PR when your model uses a new pattern/algorithm combination |

### How skills get installed

**Automatic (recommended):** The kit's `CLAUDE.md` includes a bootstrap check that runs at the start of every Claude Code session in your project. It checks whether `/generate-mdd` and `/handoff-to-mrm` are present in your project's `.claude/skills/` directory. If either is missing, Claude explains what each skill does and offers to copy them from `_mrm-mdd-kit/downstream-skills/` — before doing anything else. Say yes once and it's done. On every subsequent session the check runs silently and finds the skills already there, so you never see it again.

**Manual:** If you prefer to set them up yourself:

```bash
mkdir -p .claude/skills
cp _mrm-mdd-kit/downstream-skills/generate-mdd.md .claude/skills/
cp _mrm-mdd-kit/downstream-skills/handoff-to-mrm.md .claude/skills/
```

**Skills are optional.** If you skip them, every workflow is still available conversationally — just say "let's generate the MDD" or "I need to hand off a new template to MRM" and Claude follows the same steps. The skills are named shortcuts, not required infrastructure.

### Why skills live in your project, not in the submodule

This is worth understanding because it's non-obvious.

Claude Code resolves skills relative to the **working directory** — the root of the project you have open, not the root of any submodule inside it. That means skill files sitting inside the submodule at `_mrm-mdd-kit/.claude/skills/` are completely invisible to Claude when you're working in your ML project. They might as well not exist.

The solution is the `downstream-skills/` directory in the kit root. Those files are designed to be copied once into your project's own `.claude/skills/` directory, where Claude can actually find them. The bootstrap check in `CLAUDE.md` automates that copy so you don't have to remember it.

The `@./_mrm-mdd-kit/CLAUDE.md` import only pulls in the **text** of that file — the instructions, the workflow steps, the section guidance. It does not share the submodule's `.claude/` directory or make its skills available. Think of it as copy-pasting the instructions into your project's context. The skills need to physically live in your project for Claude to use them as named `/commands`.

---

## The Human-LLM Collaboration Model

Generating an MDD with Claude Code is not a one-shot operation. It is an iterative, back-and-forth collaboration:

| Claude infers from the codebase | You provide |
|---|---|
| Algorithm, framework, features, output schema | Business process, stakeholder names, institutional context |
| What SR 26-2 requires in each section | Portfolio size, dollar volumes, risk appetite |
| What "good" and "insufficient" look like | Compensating controls, operational procedures |
| Questions to ask | Answers — brief is fine; Claude drafts from them |

This loop is itself a human-in-the-loop governance control. The MDD that results reflects documented human judgment at every section where business knowledge was required — not just a signature at the end.

---

## The MDD Template: What It Covers

The template (`template/MDD.md`) covers all SR 26-2 requirements plus practitioner additions that no standard template includes:

| Section | SR 26-2 Reference | What It Adds Beyond Standard Templates |
|---|---|---|
| Document Control + Approval Block | Section VI | Formal approval chain with roles |
| 1. Executive Summary | — | Model risk rating table (Low/Medium/High/Critical) |
| 2. Model Purpose and Business Context | Section IV | Explicit prohibited uses; downstream impact |
| 3. Model Risk Assessment | Section III | Formal materiality scoring: inherent risk × exposure × purpose |
| 4. Data | Section IV | Data quality assessment table; split methodology |
| 5. Model Development | Section IV | Alternatives considered; developmental testing |
| 6. Model Performance | Section V | Subgroup performance; benchmark comparison |
| **7. Assumptions and Limitations** | Section V | **Assumptions registry with compensating controls** — the most commonly weak section in practitioner MDDs |
| 8. Third-Party / Vendor Components | Section VII | Vendor validation approach; ongoing oversight |
| **9. GenAI / LLM Governance** | Spirit of SR 26-2 | **Novel: prompt versioning, output schema, hallucination risk, human-in-the-loop** |
| 10. Validation and Monitoring Plan | Section V | Numeric thresholds with action levels; override policy |
| 11. Implementation and Deployment | — | Input/output specs; rollback plan |
| 12. Governance | Section VI | Roles table; model inventory; change management |
| Appendices A–D | Section VI | Glossary; references; change log; experiment log |

---

## The Five Examples

All five examples use completely fictional institutions, personnel, data, and performance metrics. They exist to demonstrate the expected depth, tone, and structure of a completed MDD — and to serve as few-shot examples for Claude Code when generating MDDs in real projects.

| Example | Model Type | Key Governance Themes |
|---|---|---|
| `llm_complaint_classifier` | LLM (claude-haiku) | SR 26-2 scope note for GenAI, Section 9 full treatment, prompt versioning |
| `fraud_detection_xgboost` | XGBoost | Critical materiality, real-time automated decisions, SHAP adverse action codes, UDAAP |
| `credit_scoring_random_forest` | Random Forest | ECOA / Reg B, fair lending disparate impact, adverse action reason codes |
| `aml_monitoring_neural_network` | Neural Network | BSA/AML, SR 21-8 supersession by SR 26-2, mandatory human-in-the-loop |
| `cecl_logistic_regression` | Logistic Regression | FASB ASC 326 CECL, external validation, macroeconomic scenario dependency |

---

## A Note on SR 26-2 and Generative AI

SR 26-2 (Section II, footnote 3) explicitly excludes generative AI and agentic AI from its formal scope, noting they are "novel and rapidly evolving." However, SR 26-2 also states that institutions' risk management practices should guide governance for tools outside the formal scope.

The `llm_complaint_classifier` example and Section 9 of the template demonstrate how to apply SR 26-2 principles in the spirit of that guidance for LLM-based models. This is the approach leading financial institutions are taking in practice — and it positions institutions well for future regulatory clarity on AI governance.

---

## Regulatory Basis

**Current:** SR 26-2 — *Supervisory Guidance on Model Risk Management*
Issued jointly by the Federal Reserve, OCC, and FDIC on April 17, 2026.
Supersedes SR 11-7 (2011) and SR 21-8 (2021).

Full text: [`governance/sr26-2.md`](governance/sr26-2.md)

For the regulatory monitoring process and how to handle future guidance changes, see [`governance/update-process.md`](governance/update-process.md).

---

## Versioning

This kit uses semantic versioning (`vMAJOR.MINOR.PATCH`). A MAJOR version bump signals that the regulatory basis has changed. See [`CHANGELOG.md`](CHANGELOG.md) for the full version history.

**Current version:** v1.0.0

---

## License

Copyright 2026 Gabriel Olson

Licensed under the Apache License, Version 2.0. See [`LICENSE`](LICENSE) for details.

---

## References

- SR 26-2: [federalreserve.gov/supervisionreg/srletters/SR2602.pdf](https://www.federalreserve.gov/supervisionreg/srletters/SR2602.pdf)
- Vectice MDD Guide (structural inspiration): [vectice.com/blog/a-guide-to-model-development-document-mdd](https://www.vectice.com/blog/a-guide-to-model-development-document-mdd)
- Eugene Yan — ml-design-docs (structural inspiration): [github.com/eugeneyan/ml-design-docs](https://github.com/eugeneyan/ml-design-docs)

---

## Contributing

See [`.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md). The data privacy rule is absolute: no real institution data, ever.
