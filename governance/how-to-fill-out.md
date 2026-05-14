# How to Fill Out the MDD Template

> **Who this guide is for:** Model developers, data scientists, and MRM teams filling out `template/MDD.md`. Also serves as structured instructions for LLMs working collaboratively with a human to generate an MDD from a project codebase.
>
> **How to use this guide:** Work through it section by section alongside the template. Each section below explains what regulators look for, what "good" looks like, what "insufficient" looks like, and how the human-LLM collaboration should work for that section.
>
> **Regulatory basis:** SR 26-2 (April 17, 2026). All section references map directly to SR 26-2 sections where applicable.

---

## How the Human-LLM Collaboration Works

Filling out an MDD with an LLM like Claude Code is not a "do it all for me" operation. It is an **iterative, back-and-forth collaboration** — more like working with a knowledgeable colleague than running a script.

The collaboration works because the two parties bring different and complementary knowledge:

| The LLM knows... | The human knows... |
|---|---|
| What the code does — algorithm, features, thresholds, output schema, data pipelines, dependencies | What the business process is — the workflow this model supports, what happened before it existed |
| What the regulatory requirements are — what SR 26-2 requires in each section | Who the stakeholders are — model owner, intended users, MRM contact |
| What "good" and "insufficient" look like — it has seen many MDDs and regulatory documents | What the institutional context is — portfolio size, dollar volumes, risk appetite |
| What questions to ask — it knows what it cannot infer from the codebase | What the compensating controls are — operational procedures not visible in code |
| How to draft — it can turn your answers into well-structured, compliance-grade prose | Whether the draft is accurate — only you can verify the business facts |

### The intended workflow

For each section of the MDD, the collaboration follows this pattern:

1. **LLM reads the codebase** and drafts what it can infer directly — algorithm used, feature list, output format, data sources referenced in code, performance metrics from experiment logs, etc.

2. **LLM asks targeted questions** for what it cannot know — business context, stakeholder names, institutional processes, dollar volumes, decisions made outside the codebase.

3. **Human answers** those specific questions. Answers can be brief — a few sentences or a list of facts. The LLM will shape them into compliant prose.

4. **LLM drafts the section** incorporating both inferred content and human-provided answers.

5. **Human reviews the draft** for accuracy. The human is the final authority on whether the content reflects reality — the LLM is the author, not the validator of facts.

6. **Iterate** — correct errors, add nuance, approve the section, move to the next one.

### Why this matters for governance

This iterative process is itself a **human-in-the-loop control**. The MDD that results is not an LLM artifact — it is a document that reflects human judgment at every section where business knowledge was required. That distinction matters to regulators: an MDD produced through genuine human-LLM collaboration has documented human accountability woven through it, not just a signature at the end.

Each section below identifies specifically what the LLM can infer vs. what requires a human answer, so the collaboration is efficient — the human is only asked for what the LLM genuinely cannot know.

---

---

## Document Control

### What this is
The document control block is the MDD's audit trail header. It answers: who built this, who owns it, what version is it, and what is its current status.

### What regulators look for
- A model ID that matches the model inventory entry — traceability from MDD to inventory is essential
- Clear identification of the model owner (business accountable party) vs. the developer (technical builder) — these must be different people
- A document status that accurately reflects where the MDD is in the approval lifecycle
- An effective date and next review date — models without a scheduled review are a governance gap

### What "good" looks like
Every field is populated. The Model Owner is a named business leader, not a team name. The Document Status is accurate (do not mark "Approved" until sign-offs are complete). The Next Review Date is within 12 months of the Effective Date for high-materiality models.

### What "insufficient" looks like
- Fields left blank or filled with "TBD"
- Model Owner listed as a team name rather than an individual
- Document Status marked "Approved" without completed sign-off signatures
- Missing or unrealistic Next Review Date

### Collaboration guide

**LLM infers from codebase:** Model Name (from repo name or README), Model Type (from algorithm/framework used), Lead Developer (from git history or README), Development Start Date (from earliest commit).

**LLM asks the human:**
- "What is the Model ID in your model inventory, or should I generate a placeholder?"
- "Who is the Model Owner — the business leader accountable for this model's performance?"
- "Who is the MRM contact for this model?"
- "What is the target Effective Date?"

**LLM drafts:** Populates all fields it can infer, inserts answers from the human, leaves Approval Sign-offs as `[AWAITING SIGNATURE]`, and sets Next Review Date to 12 months from Document Date unless told otherwise.

---

## Section 1: Executive Summary

### What this is
A non-technical, standalone summary of the model. The first and sometimes only section read by senior stakeholders, regulators, and MRM leadership.

### What regulators look for
Per SR 26-2 Section III: clear articulation of model purpose, a defensible materiality rating with evidence, and candid acknowledgment of key risks. Regulators flag Executive Summaries that are promotional rather than balanced.

### 1.1 Model Overview

**What "good" looks like:** 2–4 sentences that answer: (1) what does the model predict or produce, (2) what business decision does its output feed, (3) who uses the output, and (4) what data does it operate on. A non-technical reader should fully understand the model's role after reading this.

**What "insufficient" looks like:** Vague language ("this model uses advanced ML techniques to improve business outcomes"). No mention of what the output actually is or how it is used in practice.

**Collaboration guide:**

*LLM infers:* Model type and algorithm from the codebase, input features from the feature list or data pipeline, output type and format from the inference function or API definition.

*LLM asks the human:* "What is the plain-English description of what this model does and who uses its output? I can see it's a [model type] that predicts [target] — can you describe in one sentence how a business user or downstream system acts on its output?"

*LLM drafts:* 2–4 plain-English sentences combining inferred technical details with the human's business context answer. No jargon.

### 1.2 Model Risk Rating

**What "good" looks like:** Each of the three dimensions (Inherent Risk, Exposure, Purpose) is rated independently with a one-sentence rationale. The Overall Materiality reflects the combination — a model with High Exposure should rarely be rated Low Materiality even if Inherent Risk is Low.

**What "insufficient" looks like:** All three dimensions rated identically without differentiation. Overall Materiality inconsistent with the component ratings. No rationale provided.

**Collaboration guide:**

*LLM infers:* Inherent Risk from the algorithm complexity and codebase (simple regression = lower; black-box neural network = higher).

*LLM asks the human:*
- "What is the approximate portfolio size or dollar volume affected by this model's output?"
- "Are decisions made automatically from model output, or does a human review before acting?"
- "Does this model directly support regulatory reporting, credit decisions, or financial risk management?"

*LLM drafts:* Rates each dimension using the human's answers plus inferred complexity, with a one-sentence rationale per row. Flags any inconsistency between components and overall materiality.

### 1.3 Key Risks and Findings

**What "good" looks like:** 3–6 specific, honest bullets. Each bullet names a concrete risk, not a category. Reviewers should be able to find the full treatment of each bullet in Sections 7 or 10.

**What "insufficient" looks like:** Generic statements like "the model has some limitations" or "data quality may vary." No link between the risks listed here and the mitigations in later sections.

**Collaboration guide:**

*LLM infers:* Technical risks visible in the codebase — data period limitations, feature engineering choices, model complexity, any missing validation artifacts.

*LLM asks the human:* "Are there any business or operational risks I should know about that aren't visible in the code? For example, known data quality issues, a dependency on a vendor that's been unreliable, or a population the model hasn't been tested on?"

*LLM drafts:* One bullet per risk, combining inferred technical risks with human-provided operational risks. Each bullet names the risk, states the impact, and points to the section where it is fully treated.

### 1.4 Regulatory Applicability

**What "good" looks like:** Each regulation is explicitly marked Yes, No, or Conditional with a one-line rationale. This table should be completed even for regulations that do not apply — "No — this model does not affect credit decisions" is better than leaving a row blank.

**LLM Collaboration Guide:** Assess each regulation based on the model's use case:
- SR 26-2: Yes for any quantitative model at an institution >$30B assets; Yes by spirit of guidance for all others
- ECOA/Reg B: Yes if model output affects credit approval, pricing, or adverse action
- CECL/FASB ASC 326: Yes if model estimates expected credit losses or loan loss reserves
- BSA/AML: Yes if model flags suspicious transactions or supports SAR filing decisions
- CFPB UDAAP: Yes if model output affects consumer product access, pricing, or servicing

---

## Section 2: Model Purpose and Business Context

### What this is
The authoritative statement of what the model is authorized to do. Every subsequent section — data selection, methodology, testing rigor, validation scope — should be traceable back to what is written here.

### What regulators look for
Per SR 26-2 Section IV: a clear statement of purpose is the foundation of sound model development. Regulators look for specificity (not "improve risk management" but "score personal loan applications for probability of 90-day default within 12 months of origination"). They also look for explicit prohibited uses — a model used beyond its stated scope is a model risk finding.

### 2.1 Business Problem and Objectives

**What "good" looks like:** Describes the pre-model state (what happened before this model), the specific gap being addressed, and measurable objectives. References the business unit and process this model supports.

**What "insufficient" looks like:** Describes the model rather than the problem. ("This model uses XGBoost to predict fraud.") Missing context about why this problem matters to the institution.

**Collaboration guide:**

*LLM infers:* The technical problem being solved from the README, codebase, and any problem statement documentation in the repo.

*LLM asks the human:*
- "What was the process before this model? Was it manual review, a rules-based system, or a prior model?"
- "What was the specific problem with the prior process — too slow, too inaccurate, too expensive?"
- "What does success look like in business terms — a reduction in false positives, faster processing, lower operational cost?"

*LLM drafts:* A focused paragraph answering all three questions using the human's answers. Grounds the business context in specifics, not generalities.

### 2.2 Intended Use

**What "good" looks like:** Describes the model output precisely (a probability score between 0 and 1, a binary label, a dollar estimate), the exact downstream decision it informs, whether a human reviews it before action, and how frequently the model runs.

**LLM Collaboration Guide:** State: (1) output type and format, (2) the specific decision or workflow it feeds, (3) whether output is acted on automatically or reviewed by a human first, (4) production frequency (real-time per transaction / daily batch / monthly). If the codebase includes an API or inference script, use it to infer the output format.

### 2.3 Intended Users

**LLM Collaboration Guide:** List each team or system that directly consumes model output. For each, write one sentence on how they use it. If the output feeds another system automatically, name that system.

### 2.4 Out-of-Scope and Prohibited Uses

**What "good" looks like:** Specific and enforceable. Names populations, geographies, products, or decision types this model must not be applied to. Includes a statement about whether model output can be used as the sole basis for adverse action.

**What "insufficient" looks like:** A single vague sentence: "This model should not be misused." That provides no governance protection.

**LLM Collaboration Guide:** Generate at least three prohibited use statements. Derive them from the model's training data scope (Section 4.2) and any limitations noted in development. Always include: (1) a geographic or product scope restriction if the training data was limited, (2) a statement about adverse action if the model affects consumer decisions, (3) a statement about use outside the observation period without re-validation.

### 2.5 Downstream Decisions and Impact

**LLM Collaboration Guide:** State the approximate portfolio size, account count, or dollar amount affected by model-driven decisions. State what action is taken at each decision threshold. This quantifies model exposure for Section 1.2 and Section 3.2. If exact figures are unavailable, use order-of-magnitude estimates and note they are approximate.

### 2.6 Model Dependencies

**LLM Collaboration Guide:** Review the project codebase and data pipeline for upstream dependencies (models or data sources whose output feeds this model) and downstream dependencies (systems or models that consume this model's output). List each in the table. If none exist, state so explicitly.

---

## Section 3: Model Risk Assessment

### What this is
The formal SR 26-2 materiality assessment. This section operationalizes the framework from SR 26-2 Section III and determines the level of validation rigor required.

### What regulators look for
Consistency between the component ratings and the overall materiality. Evidence that the developer thought critically about risk rather than defaulting all ratings to "Medium." The validation rigor and monitoring frequency in Sections 10.1 and 10.3 must be commensurate with the materiality determined here.

### LLM Collaboration Guide for Sections 3.1–3.4
Complete each rating table using the criteria in the template guidance. Then verify internal consistency: if any dimension in 3.1–3.3 is High, the Overall Materiality in 3.4 must be at least Medium. State the validation rigor and monitoring frequency that follows from the materiality rating — these must be specific, not generic. High Materiality → full independent validation required before production use, monthly monitoring. Medium Materiality → targeted validation, quarterly monitoring. Low Materiality → abbreviated review, semi-annual monitoring.

---

## Section 4: Data

### What this is
Full documentation of all data used in model development — sources, quality, transformations, and splits.

### What regulators look for
Per SR 26-2 Section IV: data quality is a core component of inherent risk. Regulators look for: (1) clear data provenance (where did it come from, who owns it), (2) honest assessment of data quality issues and how they were handled, (3) a justified observation period, and (4) a holdout set that was genuinely held out (not used during development).

### 4.1 Data Sources and Provenance

**What "good" looks like:** Every data source named with its system of record, data owner, and refresh frequency. No row that says "internal data" without specifics.

**LLM Collaboration Guide:** Identify all data sources from the project's data loading code, configuration files, or data dictionaries. For each source, populate: source name, system of record (or "unknown — investigate"), data owner (team or role responsible), data type, and refresh frequency. Flag any source where provenance is unclear.

### 4.2 Observation Period and Sample Design

**What "good" looks like:** The observation period is justified — not just stated. A period that spans at least one economic cycle is preferable for credit models. Exclusion criteria are explicit and defensible.

**What "insufficient" looks like:** No justification for why the observation period was chosen. Exclusion criteria missing or described as "standard filters."

**LLM Collaboration Guide:** State the start and end date of the training data. Justify the period: does it include a stress period? Does it represent the current business environment? List every exclusion criterion applied and the reason for each exclusion.

### 4.3 Data Quality Assessment

**What "good" looks like:** A completed quality table with actual findings — not a table full of "No issues identified." Every significant quality issue should have a documented resolution.

**LLM Collaboration Guide:** For each quality dimension (completeness, accuracy, timeliness, consistency, representativeness), report the actual finding. If the project includes data quality checks or EDA notebooks, use them. If a dimension cannot be assessed from available materials, state "Not assessed — [reason]" rather than leaving it blank.

### 4.4 Feature Engineering and Preprocessing

**LLM Collaboration Guide:** Review the feature engineering code (preprocessing pipelines, feature store definitions, transformation scripts). For each transformation, state what was done and why. Pay particular attention to: imputation of missing values (state the method and rationale), encoding of categorical variables, any derived features that combine raw inputs, and normalization or scaling.

### 4.5 Train / Validation / Test Split

**What "good" looks like:** The holdout/test set was sealed during development and used only once for final evaluation. If a time-based split was used, the test set is more recent than the training set (out-of-time). The split rationale is explained.

**What "insufficient" looks like:** No holdout set. Test set used repeatedly during development (data leakage). Split method not documented.

**LLM Collaboration Guide:** Identify the split methodology from the training code. State sizes, date ranges, and whether stratification was applied. If there is evidence the holdout set was used more than once during development, flag this as a data leakage risk.

---

## Section 5: Model Development

### What this is
The technical record of how the model was built — the decisions made, the alternatives considered, and the evidence that supports those decisions.

### What regulators look for
Per SR 26-2 Section IV: sound model development "begins with a clear statement of purpose" and involves "constructive engagement around model design and assumptions." Regulators assess conceptual soundness — is the methodology appropriate for the problem? Were alternatives considered? Is the testing rigorous?

### 5.1 Problem Framing

**What "good" looks like:** Explains how the business problem was translated into a machine learning problem. The prediction target is defined precisely — not just "fraud" but "card-not-present transaction resulting in a confirmed fraud dispute within 90 days." The ground truth definition is documented and its quality assessed.

**LLM Collaboration Guide:** State the prediction target with its precise definition. State how ground truth labels were created (was there human labeling? outcome observation window? proxy variable?). Identify the key assumptions embedded in the problem framing — these are the assumptions that were made before any data was touched.

### 5.2 Methodology Selection

**What "good" looks like:** The chosen methodology is justified against its alternatives. The justification references the specific use case — interpretability requirements, regulatory acceptance, latency constraints, data size — not just "it performed best."

**LLM Collaboration Guide:** State the selected algorithm and framework with version. Write 2–3 sentences justifying the choice. List at least two alternatives that were evaluated and state specifically why each was not selected. If there is evidence from experiment logs or notebooks, reference it.

### 5.3–5.6 Architecture, Training, Hyperparameters, Developmental Testing

**LLM Collaboration Guide:** Extract all technical specifications directly from the codebase — model configuration files, training scripts, and hyperparameter search logs. For developmental testing, report cross-validation results (mean and standard deviation of the primary metric across folds). Flag any instability (high variance across folds) as a risk. Do not invent metrics — if a value is not available in the codebase, state "Not available in project artifacts."

---

## Section 6: Model Performance

### What this is
The empirical record of how the model performs on data it has never seen. All primary performance claims must be based on the holdout / test set.

### What regulators look for
Per SR 26-2 Section V: performance results should be reported honestly, including on material subgroups. Regulators flag documents that report only aggregate metrics and omit subgroup performance, particularly for consumer-facing models where fair lending may apply.

### 6.1–6.2 Metrics and Results

**What "good" looks like:** The primary metric was chosen based on the model's decision context — AUC for ranking models, precision/recall for classification where asymmetric error costs exist, RMSE for regression. Results are reported on training, validation, and holdout sets — all three.

**What "insufficient" looks like:** Only training set performance reported. Metric chosen because it looks best, not because it fits the use case. No discussion of the gap between training and holdout performance.

**LLM Collaboration Guide:** Report the primary metric on all three splits: training, validation, and holdout. If a gap exists between validation and holdout performance > 5 percentage points (or equivalent for the chosen metric), flag it as a potential overfitting concern. Justify the metric choice in terms of the business cost of false positives vs. false negatives.

### 6.3 Subgroup Performance

**What "good" looks like:** Performance is reported for every material subgroup — risk tiers, vintage, product type, and for consumer-facing models, demographic proxies. Significant performance disparities across subgroups are acknowledged and explained.

**LLM Collaboration Guide:** Identify the most material subgroups given the model's use case. For credit models: report by risk tier (low/medium/high score band) and loan vintage. For fraud models: report by transaction type and channel. For consumer-facing models: report by any available demographic proxy variables and flag for fair lending review.

### 6.4–6.6 Benchmarks, Sensitivity, Explainability

**LLM Collaboration Guide:** The benchmark must be a simpler model — logistic regression, decision tree, or the incumbent model being replaced. If no incumbent exists, use a logistic regression baseline. For sensitivity analysis, identify the top 5 features by SHAP value or feature importance and state how output changes when each is perturbed. For explainability, state whether individual predictions can be explained in plain language to a business user.

---

## Section 7: Assumptions and Limitations

### What this is
The honest accounting of what the model assumes and where it falls short. This section is routinely the weakest in developer-written MDDs — and the most scrutinized by regulators and validators.

### What regulators look for
SR 26-2 Section III explicitly requires understanding and communicating limitations. Regulators want specificity, candor, and evidence of risk awareness. A developer who lists every limitation and pairs it with a compensating control demonstrates maturity. A developer who lists none raises immediate questions.

### 7.1 Assumptions Registry

**What "good" looks like:** Every material assumption is in the table — not just methodological assumptions but data assumptions, problem framing assumptions, and deployment assumptions. High-sensitivity assumptions always have a compensating control. The validation evidence column is populated with actual evidence, not "assumption reviewed."

**What "insufficient" looks like:** Fewer than 3 assumptions listed for any non-trivial model. High-sensitivity assumptions with no compensating control. "N/A" in the validation evidence column.

**LLM Collaboration Guide:** Generate one row per material assumption. Derive assumptions from: (1) the problem framing (what must be true about the world for this model to be valid), (2) data choices (what must be true about the data quality and representativeness), (3) methodology (what statistical assumptions does the algorithm make), (4) deployment (what must be true about the production environment for model output to be reliable). Rate sensitivity High if violating the assumption would meaningfully degrade model performance or lead to materially wrong decisions.

### 7.2 Known Limitations

**What "good" looks like:** Concrete, specific limitations. Not "the model has some data limitations" but "the model was trained on data from 2019–2024; performance during interest rate environments above 6% has not been tested."

**LLM Collaboration Guide:** Derive limitations from: the observation period (regime risk), the training population (out-of-population risk), the features used (proxy variable risk), the algorithm (interpretability limits, distributional assumptions), and any data quality issues from Section 4.3. Do not soften limitations. A limitation marked "Accepted" is a legitimate outcome — what is not acceptable is an unacknowledged limitation.

### 7.3 Compensating Controls

**LLM Collaboration Guide:** For each limitation in 7.2 that is marked "Mitigated," write a specific compensating control. A control must name: who does what, how often, and what triggers escalation. Reference the monitoring threshold from Section 10.2 where applicable. Vague controls ("the team will monitor performance") do not qualify.

---

## Section 8: Third-Party and Vendor Components

### What regulators look for
Per SR 26-2 Section VII: the fact that a component is proprietary or vendor-supplied does not reduce the institution's model risk management obligations. Regulators expect evidence that vendor components were validated — not just accepted.

### LLM Collaboration Guide
Review the project's dependency files (requirements.txt, pyproject.toml, package.json) and configuration for any external APIs, pre-trained model weights, or licensed data sources. For each third-party component: document the vendor, version, and purpose. Flag any component where the institution does not have access to the underlying methodology as a validation constraint that must be addressed.

---

## Section 9: GenAI and LLM Governance

### What this is
A governance section specific to models that use generative AI or LLMs as a core component. Required because SR 26-2 explicitly excludes GenAI/LLMs from its scope while acknowledging that institutions must still govern them. This section applies SR 26-2 principles in the spirit of that guidance.

### Why this matters
Banks are actively deploying LLM-based models despite the regulatory gap. An institution that documents its LLM governance rigorously — prompt versioning, output validation, hallucination risk, human-in-the-loop controls — demonstrates sound practice and is far better positioned in a regulatory examination than one with no documentation at all.

### 9.1 LLM Model and Provider Details

**What "good" looks like:** Specific model name and version — not just "GPT-4" but the exact model ID pinned to a release. This matters because LLM providers update models; a governance document must be traceable to the specific model version in use.

**LLM Collaboration Guide:** Identify the LLM provider and model from the project's API configuration, environment variables, or model initialization code. Record the exact model ID and version. Note whether the model is accessed via API (vendor-hosted) or self-hosted. If the model version is not pinned in the codebase, flag this as a governance gap — unpinned model versions mean the model can change without a change management record.

### 9.2 Prompt Engineering Documentation

**What "good" looks like:** The system prompt and user prompt template are stored as versioned files in the repository — not hardcoded inline. Every variable injected at runtime is documented. The prompt includes explicit negative instructions (what the model must NOT do).

**What "insufficient" looks like:** Prompts hardcoded in application code with no version control. No documentation of the output schema the prompt enforces. Missing negative instructions.

**LLM Collaboration Guide:** Locate all prompt templates in the project codebase (typically in `src/prompts/` or equivalent). Document the file location, version, and structure. If prompts are hardcoded rather than stored as versioned files, flag this as a governance finding. Document the output schema — the exact JSON structure or label set the prompt instructs the LLM to return.

### 9.3 Output Validation and Error Handling

**LLM Collaboration Guide:** Identify the output validation logic in the codebase (Pydantic models, JSON schema validation, label set enforcement). Document what happens when the LLM returns output that fails validation. If no output validation exists, flag this as a critical gap — unvalidated LLM output feeding a downstream decision system is a model risk finding.

### 9.4 Hallucination and Error Risk

**LLM Collaboration Guide:** Assess hallucination risk based on the specific task. Classification tasks with a constrained label set carry lower hallucination risk than open-ended generation. For each identified error type, state the business impact of an undetected error and what design choice mitigates it. If the task involves extracting facts from documents, note retrieval hallucination as a specific risk.

### 9.5 Human-in-the-Loop Controls

**What "good" looks like:** A clearly defined human review layer with specific criteria for what gets escalated. The review rate is documented. Reviewers are trained on the model's limitations.

**What "insufficient" looks like:** "Outputs are reviewed by the team as needed." No review rate. No escalation criteria. No training documented.

---

## Section 10: Validation and Monitoring Plan

### What this is
The forward-looking governance plan — what must happen before the model goes live and how it is monitored after. This section bridges the developer's work and the independent validator's responsibilities.

### What regulators look for
Per SR 26-2 Section V: validation rigor must be commensurate with model materiality. Monitoring thresholds must be specific and actionable. Regulators flag plans with no numeric thresholds ("we will monitor performance") and plans where the escalation path is vague.

### 10.1 Pre-Production Validation

**LLM Collaboration Guide:** Populate the validation activities table based on the materiality rating from Section 3.4. High Materiality models require all activities completed before production. For any model deployed before validation is complete, add a row documenting the provisional use controls — what compensating controls are in place during the provisional period.

### 10.2 Monitoring Thresholds

**What "good" looks like:** Two thresholds per metric — a warning level that triggers investigation and an action level that triggers a defined response. Thresholds are set relative to development-time performance, not arbitrary round numbers.

**What "insufficient" looks like:** Single threshold with no defined action. Thresholds set to values that will never be breached given historical performance ranges.

**LLM Collaboration Guide:** Set thresholds as follows: Warning threshold = development performance minus 1 standard deviation of cross-validation performance (or minus 5% relative for simpler models). Action threshold = development performance minus 2 standard deviations (or minus 10% relative). Always include PSI (Population Stability Index) with standard thresholds: Warning > 0.10, Action > 0.25. For each action threshold breach, state the specific response required (not "review the model" but "MRM notified within 2 business days; model review convened within 10 business days").

### 10.5 Override and Adjustment Policy

**What "good" looks like:** A documented process with named roles, approval requirements, and an audit trail. The override rate is itself a monitored metric — a high override rate signals a model that the business no longer trusts.

**LLM Collaboration Guide:** State who can authorize an override, what documentation is required, and how overrides are logged. Add override rate to the monitoring table in 10.2 with a threshold (e.g., override rate > 15% triggers model review).

---

## Section 11: Implementation and Deployment

### LLM Collaboration Guide
Extract implementation details from the project codebase: (1) input/output specifications from the model's inference function signature or API endpoint definition, (2) latency requirements from any SLA documentation or load testing results, (3) integration dependencies from infrastructure configuration files or architecture diagrams. If the project has no production deployment yet, note the section as "Pre-production — to be completed prior to deployment approval."

---

## Section 12: Governance

### What regulators look for
Per SR 26-2 Section VI: clear roles and responsibilities with well-defined accountability. Regulators look for separation of duties — the developer and validator must be different individuals. The model owner must be a named individual, not a team.

### LLM instruction
Populate the roles table from project documentation, README, or code attribution. The Model Owner must be a named business leader — if unknown, flag as "[To be assigned — required before approval]". The MRM / Validator must be a different person from the Lead Developer — if the same person is listed for both, flag this as a separation of duties finding.

---

## Appendices

### Appendix A: Glossary
**LLM Collaboration Guide:** Populate the glossary with every technical term used in the MDD that a non-technical MRM reviewer might not know. Start from the terms already in the template and add any model-specific or domain-specific terms introduced in the document body.

### Appendix C: Model Change Log
**LLM Collaboration Guide:** Initialize with a single row for the initial release. After the model is approved, every material change — prompt update, feature addition, threshold change, recalibration — must be logged here with the date, author, type of change, and whether MRM approval was required.

### Appendix D: Experiment Log
**LLM Collaboration Guide:** Review the project's notebooks, experiment tracking logs (MLflow, W&B, or equivalent), or git history for evidence of model experiments conducted during development. Create one row per significant experiment. If no experiment log exists in the project artifacts, note "Experiment log not maintained during development — recommend establishing for future model iterations."

---

## Common Mistakes to Avoid

These are the most frequent deficiencies found in developer-written MDDs during MRM review:

1. **Listing limitations without compensating controls.** Every limitation needs either a control or an explicit "Accepted" decision with rationale.

2. **Reporting only training set performance.** Holdout performance is the only number that matters for an unbiased assessment. Training performance alone is not acceptable.

3. **Vague prohibited uses.** "Do not misuse this model" provides no governance protection. Be specific about populations, products, and decision types.

4. **No assumptions registry.** Assumptions listed in prose paragraphs are not searchable, not auditable, and not actionable. Use the table.

5. **Monitoring plan without numeric thresholds.** "We will monitor AUC quarterly" is not a monitoring plan. A monitoring plan specifies what triggers action and what the action is.

6. **Model Owner listed as a team.** The model owner must be an individual who can be held accountable for model performance and appropriate use.

7. **Missing experiment log.** Regulators expect evidence of an iterative, evidence-based development process. An undocumented development process suggests a lack of rigor, even when the final model is sound.

8. **For LLM models: prompts not versioned.** An unversioned prompt is an uncontrolled model component. Any change to the prompt changes the model — it must be tracked.
