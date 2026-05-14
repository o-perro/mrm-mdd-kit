# Model Development Document (MDD)

> **Kit version:** mrm-mdd-kit v1.0.0 | **Regulatory basis:** SR 26-2 (April 17, 2026)
>
> **How to use this template:**
> - Replace every `[FILL IN: ...]` placeholder with project-specific content.
> - Sections marked **REQUIRED** must be completed before submission for validation.
> - Sections marked **CONDITIONAL** are required only when the stated condition applies; mark as `N/A — [reason]` otherwise.
> - Sections marked **OPTIONAL** add depth for high-materiality models; include where possible.
> - Do not delete section headings — leave them with `N/A` if truly not applicable; this preserves auditability.

---

## Document Control

| Field | Value |
|---|---|
| **Model Name** | [FILL IN: descriptive name of the model] |
| **Model ID** | [FILL IN: unique identifier from model inventory, e.g. MDL-2026-001] |
| **Model Type** | [FILL IN: Traditional ML / Neural Network / LLM / Logistic Regression / Gradient Boosting / etc.] |
| **Model Version** | [FILL IN: e.g. 1.0.0] |
| **Document Version** | [FILL IN: e.g. 1.0] |
| **Document Status** | [FILL IN: Draft / Under Review / Approved / Retired] |
| **Development Start Date** | [FILL IN: YYYY-MM-DD] |
| **Document Date** | [FILL IN: YYYY-MM-DD] |
| **Effective Date** | [FILL IN: YYYY-MM-DD — date approved for use] |
| **Next Review Date** | [FILL IN: YYYY-MM-DD — typically 12 months from effective date] |
| **Model Owner** | [FILL IN: name and title of business owner accountable for model outcomes] |
| **Lead Developer** | [FILL IN: name and title] |
| **Contributing Developers** | [FILL IN: names and titles, or N/A] |
| **Primary Validator** | [FILL IN: name and title of independent validator — to be assigned] |
| **MRM Contact** | [FILL IN: name and title of Model Risk Management point of contact] |

### Approval Sign-offs

| Role | Name | Signature | Date |
|---|---|---|---|
| Lead Developer | [FILL IN] | __________ | [FILL IN: YYYY-MM-DD] |
| Model Owner | [FILL IN] | __________ | [FILL IN: YYYY-MM-DD] |
| MRM / Validation Lead | [FILL IN] | __________ | [FILL IN: YYYY-MM-DD] |
| Chief Risk Officer (or delegate) | [FILL IN] | __________ | [FILL IN: YYYY-MM-DD] |

### Document Version History

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| 1.0 | [FILL IN: YYYY-MM-DD] | [FILL IN: name] | Initial draft |

---

## Section 1: Executive Summary

> **REQUIRED.** Provide a concise, non-technical summary of the model. This section is read first by regulators, MRM leadership, and business stakeholders. It should stand alone — a reader who reads only this section should understand what the model does, what it's used for, and its primary risks.

### 1.1 Model Overview

[FILL IN: 2–4 sentences describing what the model does, what business decision or process it supports, and who uses its output. Avoid jargon. Example: "The [Model Name] is a [model type] model that [predicts / classifies / scores] [target variable] for [business process]. Its output is used by [team/system] to [decision or action]. The model operates on [data inputs] and produces [output format]."]

### 1.2 Model Risk Rating

| Dimension | Rating | Rationale |
|---|---|---|
| **Inherent Risk** | [Low / Medium / High] | [FILL IN: 1 sentence — complexity, assumptions, data quality] |
| **Model Exposure** | [Low / Medium / High] | [FILL IN: 1 sentence — portfolio size, business impact] |
| **Model Purpose** | [Low / Medium / High] | [FILL IN: 1 sentence — regulatory vs. operational use] |
| **Overall Materiality** | [Low / Medium / High / Critical] | [FILL IN: 1 sentence — combined assessment] |

> **Rating guidance:** Overall materiality is determined by the combination of inherent risk, exposure, and purpose per SR 26-2 Section III. High exposure or regulatory-facing purpose generally elevates materiality regardless of inherent risk.

### 1.3 Key Risks and Findings

[FILL IN: Bulleted list of 3–6 key risks, limitations, or findings that a reviewer should be aware of. These should be previewed here and fully documented in Sections 7 and 10. Example:
- The model was trained on data from [date range]; performance in market regimes outside this period is unknown.
- Feature X is sourced from a third-party vendor; data quality depends on vendor SLA adherence.
- The model has not been independently validated prior to initial deployment (provisional use — see Section 10.1).]

### 1.4 Regulatory Applicability

| Regulation | Applicable? | Notes |
|---|---|---|
| SR 26-2 — Model Risk Management | [Yes / No] | [FILL IN: applicability rationale] |
| ECOA / Reg B — Fair Lending | [Yes / No / CONDITIONAL] | [FILL IN: applicable if model output affects credit decisions] |
| CECL / FASB ASC 326 | [Yes / No / CONDITIONAL] | [FILL IN: applicable if model estimates expected credit losses] |
| BSA / AML — Bank Secrecy Act | [Yes / No / CONDITIONAL] | [FILL IN: applicable if model flags suspicious activity] |
| CFPB UDAAP | [Yes / No / CONDITIONAL] | [FILL IN: applicable if model output affects consumer-facing decisions] |
| Other: [FILL IN regulation name] | [Yes / No] | [FILL IN: notes] |

---

## Section 2: Model Purpose and Business Context

> **REQUIRED.** Documents why the model exists, what it is authorized to do, and the boundaries of its intended use. This section is the foundation for the entire MDD — every subsequent section (data choices, methodology, validation plan) should be traceable back to the model purpose defined here.

### 2.1 Business Problem and Objectives

[FILL IN: Describe the business problem this model solves. Include:
- What was happening before this model existed (manual process, no process, prior model)?
- What specific gap or inefficiency does the model address?
- What are the primary objectives? (e.g., reduce false positive rate in fraud detection, improve accuracy of credit loss estimates, automate complaint triage)
- What does success look like from a business perspective?]

### 2.2 Intended Use

[FILL IN: Describe precisely how the model output is used in practice:
- What is the model's output (score, label, probability, text, etc.)?
- What decision or workflow does the output feed into?
- Is model output used automatically (straight-through processing) or reviewed by a human before action is taken?
- What is the frequency of model use (real-time / batch / on-demand)?
- Who or what system consumes the output?]

### 2.3 Intended Users

[FILL IN: List the teams, roles, or systems that directly use model output. For each, describe how they use it.]

| User / System | How Output Is Used |
|---|---|
| [FILL IN: team or system name] | [FILL IN: description] |

### 2.4 Out-of-Scope and Prohibited Uses

[FILL IN: Explicitly state what this model must NOT be used for. This section protects the institution if the model is misapplied. Examples:
- This model must not be used to evaluate [population / geography / product] outside of [defined scope].
- This model must not be used as the sole basis for [adverse action / credit denial / regulatory filing] without human review.
- This model must not be applied to data collected outside of the observation period [start date] to [end date] without re-validation.]

### 2.5 Downstream Decisions and Impact

[FILL IN: Describe the downstream consequences of model output — what actions are taken based on model scores or classifications, and what populations or dollar amounts are affected. This is used to assess model exposure for the risk rating in Section 1.2.

Example: "Model scores are applied to a portfolio of approximately [N] accounts / [$ amount] in outstanding balances. A score above [threshold] triggers [action]. Approximately [X%] of accounts are affected by model-driven actions each [time period]."]

### 2.6 Model Dependencies

[FILL IN: Document any upstream or downstream model dependencies.]

| Dependency | Type | Description |
|---|---|---|
| [FILL IN: model name or data source] | [Upstream / Downstream] | [FILL IN: how this model depends on or feeds it] |

*If no dependencies exist, state: "This model has no upstream or downstream model dependencies."*

---

## Section 3: Model Risk Assessment

> **REQUIRED.** Provides the formal SR 26-2 risk assessment. This section operationalizes the materiality framework from SR 26-2 Section III and should be completed by the lead developer in consultation with MRM. The ratings here feed directly into the validation rigor required in Section 10.

### 3.1 Inherent Risk Assessment

| Risk Factor | Assessment | Rationale |
|---|---|---|
| **Model complexity** | [Low / Medium / High] | [FILL IN: simple regression vs. complex ensemble vs. black-box neural network] |
| **Number and criticality of assumptions** | [Low / Medium / High] | [FILL IN: few well-supported assumptions vs. many subjective ones] |
| **Input data quality** | [Low / Medium / High] | [FILL IN: clean, well-sourced data vs. noisy, incomplete, or proxied inputs] |
| **Data constraints / availability** | [Low / Medium / High] | [FILL IN: abundant labeled data vs. sparse or short history] |
| **Model interpretability** | [Low / Medium / High] | [FILL IN: fully interpretable vs. black-box; higher opacity = higher inherent risk] |
| **Novelty of approach** | [Low / Medium / High] | [FILL IN: well-established technique vs. novel or experimental] |
| **Overall Inherent Risk** | [Low / Medium / High] | [FILL IN: summary judgment] |

### 3.2 Model Exposure Assessment

| Exposure Factor | Value | Notes |
|---|---|---|
| **Portfolio / business scope** | [FILL IN: $ amount or count] | [FILL IN: what population or dollar amount is affected] |
| **Frequency of model use** | [FILL IN: real-time / daily / monthly] | |
| **Business impact of incorrect output** | [Low / Medium / High] | [FILL IN: financial loss, reputational risk, regulatory penalty] |
| **Degree of automation** | [Fully automated / Human-in-the-loop / Human-reviewed] | [FILL IN: more automation = higher exposure] |
| **Overall Exposure** | [Low / Medium / High] | [FILL IN: summary judgment] |

### 3.3 Model Purpose Assessment

| Purpose Factor | Assessment | Notes |
|---|---|---|
| **Regulatory-facing use** | [Yes / No] | [FILL IN: does this model directly support regulatory reporting or compliance?] |
| **Financial risk management use** | [Yes / No] | [FILL IN: does output feed into risk capital, reserving, or pricing decisions?] |
| **Consumer-impacting decisions** | [Yes / No] | [FILL IN: does output affect credit decisions, product access, or adverse actions?] |
| **Overall Purpose Risk** | [Low / Medium / High] | [FILL IN: regulatory or financial risk management use elevates to High] |

### 3.4 Overall Materiality Determination

| | |
|---|---|
| **Overall Materiality** | [Low / Medium / High / Critical] |
| **Validation rigor required** | [FILL IN: e.g., "Full independent validation required prior to production use" or "Targeted validation commensurate with medium materiality"] |
| **Monitoring frequency** | [FILL IN: e.g., Monthly / Quarterly / Annual] |
| **Rationale** | [FILL IN: 2–3 sentences summarizing why this materiality level was assigned] |

---

## Section 4: Data

> **REQUIRED.** Documents all data used in model development. Regulators and validators will scrutinize this section heavily — data quality problems are the most common source of model risk. Be specific about sources, time periods, exclusions, and quality issues.

### 4.1 Data Sources and Provenance

| Data Source | System of Record | Data Owner | Data Type | Refresh Frequency |
|---|---|---|---|---|
| [FILL IN: source name] | [FILL IN: system name] | [FILL IN: team or role] | [FILL IN: transactional / demographic / behavioral / external] | [FILL IN: real-time / daily / monthly] |

[FILL IN: For each data source, describe: what data it contains, why it was selected, and any known limitations or risks with this source.]

### 4.2 Observation Period and Sample Design

| | |
|---|---|
| **Observation period** | [FILL IN: start date to end date] |
| **Sample size** | [FILL IN: number of records / observations] |
| **Geographic scope** | [FILL IN: e.g., all U.S. accounts / specific states / etc.] |
| **Product scope** | [FILL IN: e.g., personal loans only, all card products, etc.] |
| **Exclusion criteria** | [FILL IN: what records were excluded and why] |
| **Rationale for observation period** | [FILL IN: why this time window was chosen; does it capture a full economic cycle?] |

### 4.3 Data Quality Assessment

[FILL IN: Document the data quality review performed. For each significant data quality issue found, describe the issue and how it was addressed.]

| Quality Dimension | Finding | Action Taken |
|---|---|---|
| **Completeness** | [FILL IN: % missing values by key field] | [FILL IN: imputation method, exclusion, or accepted as-is] |
| **Accuracy** | [FILL IN: cross-validation against source systems or known benchmarks] | [FILL IN: corrections applied or issues accepted] |
| **Timeliness** | [FILL IN: data lag, stale records] | [FILL IN: how lag was handled] |
| **Consistency** | [FILL IN: conflicting values across sources] | [FILL IN: resolution approach] |
| **Representativeness** | [FILL IN: does the sample reflect the target population?] | [FILL IN: adjustments or limitations noted] |

### 4.4 Feature Engineering and Preprocessing

[FILL IN: Describe all transformations applied to raw data before model training. For each transformation, explain the rationale.]

| Feature / Transformation | Description | Rationale |
|---|---|---|
| [FILL IN: feature or transformation name] | [FILL IN: what was done] | [FILL IN: why] |

*Include: encoding of categorical variables, normalization or scaling, treatment of outliers, handling of missing values, derived features, and any domain-specific transformations.*

### 4.5 Train / Validation / Test Split

| Split | Size | Time Period | Notes |
|---|---|---|---|
| **Training set** | [FILL IN: N records / X%] | [FILL IN: date range] | [FILL IN: any stratification or weighting applied] |
| **Validation set** | [FILL IN: N records / X%] | [FILL IN: date range] | [FILL IN: used for hyperparameter tuning and model selection] |
| **Test / holdout set** | [FILL IN: N records / X%] | [FILL IN: date range] | [FILL IN: held out; used only for final performance evaluation] |

[FILL IN: Describe the splitting methodology — random split, time-based split (out-of-time), or stratified split — and justify the choice in the context of the model's intended production use.]

### 4.6 Data Assumptions and Limitations

[FILL IN: List any significant assumptions made about the data and their implications. This feeds into Section 7.]

---

## Section 5: Model Development

> **REQUIRED.** Documents the technical decisions made during model development. The goal is to demonstrate that development was a deliberate, evidence-based process — not a trial-and-error black box. Validators will assess the conceptual soundness of these decisions against SR 26-2 Section IV.

### 5.1 Problem Framing and Conceptual Approach

[FILL IN: Describe how the business problem (Section 2.1) was translated into a machine learning problem. Address:
- What is the prediction target / label? How was it defined?
- Is this a classification, regression, ranking, or generation problem?
- What is the ground truth, and how reliable is it?
- What conceptual theory or domain knowledge underpins the model design?
- What are the key assumptions embedded in the problem framing itself?]

### 5.2 Methodology Selection

[FILL IN: Describe the modeling approach chosen and why it was selected over alternatives.]

**Selected approach:** [FILL IN: e.g., XGBoost gradient boosting classifier / logistic regression / fine-tuned LLM / feedforward neural network]

**Rationale for selection:** [FILL IN: explain why this methodology was chosen — performance, interpretability, regulatory acceptance, computational feasibility, etc.]

**Alternatives considered:**

| Alternative Approach | Reason Not Selected |
|---|---|
| [FILL IN: alternative 1] | [FILL IN: rationale] |
| [FILL IN: alternative 2] | [FILL IN: rationale] |

### 5.3 Model Architecture and Specifications

[FILL IN: Provide the technical specifications of the final model. The level of detail should match model complexity.]

| Specification | Value |
|---|---|
| **Algorithm / framework** | [FILL IN: e.g., XGBoost 2.0, scikit-learn 1.4, PyTorch 2.3] |
| **Number of features (inputs)** | [FILL IN] |
| **Output type** | [FILL IN: binary probability / multi-class label / continuous score / text] |
| **Output range** | [FILL IN: e.g., 0.0–1.0 probability / 300–850 score range] |
| **Decision threshold(s)** | [FILL IN: e.g., score ≥ 0.65 → [action]; justify threshold selection] |
| **Model size / complexity** | [FILL IN: e.g., 500 trees, max depth 6 / 4-layer network, 128 hidden units] |
| **Inference latency (if applicable)** | [FILL IN: e.g., <50ms p99 in production] |

### 5.4 Training Procedure

[FILL IN: Describe how the model was trained. Include:
- Loss function and why it was chosen
- Optimization algorithm and learning rate (if applicable)
- Regularization techniques and rationale
- Any data augmentation or resampling (e.g., SMOTE for class imbalance)
- Hardware and compute environment used for training
- Approximate training time and reproducibility (random seed, version pinning)]

### 5.5 Hyperparameter Selection and Calibration

[FILL IN: Document the hyperparameter tuning process.]

| Hyperparameter | Search Range | Final Value | Selection Method |
|---|---|---|---|
| [FILL IN: parameter name] | [FILL IN: range explored] | [FILL IN: final value] | [FILL IN: grid search / random search / Bayesian / manual] |

[FILL IN: Describe the tuning methodology (cross-validation strategy, early stopping criteria, metric optimized). If probability calibration was applied (e.g., Platt scaling, isotonic regression), document it here.]

### 5.6 Developmental Testing

[FILL IN: Describe the testing performed during development before the holdout set was evaluated. Include:
- Cross-validation results (mean ± std of primary metric across folds)
- Out-of-time testing results (if applicable)
- Stability of performance across data subgroups or time periods
- Any failure modes identified during development and how they were addressed

This section documents the developer's own quality assurance — distinct from independent validation in Section 10.]

---

## Section 6: Model Performance

> **REQUIRED.** Documents the empirical performance of the final model. All performance metrics should be computed on the holdout / test set (Section 4.5) unless otherwise noted. Report both aggregate performance and performance across material subgroups.

### 6.1 Evaluation Metrics and Rationale

[FILL IN: List the primary and secondary metrics used to evaluate model performance and explain why each was chosen given the model's purpose.]

| Metric | Type | Rationale |
|---|---|---|
| [FILL IN: e.g., AUC-ROC] | [Primary / Secondary] | [FILL IN: why this metric is appropriate for this use case] |
| [FILL IN: e.g., Precision @ threshold] | [Primary / Secondary] | [FILL IN] |
| [FILL IN: e.g., KS statistic] | [Primary / Secondary] | [FILL IN] |

### 6.2 Holdout Performance Results

| Metric | Training Set | Validation Set | Holdout / Test Set |
|---|---|---|---|
| [FILL IN: metric name] | [FILL IN] | [FILL IN] | [FILL IN] |
| [FILL IN: metric name] | [FILL IN] | [FILL IN] | [FILL IN] |

[FILL IN: Comment on the magnitude of any gap between training/validation performance and holdout performance. A large gap is a signal of overfitting and should be addressed.]

### 6.3 Performance Across Material Subgroups

[FILL IN: Report model performance disaggregated by material subgroups. The specific subgroups depend on the model type — for credit models, include performance by risk tier, product type, and vintage; for classification models, include performance by class; for fair lending-relevant models, include performance by protected class proxy variables.]

| Subgroup | N | [Primary Metric] | Notes |
|---|---|---|---|
| [FILL IN: subgroup name] | [FILL IN] | [FILL IN] | [FILL IN: any concerns or observations] |

### 6.4 Benchmark Comparisons

[FILL IN: Compare the selected model against at least one benchmark. A benchmark demonstrates that the chosen model adds value above a simpler alternative.]

| Model | [Primary Metric] | Notes |
|---|---|---|
| **Selected model** | [FILL IN] | |
| [FILL IN: benchmark — e.g., logistic regression baseline] | [FILL IN] | |
| [FILL IN: incumbent model, if replacing one] | [FILL IN] | |

### 6.5 Sensitivity Analysis

[FILL IN: Describe how model output responds to changes in key inputs. Address:
- Which features have the highest influence on model output? (feature importance, SHAP values, or equivalent)
- How does model output respond to plausible shifts in key input variables?
- Are there input ranges where the model behaves unexpectedly?

For LLM-based models: how does model output change with prompt variations, different input formats, or edge-case inputs?]

### 6.6 Explainability and Interpretability

[FILL IN: Document the approach used to explain individual predictions and overall model behavior. Address:
- What explainability method was used (SHAP, LIME, attention weights, coefficient analysis, etc.)?
- Can the model's key decision drivers be explained to a business user in plain language?
- Are there specific predictions that are difficult to explain? How are these handled?
- For regulatory-facing models: does the model support adverse action reason codes or similar regulatory explainability requirements?]

---

## Section 7: Assumptions and Limitations

> **REQUIRED.** This section is one of the most important in the MDD for regulatory review. Every model rests on assumptions; documenting them — and the controls that mitigate the risk when they break — demonstrates risk awareness and sound governance. Do not minimize or omit limitations.

### 7.1 Key Assumptions Registry

[FILL IN: List every material assumption embedded in the model — in problem framing, data, methodology, and deployment context.]

| # | Assumption | Sensitivity if Violated | Validation Evidence | Compensating Control |
|---|---|---|---|---|
| 1 | [FILL IN: e.g., "The relationship between features and the target is stable over time"] | [High / Medium / Low] | [FILL IN: how this assumption was tested or validated] | [FILL IN: e.g., quarterly performance monitoring with drift detection] |
| 2 | [FILL IN] | [High / Medium / Low] | [FILL IN] | [FILL IN] |

*Add rows as needed. Every High-sensitivity assumption must have a compensating control.*

### 7.2 Known Limitations

[FILL IN: Describe limitations inherent to the model that cannot be fully mitigated. Be direct — regulators value candor about limitations far more than a document that overstates model reliability.]

| Limitation | Impact | Status |
|---|---|---|
| [FILL IN: e.g., model trained on pre-2026 data; performance during rate shock environments is untested] | [FILL IN: potential impact on model output quality] | [Accepted / Mitigated — see control X / Under investigation] |

### 7.3 Compensating Controls for Known Limitations

[FILL IN: For each limitation where a compensating control exists, describe the control in detail. Reference the relevant monitoring threshold from Section 10.2 where applicable.

Example: "Limitation: Model may underperform during macroeconomic stress periods not represented in training data. Compensating control: Model output is reviewed by the [team] against macro indicators monthly. If [trigger condition], a model overlay is applied per the override policy in Section 10.5."]

### 7.4 Conditions Under Which the Model May Fail or Underperform

[FILL IN: Document specific scenarios or environmental conditions under which model performance is expected to degrade meaningfully. This is distinct from limitations — these are known failure modes.

Examples:
- Data distribution shift (input data that differs materially from training distribution)
- Sparse data populations (model may produce unreliable output for thin segments)
- Concept drift (the relationship between features and outcome changes over time)
- Adversarial inputs (for fraud or AML models: behavior designed to evade detection)
- Regime change (for financial models: market conditions outside the training period)]

### 7.5 Appropriate Use Boundaries

[FILL IN: Define the operating envelope within which the model is valid. Uses outside this envelope require MRM review before proceeding.

Examples:
- "This model is valid for [product type] originated between [date range] with [balance range]."
- "This model is not valid for accounts in bankruptcy or forbearance."
- "This model must not be applied to populations from [geography] without re-validation."]

---

## Section 8: Third-Party and Vendor Components

> **CONDITIONAL — Required if the model uses any third-party data, pre-trained model weights, external APIs, or vendor-supplied components. Mark N/A if the model is built entirely from internally developed code and first-party data.**
>
> SR 26-2 Section VII applies model risk management principles fully to vendor and third-party components.

### 8.1 Third-Party Components Inventory

| Component | Vendor / Source | Version | Type | Purpose |
|---|---|---|---|---|
| [FILL IN: component name] | [FILL IN: vendor name] | [FILL IN] | [Data / Pre-trained model / API / Library] | [FILL IN: how it is used] |

### 8.2 Vendor Validation Approach

[FILL IN: Describe how each third-party component was validated. Address:
- What documentation was received from the vendor (methodology, data dictionary, performance benchmarks)?
- What independent testing was performed on vendor output?
- Were proprietary components that could not be fully reviewed identified? How was that limitation managed?]

### 8.3 Customizations and Adjustments

[FILL IN: Document any customizations, calibrations, or adjustments made to vendor-supplied components to fit the institution's specific use case. For each customization, document the rationale and how the adjustment was validated.]

### 8.4 Ongoing Vendor Oversight

[FILL IN: Describe the process for monitoring vendor component quality over time:
- How are vendor model updates or data changes communicated to the institution?
- What triggers a re-validation of the vendor component?
- What is the fallback procedure if a vendor component becomes unavailable or underperforms?]

---

## Section 9: GenAI and LLM Governance

> **CONDITIONAL — Required for models that use generative AI, large language models, or agentic AI components. Mark N/A for traditional statistical or ML models.**
>
> **Regulatory Note:** SR 26-2 Section II explicitly excludes generative AI and agentic AI from its formal scope, noting they are "novel and rapidly evolving." However, SR 26-2 states that "a banking organization's risk management and governance practices should guide the determination of appropriate governance and controls for any tools, processes, or systems not covered in this document." This section applies SR 26-2 principles in the spirit of that guidance — a sound practice approach to LLM governance consistent with emerging industry standards.

### 9.1 LLM Model and Provider Details

| Field | Value |
|---|---|
| **LLM provider** | [FILL IN: e.g., Anthropic, OpenAI, Google, open-source] |
| **Model name and version** | [FILL IN: e.g., claude-sonnet-4-6, gpt-4o-2024-11-20] |
| **API or self-hosted** | [FILL IN: API / self-hosted / fine-tuned hosted] |
| **Fine-tuning applied** | [Yes / No — if Yes, describe in 9.5] |
| **Context window size** | [FILL IN: e.g., 200,000 tokens] |
| **Output format** | [FILL IN: structured JSON / free text / classification label] |

### 9.2 Prompt Engineering Documentation

[FILL IN: Document the prompting approach. Prompts are a core component of LLM-based models and must be version-controlled and documented like code.

- **System prompt:** [FILL IN: reference the versioned prompt file location, e.g., `src/prompts/classifier_v1.0.txt`]
- **User prompt template:** [FILL IN: describe the template structure; include the schema of variables injected at runtime]
- **Few-shot examples:** [FILL IN: number and description; confirm they use synthetic data only]
- **Output schema:** [FILL IN: define the expected JSON schema or label set for model output]
- **Prompt version:** [FILL IN: version tag for the prompt used in production]
- **Negative instructions:** [FILL IN: what the model is explicitly instructed NOT to do or return]]

### 9.3 Output Validation and Error Handling

[FILL IN: Describe how LLM output is validated before it is used downstream. Address:
- Is output validated against a schema (e.g., Pydantic model)?
- What happens when the LLM returns malformed, incomplete, or out-of-schema output?
- What is the retry and fallback logic?
- What is the escalation path when the LLM cannot produce a valid output?]

### 9.4 Hallucination and Error Risk

[FILL IN: Assess the risk that the LLM produces plausible-sounding but factually incorrect or fabricated output for this specific use case. Address:
- What types of errors or hallucinations are most likely given the task?
- What is the business impact of an undetected hallucination?
- What design choices were made to reduce hallucination risk (structured output, constrained prompts, retrieval augmentation, etc.)?
- What human or automated review catches hallucinations before they affect decisions?]

### 9.5 Human-in-the-Loop Controls

[FILL IN: Describe the human oversight layer for this LLM-based model. Address:
- Is LLM output acted upon automatically or reviewed by a human before action?
- What percentage of outputs are reviewed by humans? Is review sampled or comprehensive?
- What are the criteria for escalating a model output to human review?
- How are human reviewers trained on appropriate use and override criteria?]

### 9.6 Fine-Tuning Documentation

> **CONDITIONAL — Complete only if fine-tuning was applied to the base LLM.**

[FILL IN: Describe the fine-tuning process:
- What training data was used? Confirm it is synthetic or properly licensed (no real consumer data).
- What fine-tuning technique was used (full fine-tune, LoRA, RLHF, etc.)?
- How was the fine-tuned model evaluated vs. the base model?
- How is the fine-tuned model versioned and stored?]

---

## Section 10: Validation and Monitoring Plan

> **REQUIRED.** This section documents what must happen for the model to be approved for use and to remain approved over time. The developer completes this section; the independent validation team will verify and supplement it. SR 26-2 Section V governs the requirements addressed here.

### 10.1 Pre-Production Validation Requirements

[FILL IN: Describe the validation that must be completed before this model is approved for production use. Reference the materiality rating from Section 3.4 — high materiality models require full independent validation; lower materiality models may require targeted or abbreviated validation.]

| Validation Activity | Responsible Party | Required Before Production? | Status |
|---|---|---|---|
| Conceptual soundness review | Independent Validator | [Yes / No] | [Pending / In Progress / Complete] |
| Data quality independent assessment | Independent Validator | [Yes / No] | [Pending / In Progress / Complete] |
| Performance replication | Independent Validator | [Yes / No] | [Pending / In Progress / Complete] |
| Outcomes analysis | Independent Validator | [Yes / No] | [Pending / In Progress / Complete] |
| Ongoing monitoring plan review | MRM | [Yes / No] | [Pending / In Progress / Complete] |
| [FILL IN: any model-specific validation activity] | [FILL IN] | [Yes / No] | [Pending] |

*If the model is deployed before validation is complete (urgent business need per SR 26-2 Section V), document the provisional use controls here and in the approval sign-off.*

### 10.2 Ongoing Monitoring Metrics and Thresholds

[FILL IN: Define the specific metrics that will be monitored in production and the thresholds that trigger action. These thresholds should be set based on developmental performance — significant degradation from development-time performance is the signal.]

| Metric | Baseline (Development) | Warning Threshold | Action Threshold | Action Required |
|---|---|---|---|---|
| [FILL IN: e.g., AUC-ROC] | [FILL IN] | [FILL IN: e.g., < 0.72] | [FILL IN: e.g., < 0.68] | [FILL IN: e.g., MRM notification / model review] |
| [FILL IN: e.g., PSI — Population Stability Index] | [FILL IN: ~0.0 at baseline] | [FILL IN: > 0.10] | [FILL IN: > 0.25] | [FILL IN: e.g., data quality investigation / redevelopment review] |
| [FILL IN: e.g., Feature drift on top 5 inputs] | [FILL IN] | [FILL IN] | [FILL IN] | [FILL IN] |

### 10.3 Monitoring Frequency and Reporting

| Activity | Frequency | Owner | Report Delivered To |
|---|---|---|---|
| Performance monitoring report | [Monthly / Quarterly] | [FILL IN: team] | [FILL IN: MRM / Model Owner / Risk Committee] |
| Data quality check | [FILL IN] | [FILL IN] | [FILL IN] |
| Full model review | [Annual / Bi-annual] | [FILL IN] | [FILL IN] |

### 10.4 Performance Deterioration Triggers and Escalation

[FILL IN: Describe the escalation process when a monitoring threshold is breached. Who is notified, in what timeframe, and what decisions are made? Document the path from detection to resolution.

Example: "If the Action Threshold for [metric] is breached, the [team] notifies the Model Owner and MRM within [N business days]. MRM convenes a model review within [N days]. Possible outcomes: (1) model overlay applied, (2) model recalibration initiated, (3) model redevelopment initiated, (4) model use restricted pending review."]

### 10.5 Override and Adjustment Policy

[FILL IN: Document the policy for manual overrides or expert judgment adjustments to model output. Address:
- Under what circumstances can model output be overridden or adjusted?
- Who is authorized to approve an override?
- How are overrides documented (log, audit trail)?
- How are override rates monitored — and what rate triggers a review of model performance?]

### 10.6 Redevelopment Criteria

[FILL IN: Define the conditions that trigger model redevelopment (as distinct from recalibration or overlay):
- Performance falls below [threshold] for [N consecutive reporting periods]
- Significant data distribution shift that cannot be addressed through recalibration
- Material change in the business process, product, or regulatory environment the model supports
- Third-party component (Section 8) becomes unavailable or materially changed
- New regulatory guidance requires changes incompatible with the current model design]

---

## Section 11: Implementation and Deployment

> **REQUIRED for models moving to production. May be deferred for research or prototype-stage models — mark as "Deferred — not yet in production" in that case.**

### 11.1 Production Architecture

[FILL IN: Describe how the model is deployed and integrated into the production environment. Include a high-level architecture diagram or description covering: where the model runs (cloud / on-premises / vendor API), how inputs flow in, how outputs flow out, and what systems depend on model output.]

### 11.2 Input/Output Specifications

| Field | Description | Type | Format / Range | Required? |
|---|---|---|---|---|
| **Input: [field name]** | [FILL IN] | [FILL IN: string / float / int / etc.] | [FILL IN] | [Yes / No] |
| **Output: [field name]** | [FILL IN] | [FILL IN] | [FILL IN] | — |

[FILL IN: Document what happens when a required input is missing, out of range, or malformed. Is the model call rejected, does a default value substitute, or is a fallback model invoked?]

### 11.3 Performance and Latency Requirements

| Requirement | Target | Notes |
|---|---|---|
| **Inference latency (p50)** | [FILL IN: e.g., < 30ms] | |
| **Inference latency (p99)** | [FILL IN: e.g., < 100ms] | |
| **Throughput** | [FILL IN: e.g., 500 predictions/second] | |
| **Availability SLA** | [FILL IN: e.g., 99.9% uptime] | |

### 11.4 Integration Dependencies

| Dependency | System | Criticality | Fallback if Unavailable |
|---|---|---|---|
| [FILL IN: e.g., input data feed] | [FILL IN: system name] | [High / Medium / Low] | [FILL IN: what happens if this dependency fails] |

### 11.5 Rollback Plan

[FILL IN: Describe the process for rolling back to a prior model version or to a manual process if the deployed model must be taken offline. Include:
- What triggers a rollback decision?
- Who authorizes a rollback?
- How quickly can rollback be executed?
- What is the fallback process (prior model version / manual review / reject-all / approve-all)?]

---

## Section 12: Governance

> **REQUIRED.** Documents the governance structure for this model. Ensures accountability is clearly assigned throughout the model lifecycle per SR 26-2 Section VI.

### 12.1 Roles and Responsibilities

| Role | Name / Team | Responsibilities |
|---|---|---|
| **Model Owner** | [FILL IN] | Accountable for model performance and appropriate use; approves changes |
| **Lead Developer** | [FILL IN] | Responsible for model development, documentation, and initial testing |
| **MRM / Validator** | [FILL IN] | Independent validation; ongoing model risk oversight |
| **Business User(s)** | [FILL IN] | Consumes model output; responsible for appropriate use in business processes |
| **Data Owner** | [FILL IN] | Responsible for input data quality and access |
| **IT / MLOps** | [FILL IN] | Responsible for production deployment, monitoring infrastructure, and data pipelines |
| **Internal Audit** | [FILL IN] | Evaluates whether model risk management practices are rigorous and effective |

### 12.2 Model Inventory Registration

| Field | Value |
|---|---|
| **Model inventory system** | [FILL IN: name of model inventory tool or system] |
| **Model ID in inventory** | [FILL IN: same as Document Control block] |
| **Inventory registration date** | [FILL IN: YYYY-MM-DD] |
| **Tier in model inventory** | [FILL IN: e.g., Tier 1 / High Materiality] |

### 12.3 Change Management

[FILL IN: Describe the process for approving and documenting changes to the model after initial approval. Address:
- What constitutes a material change requiring full re-validation vs. a minor change requiring targeted review?
- Who approves material changes?
- How are changes logged in this document (see Appendix C)?]

### 12.4 Document Retention

[FILL IN: State the retention period for this document and all supporting artifacts (training data, code, model artifacts, validation reports). Reference the institution's records management policy as applicable.

Example: "This MDD and all supporting documentation shall be retained for a minimum of [7 years / per institutional policy] following model retirement."]

---

## Appendix A: Glossary

| Term | Definition |
|---|---|
| **AUC-ROC** | Area Under the Receiver Operating Characteristic Curve; measures discriminatory power of a binary classifier |
| **Conceptual soundness** | The degree to which a model's theory, design, and assumptions are appropriate for its intended use |
| **Effective challenge** | Critical analysis by objective experts with appropriate expertise, independence, and organizational standing (SR 26-2) |
| **Inherent risk** | The level of model risk arising from the model's design, assumptions, complexity, and data quality, independent of controls |
| **KS statistic** | Kolmogorov-Smirnov statistic; measures the separation between score distributions of two populations |
| **Model exposure** | The significance of model output to business decisions, often measured by portfolio size or business impact |
| **Model materiality** | The overall magnitude of model risk, combining inherent risk, model exposure, and model purpose |
| **Model purpose** | The nature and importance of a model's use — regulatory-facing uses carry higher purpose risk |
| **Overlay / adjustment** | Expert judgment applied to model output to account for known limitations or changed conditions |
| **PSI** | Population Stability Index; measures the shift in score distribution between development and production populations |
| **SR 26-2** | Supervisory Guidance on Model Risk Management, issued April 17, 2026 by the Fed, OCC, and FDIC |
| [FILL IN: additional terms] | [FILL IN: definition] |

---

## Appendix B: References and Source Documents

| Reference | Description | Location |
|---|---|---|
| SR 26-2 | Supervisory Guidance on Model Risk Management (April 17, 2026) | `governance/sr26-2.md` |
| [FILL IN: training data documentation] | [FILL IN] | [FILL IN: path or system] |
| [FILL IN: model code repository] | [FILL IN] | [FILL IN: repo URL and branch/tag] |
| [FILL IN: validation report, when complete] | [FILL IN] | [FILL IN] |
| [FILL IN: other reference] | [FILL IN] | [FILL IN] |

---

## Appendix C: Model Change Log

> Record every material change to the model or this document after initial approval. Minor edits (typos, formatting) do not require an entry.

| Version | Date | Author | Type of Change | MRM Approval Required? | Summary |
|---|---|---|---|---|---|
| 1.0 | [FILL IN: YYYY-MM-DD] | [FILL IN] | Initial release | N/A | |

---

## Appendix D: Experiment Log

> **OPTIONAL — Recommended for high-materiality models.** Summarize key experiments conducted during model development. This provides evidence of the iterative development process and supports conceptual soundness review.

| Experiment ID | Date | Hypothesis | Approach | Result | Decision |
|---|---|---|---|---|---|
| EXP-001 | [FILL IN: YYYY-MM-DD] | [FILL IN: what were you testing?] | [FILL IN: what did you try?] | [FILL IN: what did you observe?] | [FILL IN: what decision did this inform?] |
