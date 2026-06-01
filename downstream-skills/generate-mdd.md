# Skill: /generate-mdd

> **Installation note:** This skill is distributed with mrm-mdd-kit. Copy it to your project's `.claude/skills/` directory to enable the `/generate-mdd` command. If mrm-mdd-kit is installed as a submodule at `_mrm-mdd-kit/`, run:
> ```
> cp _mrm-mdd-kit/downstream-skills/generate-mdd.md .claude/skills/
> ```

Generates a complete, SR 26-2 compliant Model Development Document for this project through iterative human-LLM collaboration. Works through all 12 sections and appendices, inferring what it can from the codebase and asking targeted questions for what it cannot know.

---

## Step 0: Orient — read the codebase in parallel

Before writing a single line of the MDD, dispatch three parallel Explore agents to read the codebase simultaneously. This cuts orientation time and gives you richer context before drafting begins.

**Launch all three agents at once:**

- **Agent 1 — Model code:** Read the main training or inference script. Identify: algorithm/model type, framework and version, hyperparameters, output format, prediction target
- **Agent 2 — Data pipeline:** Read feature engineering, preprocessing, and data loading code. Identify: data sources, features used, split methodology, any exclusion criteria visible in code
- **Agent 3 — Config and experiments:** Read config files, experiment logs, notebooks, and dependency files (pyproject.toml, requirements). Identify: library versions, tracked metrics, development decisions, any held-out evaluation results

Also read:
- Project `README.md` — understand the model's purpose at a high level
- `_mrm-mdd-kit/governance/how-to-fill-out.md` — review the LLM Collaboration Guide for the sections you are about to write

**Do not begin drafting until you can answer:** *What does this model predict? What data does it use? What algorithm? What is the output format?*

---

## Step 1: Copy the template

Check if `MDD.md` already exists in the project root.

- If it does not exist, say: *"I'm going to create `MDD.md` in the project root by copying the mrm-mdd-kit template. This will be the living MDD for this project — committed and versioned alongside the code. Ready to proceed?"* Then copy `_mrm-mdd-kit/template/MDD.md` to `MDD.md`.
- If it already exists, say: *"I found an existing `MDD.md`. I'll work through it section by section and fill in or improve each section."*

---

## Step 2: Identify model type

Determine whether the model is:
- **Traditional ML** — statistical or ML model (regression, tree-based, neural network, etc.)
- **LLM-based** — uses a generative AI or large language model as a core component

This determines whether Section 9 (GenAI and LLM Governance) is required or marked N/A.

If uncertain, ask: *"Does this model use a large language model (like Claude, GPT, or an open-source LLM) as part of its core logic, or is it a traditional statistical/ML model?"*

---

## Step 3: Work through the MDD section by section

Work through sections in order: Document Control → Section 1 → Section 2 → ... → Section 12 → Appendices.

**For each section:**
1. Read the corresponding **LLM Collaboration Guide** in `_mrm-mdd-kit/governance/how-to-fill-out.md`
2. Infer everything you can from the codebase — do not ask for information you can get from the code
3. Ask the developer only for what you genuinely cannot infer (business context, stakeholder names, institutional details, compensating controls, dollar volumes)
4. Draft the section
5. Ask the developer to review: *"Here's my draft for [Section X]. Does this accurately reflect the project? Anything to correct or add?"*
6. Incorporate feedback and move to the next section

**Key rules:**
- Never fabricate metrics — if no holdout evaluation results exist, flag it and ask the developer to run evaluation first
- Never soften or omit risks — a document that lists every limitation is stronger than one that minimizes them
- Reserve questions for things that genuinely require human knowledge; do not ask about things visible in the code

---

## Step 4: Track and flag incomplete sections

As you work through each section, maintain a running list of sections that cannot be completed because the necessary project artifacts do not exist (no holdout results, no experiment logs, no stakeholder info provided, etc.).

At the end of the session, present this list as a punch list:
> *"The following sections required by the template could not be completed — the project artifacts needed to fill them don't exist yet: [list]. Want to work through these now or leave them as placeholders?"*

---

## Step 5: Run the completion check

Before declaring the MDD ready for validation review, check every item:

- [ ] All REQUIRED sections fully populated — no `[FILL IN: ...]` placeholders remaining
- [ ] All CONDITIONAL sections either fully populated or marked `N/A — [reason]`
- [ ] Document Control block complete with real names (not placeholders)
- [ ] Model risk rating (Section 1.2) is internally consistent with Sections 3.1–3.4
- [ ] Performance results (Section 6) are from the holdout set, not training set
- [ ] At least 3 assumptions in the assumptions registry (Section 7.1)
- [ ] Every High-sensitivity assumption has a compensating control
- [ ] Monitoring thresholds in Section 10.2 are numeric, not qualitative
- [ ] Approval sign-offs block is present
- [ ] Appendix C (Change Log) has an initial entry

If any item is missing, surface it before marking the MDD complete.
