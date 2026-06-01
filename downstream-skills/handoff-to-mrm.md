# Skill: /handoff-to-mrm

> **Installation note:** This skill is distributed with mrm-mdd-kit. Copy it to your project's `.claude/skills/` directory to enable the `/handoff-to-mrm` command. If mrm-mdd-kit is installed as a submodule at `_mrm-mdd-kit/`, run:
> ```
> cp _mrm-mdd-kit/downstream-skills/handoff-to-mrm.md .claude/skills/
> ```

Handles the template gap handoff when your model uses a pattern and algorithm combination that has no approved template in mrm-mdd-kit yet. Classifies MDD sections against existing approved templates, generates a candidate template, and opens a review PR in mrm-mdd-template-generator for the MRM team.

**When to use:** When you have confirmed your go-forward model type and algorithm, and you or Claude have determined that no approved template exists in `_mrm-mdd-kit/examples/` that covers your specific combination. Never invoke this speculatively — the developer must explicitly confirm the model before this skill runs.

---

## Step 1: Confirm before proceeding

Say exactly this before doing anything:

> *"I'm about to generate a candidate MDD template for [pattern/algorithm] and open a review PR in mrm-mdd-template-generator. This will notify the MRM team and start a formal review process.*
>
> *Before I proceed:*
> *1. Have you confirmed this is your go-forward model type and algorithm?*
> *2. Is there a tollgate or validation deadline I should include? (Used to prioritize the MRM review)*
>
> *Ready to proceed? (yes / no)"*

Do not proceed until the developer says yes.

---

## Step 2: Identify the closest approved template

Read the examples in `_mrm-mdd-kit/examples/` to find the closest match to this project's pattern and algorithm. List the available examples and identify which one is most structurally similar.

Available examples (read their `scenario.md` files for pattern/algorithm details):
- `examples/llm_complaint_classifier/` — LLM, text classification
- `examples/fraud_detection_xgboost/` — XGBoost, binary classification
- `examples/credit_scoring_random_forest/` — Random Forest, probability of default
- `examples/aml_monitoring_neural_network/` — Neural Network, anomaly detection
- `examples/cecl_logistic_regression/` — Logistic Regression, expected loss forecasting

---

## Step 3: Classify every MDD section

For each section in the template, classify it as one of:

- **INHERITED** — the section content from the closest approved template applies without modification to this pattern and algorithm. No MRM review needed.
- **AMENDED** — the section requires changes from the approved template to accurately document this pattern/algorithm combination. MRM review required.
- **NET NEW** — no equivalent section exists in any approved template. This section is entirely new. MRM review required.

Read the full template at `_mrm-mdd-kit/template/MDD.md` and the closest approved example. Classify every section. Where content differs due to the algorithm or pattern, explain why in a brief inline comment.

---

## Step 4: Generate the candidate template

Create a candidate template file at `_mrm-mdd-kit/incoming/[pattern]_[algorithm]_candidate.md` (using kebab-case, e.g., `credit-risk_neural-network_candidate.md`).

**Document header:**
```
# Model Development Document
## [PATTERN] — [ALGORITHM]
## STATUS: CANDIDATE TEMPLATE — PENDING MRM APPROVAL
## Generated: [DATE]
## Requested by: [PROJECT REPO NAME]
## MRM Review Required: Sections [LIST]
```

**For INHERITED sections:** Render normally. Add this comment above the section:
```
<!-- INHERITED from [SOURCE EXAMPLE] — no review required -->
```

**For AMENDED sections:** Add a flag at the section header:
```
## [SECTION TITLE] ⚠️ AMENDED — MRM REVIEW REQUIRED
<!-- AMENDED from [SOURCE EXAMPLE] -->
<!-- CHANGES: [brief description of what was amended and why] -->
```
Where content differs from the source, add inline:
```
<!-- CHANGED: [original language] → [amended language] because [rationale] -->
```

**For NET NEW sections:** Add a flag at the section header:
```
## [SECTION TITLE] ⚠️ NET NEW — MRM REVIEW REQUIRED
<!-- NET NEW — no equivalent section exists in any approved template -->
<!-- Rationale: [why this section is required for this pattern/algorithm] -->
```

---

## Step 5: Build the PR payload

Prepare the structured payload for the PR body. This is what mrm-mdd-template-generator's GitHub Actions and MRM reviewers use to understand scope at a glance.

```
## Handoff Request

**Pattern:** [pattern]
**Algorithm:** [algorithm]
**Requesting project:** [repo name or URL]
**Developer:** [GitHub username]
**Tollgate date:** [date or "Not specified"]
**Submitted:** [today's date]

## Section Classification

| Classification | Count | Sections |
|---|---|---|
| Inherited (no review needed) | [N] | [list] |
| Amended (MRM review required) | [N] | [list] |
| Net New (MRM review required) | [N] | [list] |

**Total MRM review scope:** [N] sections

## Closest approved template used as basis
[example name and path]

## Notes
[Any context MRM should have — unusual aspects of this model, regulatory considerations, tight timeline, etc.]
```

---

## Step 6: Open the PR

Use the GitHub MCP to open a pull request in `mrm-mdd-template-generator`:

- **Title:** `Template request: [pattern] — [algorithm]`
- **Base branch:** `main`
- **Body:** The structured payload from Step 5
- **Files:** The candidate template from Step 4

After opening the PR, report to the developer:
> *"Handoff PR opened: [PR URL]*
>
> *The MRM team has been notified. They will review the [N] amended/net-new sections and either approve the template or request revisions. SLA: [HIGH/STANDARD/LOW] priority based on your tollgate date.*
>
> *You can continue building your model. If MRM requests changes, they will comment on the PR. You don't need to wait for approval to continue working — the candidate template is available in `_mrm-mdd-kit/incoming/` in the meantime."*
