# mrm-mdd-kit — Claude Code Instructions

> This file is imported by a project's `CLAUDE.md` via `@./_mrm-mdd-kit/CLAUDE.md`. When Claude Code reads a project's `CLAUDE.md` and hits this import, these instructions are loaded automatically. Everything below tells Claude how to use this kit to generate a compliant Model Development Document (MDD) for the project.

---

## First-Run Setup Check

**Every time this file is loaded, run this check before doing anything else:**

Check whether the following files exist in the project's `.claude/skills/` directory:
- `.claude/skills/generate-mdd.md`
- `.claude/skills/handoff-to-mrm.md`

If either file is missing, say exactly this before proceeding with any other work:

> *"I noticed the mrm-mdd-kit skills aren't set up in this project yet. Here's what they do:*
>
> - **`/generate-mdd`** — runs the full 5-step MDD generation workflow with parallel codebase orientation, section-by-section drafting, gap tracking, and a completion check. Gives you a reliable, named entry point instead of having to explain the process each session.
> - **`/handoff-to-mrm`** — handles the template gap workflow when your model uses a pattern/algorithm combination with no approved template yet. Classifies sections, generates the candidate template, and opens the MRM review PR automatically.
>
> *Want me to copy these skills to `.claude/skills/` now? It's a one-time setup and you can still invoke the workflows by asking me directly if you prefer not to.*
> *(yes / no)"*

If the developer says **yes**: copy `_mrm-mdd-kit/downstream-skills/generate-mdd.md` and `_mrm-mdd-kit/downstream-skills/handoff-to-mrm.md` to `.claude/skills/`. Confirm when done and explain the skills are now available as `/generate-mdd` and `/handoff-to-mrm`.

If the developer says **no**: acknowledge and continue. The full MDD workflows remain available via natural language — the skills are optional named shortcuts, not required.

If both files already exist: skip this check entirely and proceed normally.

---

## What This Kit Is

**mrm-mdd-kit** is an open source MDD kit for ML and LLM projects in regulated industries — banking, insurance, and finance. It provides:

1. **The first open source SR 26-2 compliant MDD template** — grounded in the current authoritative Federal Reserve guidance (SR 26-2, April 17, 2026). No comparable public template exists.

2. **The first MDD designed for dual human/LLM consumption** — the template and this CLAUDE.md are structured so that Claude Code can fill out a compliant MDD in real time, collaboratively with the developer, as the project is built. The MDD grows alongside the code — not as a compliance afterthought after the fact.

**Regulatory basis:** SR 26-2 (*Supervisory Guidance on Model Risk Management*, April 17, 2026), issued jointly by the Federal Reserve, OCC, and FDIC. This guidance supersedes SR 11-7 (2011). Full text: `_mrm-mdd-kit/governance/sr26-2.md`.

---

## Critical: Data Privacy Rule

**This rule is absolute and applies to Claude Code at all times when using this kit.**

Never put real company data, proprietary model details, internal documents, customer data, employee names, or any confidential information into any MDD generated from this kit if the output will be committed to a public repository or shared outside the institution.

When generating example or test content, always use:
- Fictional institution names
- Invented performance metrics
- Synthetic or hypothetical data descriptions
- Made-up personnel names

This rule applies to the `examples/` directory in this kit and to any MDD generated in a development or demonstration context outside a private, controlled environment.

---

## Kit Contents Reference

| File | Purpose |
|---|---|
| `_mrm-mdd-kit/template/MDD.md` | The blank MDD template — copy this into the project root as `MDD.md` to begin |
| `_mrm-mdd-kit/governance/sr26-2.md` | Full text of SR 26-2 — the authoritative regulatory source |
| `_mrm-mdd-kit/governance/how-to-fill-out.md` | Section-by-section guidance — what regulators look for, what good looks like, LLM Collaboration Guides for every section |
| `_mrm-mdd-kit/examples/` | Five fully completed fictional MDDs across different model types — use as reference for depth, tone, and structure |

---

## The Human-LLM Collaboration Model

Generating an MDD with Claude Code is **not a one-shot operation**. It is an iterative, back-and-forth collaboration between Claude and the developer. This is by design — and it is what makes the resulting MDD credible for regulatory review.

**Why collaboration matters:** An MDD produced through genuine human-LLM collaboration has documented human judgment woven through it at every section where business context was required. That is a fundamentally different artifact than one generated autonomously. The iterative process is itself a human-in-the-loop governance control.

**The two roles:**

| Claude knows... | The developer knows... |
|---|---|
| What the code does — algorithm, features, output schema, data pipelines, thresholds, dependencies | What the business process is — the workflow this model supports, what existed before it |
| What SR 26-2 requires in each section | Who the stakeholders are — model owner, MRM contact, intended users |
| What "good" and "insufficient" look like | What the institutional context is — portfolio size, dollar volumes, risk appetite |
| What questions to ask | What the compensating controls are — operational procedures not in the code |
| How to draft compliant prose | Whether the draft is factually accurate — only the developer can verify business facts |

**The workflow for each section:**
1. Claude reads the codebase and drafts what it can infer directly
2. Claude asks targeted questions for what it cannot know
3. Developer answers — brief answers are fine; Claude shapes them into compliant prose
4. Claude drafts the section
5. Developer reviews for accuracy and approves or redirects
6. Move to the next section

---

## MDD Generation Workflow

When a developer asks Claude to generate or work on an MDD for a project, follow this workflow precisely.

### Step 0: Orient

Before writing a single line of the MDD, read the following in order:
1. The project `README.md` — understand what the model does at a high level
2. The main training or inference script — understand the algorithm, inputs, and outputs
3. The data pipeline or feature engineering code — understand the data
4. Any experiment logs, notebooks, or config files — understand development decisions
5. `_mrm-mdd-kit/governance/how-to-fill-out.md` — review the guidance for the sections you are about to write

Do not begin drafting until you have read enough to answer: *What does this model predict? What data does it use? What algorithm? What is the output format?*

### Step 1: Copy the Template

If `MDD.md` does not already exist in the project root, tell the developer:

> "I'm going to create `MDD.md` in the project root by copying the kit template. This will be the living MDD for this project — committed and versioned alongside the code. Ready to proceed?"

Then copy `_mrm-mdd-kit/template/MDD.md` to the project root as `MDD.md`.

### Step 2: Identify Model Type

Determine whether the model is:
- **Traditional ML** — statistical or ML model (regression, tree-based, neural network, etc.)
- **LLM-based** — uses a generative AI or large language model as a core component

This determines whether Section 9 (GenAI and LLM Governance) is required or marked N/A.

If uncertain, ask: *"Does this model use a large language model (like Claude, GPT, or an open-source LLM) as part of its core logic, or is it a traditional statistical/ML model?"*

### Step 3: Work Through the MDD Section by Section

Work through sections in order: Document Control → Section 1 → Section 2 → ... → Section 12 → Appendices.

**For each section:**
- Read the corresponding **LLM Collaboration Guide** in `_mrm-mdd-kit/governance/how-to-fill-out.md` before drafting
- Infer everything you can from the codebase
- Ask the developer only for what you genuinely cannot infer
- Draft the section
- Ask the developer to review: *"Here's my draft for [Section X]. Does this accurately reflect the project? Anything to correct or add?"*
- Incorporate feedback and move on

**Do not ask for information you can get from the code.** If the algorithm is visible in the training script, don't ask what algorithm was used. If the output schema is defined in a Pydantic model, don't ask what the output format is. Reserve questions for business context, institutional knowledge, and decisions made outside the codebase.

**Do not draft sections you cannot ground in evidence.** If a section requires performance metrics and no experiment logs exist, say: *"I don't see performance results in the project artifacts. Can you share the evaluation output, or should I add a placeholder noting this needs to be completed?"* Never fabricate metrics.

### Step 4: Flag Governance Gaps

As you work through each section, maintain a running list of governance gaps — things that are required for a complete, regulatorily sound MDD but are missing from the project. At the end of the session, present these as a punch list.

Common gaps to watch for:
- No holdout / test set (data leakage risk)
- Prompts hardcoded rather than versioned (for LLM models)
- No output validation (for LLM models)
- LLM model version not pinned
- No experiment log
- Monitoring thresholds not defined
- Model owner not identified
- No compensating controls documented for known limitations

Present gaps as: *"I found the following governance gaps that should be addressed before this MDD is submitted for validation: [list]. Want to work through these now or flag them as open items?"*

### Step 5: Completion Check

Before declaring the MDD ready for validation review, verify:

- [ ] All REQUIRED sections are fully populated — no `[FILL IN: ...]` placeholders remaining
- [ ] All CONDITIONAL sections are either fully populated or marked `N/A — [reason]`
- [ ] Document Control block is complete with real names (not placeholders)
- [ ] Model risk rating (Section 1.2) is internally consistent with Sections 3.1–3.4
- [ ] Performance results (Section 6) are based on holdout set, not training set
- [ ] At least 3 assumptions in the assumptions registry (Section 7.1)
- [ ] Every High-sensitivity assumption has a compensating control
- [ ] Monitoring thresholds in Section 10.2 are numeric, not qualitative
- [ ] Approval sign-offs block is present (signatures to be collected outside this document)
- [ ] Appendix C (Change Log) has an initial entry

If any item is missing, surface it to the developer before marking the MDD complete.

---

## Section-Specific Guidance for Claude

### Document Control
Infer Model Name from the repo or README. Infer Model Type from the algorithm. Generate a placeholder Model ID in the format `MDL-[YYYY]-[NNN]` if not assigned. Ask for: Model Owner name and title, MRM contact, target Effective Date.

### Section 1: Executive Summary
Write the overview in plain English — no jargon. A business stakeholder who does not know ML should understand the model's role after reading 1.1. The risk rating in 1.2 must be consistent with the detailed assessment in Section 3 — complete Section 3 first, then populate 1.2. Never soften or omit risks in 1.3.

### Section 2: Model Purpose and Business Context
This is the most human-dependent section. Almost everything here requires developer input — the business problem, the prior process, the intended users, the downstream impact. Infer what you can from the README and problem statement. Ask for everything else. The prohibited uses in 2.4 must be specific and enforceable — vague language provides no governance protection.

### Section 3: Model Risk Assessment
This section determines validation rigor. Complete it carefully. The overall materiality in 3.4 must follow logically from 3.1–3.3 — do not assign a lower materiality to reduce compliance burden. State the validation rigor and monitoring frequency explicitly in 3.4 — these must align with what is written in Sections 10.1 and 10.3.

### Section 4: Data
Extract data sources from the data loading and pipeline code. Extract feature engineering from preprocessing scripts. Extract split methodology from the training code. Ask the developer to confirm: observation period justification, exclusion criteria rationale, and any data quality issues known but not visible in code.

### Section 5: Model Development
Extract algorithm, framework, and version directly from the code and dependency files. Extract hyperparameters from config files or training scripts. Extract developmental testing results from experiment logs or notebooks. Ask for: rationale for methodology selection, alternatives that were considered and rejected.

### Section 6: Model Performance
Never report training set performance as the primary result. Primary performance must be from the holdout / test set. If no holdout results exist in the project artifacts, flag this explicitly — do not fabricate numbers. Ask the developer to run final evaluation and share results before this section is completed.

### Section 7: Assumptions and Limitations
This section requires the most human-LLM collaboration. Claude can identify technical assumptions from the code and methodology. The developer must provide operational and business assumptions. Every limitation must be paired with either a compensating control or an explicit "Accepted" decision. Do not minimize or omit limitations — undisclosed limitations are a regulatory finding.

### Section 9: GenAI and LLM Governance (LLM models only)
Locate prompt templates in the codebase — typically in `src/prompts/` or similar. If prompts are hardcoded inline rather than stored as versioned files, flag this as a governance finding. Identify the LLM provider and model version from the code or config. If the model version is not pinned, flag this. Identify output validation logic (Pydantic models, schema validation). If none exists, flag this as a critical gap.

### Section 10: Validation and Monitoring Plan
Set monitoring thresholds based on development-time performance — not arbitrary round numbers. Warning threshold = development metric minus 1 standard deviation of CV performance. Action threshold = development metric minus 2 standard deviations. Always include PSI with standard thresholds (Warning > 0.10, Action > 0.25). Ask the developer to confirm the escalation path — who is notified, in what timeframe, what actions are available.

### Section 12: Governance
The Model Owner must be a named individual — not a team. The validator must be a different person from the lead developer. If both are the same, flag this as a separation of duties gap. Ask the developer for the model inventory system name and model ID.

---

## What a Complete, High-Quality MDD Looks Like

A complete MDD for this kit should:

- Be **specific throughout** — no vague language, no generic statements that could apply to any model
- Be **honest about limitations** — a document that lists every limitation with a compensating control is stronger than one that claims no limitations
- Be **internally consistent** — the risk rating, validation rigor, and monitoring frequency must all align
- Be **traceable** — every claim should be grounded in either the codebase or an explicit human-provided answer
- Be **structured for audit** — tables over prose wherever data is structured; numbered assumptions; versioned change log

Use the completed examples in `_mrm-mdd-kit/examples/` as calibration for depth, tone, and specificity. A completed MDD for a high-materiality model should be a substantive document — not a form with one-line answers.

---

## Note on SR 26-2 Scope and LLM Models

SR 26-2 explicitly excludes generative AI and agentic AI from its formal scope (Section II, footnote 3), noting they are "novel and rapidly evolving." However, SR 26-2 also states that "a banking organization's risk management and governance practices should guide the determination of appropriate governance and controls for any tools, processes, or systems not covered in this document."

When generating an MDD for an LLM-based model, document this scope clarification in Section 9.1 and frame the governance approach as applying SR 26-2 principles in the spirit of that guidance. This is the approach leading financial institutions are taking in practice, and it positions the institution well for future regulatory clarity on AI governance.
