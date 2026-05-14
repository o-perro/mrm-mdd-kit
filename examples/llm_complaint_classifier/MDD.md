# Model Development Document (MDD)

> **Kit version:** mrm-mdd-kit v1.0.0 | **Regulatory basis:** SR 26-2 (April 17, 2026)
>
> **FICTIONAL EXAMPLE** — All institution names, personnel, performance metrics, and business details are completely made up. This document exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## Document Control

| Field | Value |
|---|---|
| **Model Name** | Customer Contact Complaint Classifier |
| **Model ID** | MDL-2026-041 |
| **Model Type** | LLM-Based Binary Classifier |
| **Model Version** | 1.0.0 |
| **Document Version** | 1.2 |
| **Document Status** | Approved |
| **Development Start Date** | 2026-01-06 |
| **Document Date** | 2026-04-01 |
| **Effective Date** | 2026-05-01 |
| **Next Review Date** | 2027-05-01 |
| **Model Owner** | Sarah Kowalski, SVP Customer Experience Operations |
| **Lead Developer** | Marcus Chen, Senior Data Scientist, Model Risk & Analytics |
| **Contributing Developers** | Priya Nair, Data Engineer, Model Risk & Analytics |
| **Primary Validator** | James Whitfield, Senior Model Validator |
| **MRM Contact** | Diana Flores, Director of Model Risk Management |

### Approval Sign-offs

| Role | Name | Signature | Date |
|---|---|---|---|
| Lead Developer | Marcus Chen | *(signed)* | 2026-04-01 |
| Model Owner | Sarah Kowalski | *(signed)* | 2026-04-03 |
| MRM / Validation Lead | James Whitfield | *(signed)* | 2026-04-14 |
| Chief Risk Officer (or delegate) | Robert Anand, Deputy CRO | *(signed)* | 2026-04-17 |

### Document Version History

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| 1.0 | 2026-03-10 | Marcus Chen | Initial draft |
| 1.1 | 2026-03-24 | Marcus Chen | Incorporated MRM pre-review feedback; expanded Section 7 and Section 9 |
| 1.2 | 2026-04-01 | Marcus Chen | Final pre-approval revisions; updated performance results with holdout set |

---

## Section 1: Executive Summary

### 1.1 Model Overview

The Customer Contact Complaint Classifier is an LLM-based binary classification model that reads the full text of inbound customer contacts — email messages, live chat transcripts, and ASR-generated phone transcripts — and classifies each as either a regulatory complaint or a non-complaint according to the CFPB definition of a consumer complaint. Its output feeds directly into Pinnacle Community Bank's complaint management system, Resolve360, where a human complaint specialist completes formal intake for every AI-identified complaint before it enters the regulated CFPB workflow. The model operates in near-real-time, processing each contact within 30 seconds of receipt across all three inbound channels.

### 1.2 Model Risk Rating

| Dimension | Rating | Rationale |
|---|---|---|
| **Inherent Risk** | Medium | LLM-based approach introduces output uncertainty not present in traditional classifiers; however, the constrained binary output schema and structured prompt significantly limit failure modes. The task is well-defined and the label set is closed. |
| **Model Exposure** | Medium | Approximately 85,000 contacts processed monthly. Missed complaints (false negatives) carry regulatory exposure; false positives create operational inefficiency but are caught by human review before regulatory filing. |
| **Model Purpose** | High | Directly supports CFPB regulatory compliance. Missed complaints can result in CFPB examination findings, consent orders, and civil money penalties. |
| **Overall Materiality** | High | High purpose risk elevates overall materiality despite medium inherent risk and exposure. Full independent validation required prior to production use. |

### 1.3 Key Risks and Findings

- **False negative risk (missed complaints):** The model achieves 88.3% recall on the holdout set, meaning approximately 11.7% of actual complaints are not identified by the model. A sampling-based human review control (Section 10.5) partially mitigates this, but some missed complaints may not be identified until the next scheduled audit. Full treatment in Sections 7.2 and 10.2.
- **LLM output variability:** Unlike traditional classifiers, LLM output can vary with identical inputs across API calls due to temperature and model-side sampling. The model is configured with temperature=0 to minimize this, but residual variability has been observed in edge cases. Full treatment in Section 9.4.
- **LLM provider dependency:** The model depends on a third-party LLM API. Service degradation, model version changes, or provider policy changes can affect model behavior without a change management record. Addressed via version pinning and fallback procedure in Sections 9.1 and 11.4.
- **Prompt version control:** A prior version of the model used hardcoded prompts in application code. Version 1.0.0 moves all prompts to versioned files in `src/prompts/`. Any future prompt change requires a model change record in Appendix C. Full treatment in Section 9.2.
- **Regulatory scope gap:** SR 26-2 explicitly excludes generative AI from its formal scope. This MDD applies SR 26-2 principles in the spirit of that guidance. This approach is noted and disclosed in Section 9.1.

### 1.4 Regulatory Applicability

| Regulation | Applicable? | Notes |
|---|---|---|
| SR 26-2 — Model Risk Management | Yes — by spirit of guidance | SR 26-2 Section II explicitly excludes GenAI from formal scope. This model is governed under SR 26-2 principles as sound practice per the guidance's own instruction that institutions should apply risk management practices to tools outside the formal scope. |
| ECOA / Reg B — Fair Lending | No | Model classifies contacts, not credit applicants. Does not affect credit decisions. |
| CECL / FASB ASC 326 | No | Not a credit loss estimation model. |
| BSA / AML — Bank Secrecy Act | No | Not a transaction monitoring model. |
| CFPB UDAAP | Yes | Model output directly determines which customer contacts enter the CFPB complaint workflow. Missed complaints (false negatives) represent potential UDAAP exposure. |

---

## Section 2: Model Purpose and Business Context

### 2.1 Business Problem and Objectives

Pinnacle Community Bank is required under CFPB supervisory guidance to identify, acknowledge, log, and resolve all consumer complaints within defined timeframes. A "complaint" is defined by the CFPB as any oral or written expression of dissatisfaction — whether or not the customer uses the word "complaint" — where the customer expects a response or resolution.

Before this model, Pinnacle's complaint identification relied on a manual sampling process: 14 quality analysts reviewed a random 12% sample of all inbound contacts. Contacts outside the sample were reviewed only if a frontline agent manually escalated them. This approach had two material deficiencies: (1) an estimated 6–9% of non-sampled contacts containing complaints went unidentified each month based on internal audit extrapolation, creating CFPB regulatory exposure; and (2) the average time from contact receipt to complaint queue entry was 3.5 business days, creating acknowledgment timeline risk.

The model's objectives are: (1) achieve 100% coverage of inbound contacts for complaint identification, eliminating the sampling gap; and (2) reduce the average time from contact receipt to complaint queue entry to under one business day. Secondary objective: maintain a false positive rate below 15% to avoid overwhelming the complaint specialist team with non-complaint reviews.

### 2.2 Intended Use

The model receives the full text of each inbound customer contact (email body, chat transcript, or phone ASR transcript) and returns a structured JSON response containing: a binary classification label (`complaint` or `non_complaint`), a confidence score (0.0–1.0), and up to three plain-English reason codes explaining the classification. Contacts labeled `complaint` with confidence ≥ 0.75 are automatically queued in Resolve360 for human specialist review. Contacts labeled `complaint` with confidence < 0.75 are flagged for priority human review before queuing. All contacts labeled `non_complaint` are sampled at a 5% rate for ongoing quality monitoring. The model runs in near-real-time; each contact is processed within 30 seconds of receipt.

### 2.3 Intended Users

| User / System | How Output Is Used |
|---|---|
| Resolve360 (complaint management system) | Receives structured model output via API; queues complaint-labeled contacts for specialist intake |
| Complaint Specialists (Customer Experience Ops) | Reviews every AI-identified complaint before formal CFPB filing; completes intake form |
| Quality Monitoring Team | Reviews sampled non-complaint contacts to track false negative rate |
| Diana Flores, MRM | Reviews monthly model performance report |

### 2.4 Out-of-Scope and Prohibited Uses

- This model must not be used as the sole basis for determining whether a customer contact is a regulatory complaint. Every contact classified as a complaint by the model must be reviewed by a human complaint specialist before entering the formal CFPB workflow.
- This model must not be applied to contact text in languages other than English without re-validation on multilingual data.
- This model must not be used to classify contacts from business or commercial accounts; it was trained on retail and small business consumer contacts only.
- This model must not be used to classify contacts received prior to January 1, 2024 (outside the training observation period) for backfill purposes without MRM review.
- Model output must not be shared externally or with third parties.

### 2.5 Downstream Decisions and Impact

Model output drives the entry of customer contacts into the formal CFPB complaint workflow. Approximately 85,000 contacts are processed monthly. Based on the historical complaint rate of approximately 3.2%, the model is expected to identify approximately 2,700 complaints per month for specialist review. Each missed complaint (false negative) represents a potential CFPB regulatory finding if discovered during examination. Pinnacle's internal compliance team has assessed that a sustained false negative rate above 15% would be a material compliance risk requiring immediate remediation.

### 2.6 Model Dependencies

| Dependency | Type | Description |
|---|---|---|
| Third-party LLM API (see Section 9.1) | Upstream | Core model component — contact text is sent to the LLM API for classification |
| ASR Transcription Service (VoiceCapture Pro) | Upstream | Generates phone transcripts; transcript quality directly affects model performance on the phone channel |
| Resolve360 API | Downstream | Receives model output and queues complaint-labeled contacts |

---

## Section 3: Model Risk Assessment

### 3.1 Inherent Risk Assessment

| Risk Factor | Assessment | Rationale |
|---|---|---|
| **Model complexity** | Medium | LLM-based classification is more complex than a traditional ML classifier, but the task is constrained to a binary output with a defined schema. Complexity is bounded by the structured prompt. |
| **Number and criticality of assumptions** | Medium | Key assumptions include: (1) LLM accurately interprets CFPB complaint definition as specified in the prompt; (2) ASR transcript quality is sufficient for classification; (3) contact text is in English. Each carries medium sensitivity. |
| **Input data quality** | Medium | Email and chat text is generally clean. Phone ASR transcripts have a word error rate of approximately 8%, which can affect classification on ambiguous contacts. |
| **Data constraints / availability** | Low | 14 months of labeled training data (January 2024 – February 2026) with 42,800 labeled contacts across all three channels provides a robust training corpus. |
| **Model interpretability** | Medium | Reason codes in the output schema provide human-readable explanation for each classification. However, the LLM's internal reasoning is not fully transparent. |
| **Novelty of approach** | Medium | LLM-based complaint classification is a newer approach; limited published benchmarks exist for this specific regulatory use case. |
| **Overall Inherent Risk** | Medium | Well-defined task with constrained output partially offsets LLM complexity and novelty. |

### 3.2 Model Exposure Assessment

| Exposure Factor | Value | Notes |
|---|---|---|
| **Portfolio / business scope** | ~85,000 contacts/month | Covers 100% of inbound retail and small business contacts |
| **Frequency of model use** | Near-real-time | Processes each contact within 30 seconds of receipt |
| **Business impact of incorrect output** | High (false negatives) / Low (false positives) | False negatives carry CFPB regulatory risk; false positives create minor operational overhead caught by human review |
| **Degree of automation** | Human-in-the-loop | Every AI-identified complaint is reviewed by a human before formal CFPB filing |
| **Overall Exposure** | Medium | High volume partially offset by mandatory human review layer before regulatory action |

### 3.3 Model Purpose Assessment

| Purpose Factor | Assessment | Notes |
|---|---|---|
| **Regulatory-facing use** | Yes | Output directly determines entry into CFPB regulated complaint workflow |
| **Financial risk management use** | No | Does not feed into financial risk capital, reserving, or pricing |
| **Consumer-impacting decisions** | Yes | Missed complaints may result in consumers not receiving required regulatory acknowledgment and resolution |
| **Overall Purpose Risk** | High | Regulatory compliance use with direct consumer impact |

### 3.4 Overall Materiality Determination

| | |
|---|---|
| **Overall Materiality** | High |
| **Validation rigor required** | Full independent validation required prior to production use, including conceptual soundness review, data quality assessment, performance replication, and outcomes analysis plan |
| **Monitoring frequency** | Monthly |
| **Rationale** | High purpose risk (CFPB regulatory compliance, direct consumer impact) drives High overall materiality despite medium inherent risk and exposure. The potential for regulatory findings from missed complaints requires the highest level of governance rigor. |

---

## Section 4: Data

### 4.1 Data Sources and Provenance

| Data Source | System of Record | Data Owner | Data Type | Refresh Frequency |
|---|---|---|---|---|
| Customer Email Archive | Salesforce Service Cloud | Customer Experience Ops | Inbound email text | Real-time |
| Live Chat Transcript Archive | Zendesk | Digital Banking | Chat session transcript | Real-time |
| Phone ASR Transcript Archive | VoiceCapture Pro / S3 | Contact Center Ops | ASR-generated phone transcript | Near-real-time (15-min lag) |
| Complaint Labels (historical) | Resolve360 | Compliance | Binary complaint / non-complaint label | Historical extract |

Historical labeled data was extracted from Resolve360, which contains all contacts formally classified as complaints by the quality analyst team from January 2024 onward. Non-complaint labels were assigned to all contacts in the same period that were reviewed by quality analysts and not flagged as complaints.

### 4.2 Observation Period and Sample Design

| | |
|---|---|
| **Observation period** | January 1, 2024 – February 28, 2026 (14 months) |
| **Sample size** | 42,847 labeled contacts (7,621 complaint, 35,226 non-complaint) |
| **Geographic scope** | All seven Pinnacle operating states |
| **Product scope** | Retail deposit, retail credit, small business deposit, and small business credit contacts. Excludes commercial contacts (separate team, different contact handling). |
| **Exclusion criteria** | Contacts with fewer than 10 words of substantive text excluded (1,204 records, primarily misdirected or blank submissions). Contacts marked as duplicate submissions excluded (387 records). |
| **Rationale for observation period** | January 2024 represents the date Pinnacle implemented Resolve360 with consistent complaint labeling. Earlier data used a legacy system with inconsistent label definitions. The 14-month period captures seasonal variation across two calendar years and reflects the current product mix. |

### 4.3 Data Quality Assessment

| Quality Dimension | Finding | Action Taken |
|---|---|---|
| **Completeness** | Email (98.4% complete), Chat (99.1% complete), Phone ASR (94.7% complete — 5.3% of transcripts had >20% blank segments due to ASR confidence threshold failures) | Phone contacts with >20% blank segments excluded from training (847 records). A note was added to limitations (Section 7.2). |
| **Accuracy** | Manual spot-check of 500 randomly sampled labeled contacts against original contact text confirmed 96.2% label accuracy. Primary labeling errors: ambiguous contacts where the customer expressed dissatisfaction but did not request resolution. | Label discrepancies reviewed by compliance team; 19 records relabeled. No systematic labeling error identified. |
| **Timeliness** | All data is historical; no timeliness concern for training. In production, phone ASR transcripts have a 15-minute processing lag. | Production pipeline configured to process phone contacts on 15-minute delay. No impact on complaint identification SLA. |
| **Consistency** | Label definitions changed slightly in August 2024 when Pinnacle updated its internal complaint policy to align more closely with the CFPB definition. Pre-August 2024 labels may use the prior, slightly narrower definition. | Compliance team reviewed and relabeled a random sample of 1,000 pre-August 2024 contacts using the updated definition. No significant systematic difference found; all pre-August data retained. |
| **Representativeness** | Complaint rate varies by channel: email (4.1%), chat (3.8%), phone (2.4%). Class imbalance is present (~17.8% complaint rate in the labeled set due to analyst sampling bias toward complaint-likely contacts). | Class imbalance addressed via prompt instruction and confidence threshold calibration rather than resampling, given the LLM-based approach. |

### 4.4 Feature Engineering and Preprocessing

| Feature / Transformation | Description | Rationale |
|---|---|---|
| Contact text normalization | Leading/trailing whitespace removed; null/empty strings filtered | Prevents API errors and ensures clean input to the LLM |
| Channel tag injection | Contact text prefixed with `[CHANNEL: EMAIL]`, `[CHANNEL: CHAT]`, or `[CHANNEL: PHONE_TRANSCRIPT]` | Allows the LLM to apply channel-appropriate interpretation (e.g., more lenient parsing for ASR artifacts in phone transcripts) |
| PII scrubbing | Customer name, account number, SSN, and phone number patterns replaced with tokens (`[CUSTOMER_NAME]`, `[ACCOUNT_NUMBER]`, etc.) using regex before sending to LLM API | Minimizes PII exposure to the third-party LLM provider; required by Pinnacle's data governance policy |
| Text truncation | Contacts exceeding 4,000 tokens truncated to 4,000 tokens, retaining the first 3,500 and last 500 tokens | LLM context window management; complaints are typically identified in the opening or closing of a contact |

### 4.5 Train / Validation / Test Split

| Split | Size | Time Period | Notes |
|---|---|---|---|
| **Training set** | 30,009 (70%) | Jan 2024 – Sep 2025 | Used to develop and refine the prompt and calibrate confidence thresholds |
| **Validation set** | 6,417 (15%) | Oct 2025 – Dec 2025 | Used for threshold tuning and prompt iteration |
| **Test / holdout set** | 6,421 (15%) | Jan 2026 – Feb 2026 | Held out; used only for final performance evaluation reported in Section 6 |

An out-of-time split was used (training precedes validation precedes test chronologically) to reflect the production scenario where the model scores new contacts not seen during development.

### 4.6 Data Assumptions and Limitations

The training data reflects the complaint definition and labeling practices in effect from January 2024 onward. A change in the CFPB complaint definition or Pinnacle's internal complaint policy would require re-evaluation of the training labels and potential model retraining. The model has not been tested on contacts in languages other than English; multilingual contacts are out of scope.

---

## Section 5: Model Development

### 5.1 Problem Framing and Conceptual Approach

The business problem — identifying regulatory complaints in customer contact text — is framed as a binary text classification task. The prediction target is a binary label: `complaint` (any contact meeting the CFPB definition of a consumer complaint) or `non_complaint` (all other contacts). Ground truth labels were produced by Pinnacle's quality analyst team using the CFPB complaint definition as their labeling guide, supplemented by Pinnacle's internal complaint policy.

The key insight driving the LLM-based approach is that the CFPB complaint definition requires semantic and contextual understanding, not just keyword matching. A customer writing "I am very unhappy with this situation and expect this to be fixed immediately" is a complaint; a customer writing "I am unhappy that it's raining today — anyway, can you help me with my account?" is not. A traditional ML model trained on bag-of-words features consistently misclassified the latter category. The LLM approach embeds the CFPB definition directly in the prompt, allowing it to apply the same interpretive judgment a trained human analyst would apply.

A key assumption embedded in the problem framing is that the CFPB complaint definition can be faithfully communicated to an LLM in natural language in a prompt, and that the LLM will apply it consistently. This assumption was validated through the developmental testing described in Section 5.6.

### 5.2 Methodology Selection

**Selected approach:** LLM-based binary classification with structured JSON output, using a hosted third-party LLM API (see Section 9.1). The LLM is provided with the CFPB complaint definition, Pinnacle's internal complaint policy, the contact text, and a structured output schema requiring a binary label, confidence score, and reason codes.

**Rationale for selection:** The semantic complexity of the task makes rule-based and traditional ML approaches insufficient. Keyword approaches achieved only 71% accuracy on the validation set. A fine-tuned BERT-based classifier was explored but required significant labeled data infrastructure and achieved 84.1% F1 — below the LLM-based approach (89.5% F1). The LLM approach also produces human-readable reason codes at no additional cost, supporting the human review layer.

**Alternatives considered:**

| Alternative Approach | Reason Not Selected |
|---|---|
| Keyword / rule-based classifier | 71.2% accuracy on validation set; too many false negatives on paraphrase-heavy complaints and too many false positives on venting-without-resolution contacts |
| Fine-tuned BERT-based classifier (DistilBERT) | 84.1% F1 on validation set; lower performance than LLM approach; requires fine-tuning infrastructure and retraining when complaint definition changes |
| Human-only review (status quo) | 12% sampling coverage creates unacceptable regulatory exposure; 3.5-day average queue entry time creates acknowledgment timeline risk |
| LLM with free-text output (no JSON schema) | Tested in early experiments; output was difficult to parse reliably; structured JSON schema reduced parsing failures from 4.2% to 0.1% |

### 5.3 Model Architecture and Specifications

| Specification | Value |
|---|---|
| **Algorithm / framework** | claude-haiku-4-5-20251001 via Anthropic API; Python inference service using `anthropic` SDK v0.40.0 |
| **Number of features (inputs)** | 1 primary input (contact text) + 1 metadata input (channel tag) |
| **Output type** | Structured JSON: `{"label": "complaint" or "non_complaint", "confidence": 0.0–1.0, "reason_codes": ["...", ...]}` |
| **Output range** | Binary label + continuous confidence score 0.0–1.0 |
| **Decision threshold(s)** | confidence ≥ 0.75 → auto-queue to Resolve360; 0.50 ≤ confidence < 0.75 → priority human review queue; confidence < 0.50 with label = complaint → standard monitoring queue |
| **Model size / complexity** | Hosted model; internal architecture not available. Input limited to 4,000 tokens per contact. |
| **Inference latency** | Median 2.1 seconds; p99 8.4 seconds per contact |

### 5.4 Training Procedure

This model does not involve traditional model training in the gradient-descent sense. Model "development" consisted of iterative prompt engineering on the training set, with performance evaluated on the validation set. The development process involved:

- **Prompt iteration:** 23 prompt versions were evaluated on the training and validation sets. Prompt v7.0 was selected as the production prompt based on highest F1 on the validation set while maintaining recall ≥ 0.88.
- **Temperature setting:** temperature=0 (deterministic output) for all inference calls. This was set explicitly to reduce output variability — a key governance requirement for a regulatory-facing model.
- **Threshold calibration:** Confidence thresholds (0.75 and 0.50) were calibrated on the validation set to balance precision and recall at each routing tier.
- **Output schema enforcement:** A Pydantic model (`ComplaintClassificationOutput`) validates every API response before downstream use.

### 5.5 Hyperparameter Selection and Calibration

| Parameter | Search Range | Final Value | Selection Method |
|---|---|---|---|
| Temperature | 0.0, 0.3, 0.7 | 0.0 | Manual evaluation; temperature=0 produced most consistent results and highest F1 on validation set |
| Complaint confidence threshold (auto-queue) | 0.60, 0.65, 0.70, 0.75, 0.80 | 0.75 | Threshold sweep on validation set; 0.75 achieved best precision (0.924) while keeping recall above 0.85 threshold |
| Non-complaint routing threshold | 0.40, 0.45, 0.50 | 0.50 | Below 0.50 confidence on a complaint label is treated as uncertain; priority review provides safety net |
| Max input tokens | 2,000, 3,000, 4,000 | 4,000 | Performance on long contacts improved meaningfully from 3,000 to 4,000; diminishing returns beyond 4,000 |

### 5.6 Developmental Testing

Developmental testing was performed on the training and validation sets across 23 prompt versions. Key findings:

- **Cross-prompt stability:** Performance on the validation set was stable across the final 5 prompt versions (v6.0 through v7.0), with F1 varying by ±0.012. This indicates the prompt design is not overly sensitive to minor wording changes.
- **Channel-level performance:** Validation set F1 by channel: Email 0.912, Chat 0.903, Phone 0.871. Lower phone performance is expected given ASR transcript quality; this is documented as a known limitation.
- **Edge case testing:** 200 manually constructed edge cases were added to the validation set to test: (1) venting without resolution request (should be non-complaint), (2) complaints phrased as questions, (3) contacts referencing prior unresolved complaints. Model performed correctly on 187/200 (93.5%) edge cases.
- **Temperature=0 reproducibility:** 500 contacts run three times each with temperature=0 produced identical output in 499/500 cases (99.8%); 1 case showed variation on an ambiguous contact, flagged as a known limitation.

---

## Section 6: Model Performance

### 6.1 Evaluation Metrics and Rationale

| Metric | Type | Rationale |
|---|---|---|
| Recall (Sensitivity) | Primary | The most critical metric: minimizing missed complaints (false negatives) is the primary regulatory requirement. Target ≥ 0.88. |
| Precision | Primary | Controls false positive rate — too many false positives overload the complaint specialist team. Target ≥ 0.88. |
| F1 Score | Primary | Harmonic mean of precision and recall; primary overall performance indicator given importance of both error types |
| AUC-ROC | Secondary | Measures discrimination ability across all thresholds; used for model comparison and monitoring |
| False Negative Rate | Secondary | Direct measure of missed complaints; regulatory risk metric |

### 6.2 Holdout Performance Results

| Metric | Training Set | Validation Set | Holdout / Test Set |
|---|---|---|---|
| Recall | 0.912 | 0.896 | 0.883 |
| Precision | 0.931 | 0.921 | 0.924 |
| F1 Score | 0.921 | 0.908 | 0.903 |
| AUC-ROC | 0.967 | 0.951 | 0.946 |
| False Negative Rate | 0.088 | 0.104 | 0.117 |

The holdout F1 of 0.903 represents a 0.005 decline from the validation set (0.908), indicating minimal overfitting to the validation set during threshold calibration. The false negative rate of 11.7% on the holdout set means approximately 1 in 8.5 actual complaints is not identified by the model. This is addressed via the 5% non-complaint sampling control in Section 10.5.

### 6.3 Performance Across Material Subgroups

| Subgroup | N | F1 Score | Notes |
|---|---|---|---|
| Email channel | 2,847 | 0.921 | Highest performance; email text is typically well-structured |
| Chat channel | 2,104 | 0.908 | Strong performance; chat language is informal but clear |
| Phone (ASR) channel | 1,470 | 0.874 | Lowest performance; ASR errors reduce input quality |
| High-confidence complaints (≥0.75) | 1,089 | 0.971 | Near-certain classifications; these auto-queue without human review |
| Low-confidence complaints (0.50–0.74) | 423 | 0.836 | Uncertain classifications; all routed to priority human review |
| Small business contacts | 1,284 | 0.897 | Slightly lower than retail (0.906); small business complaints use more technical language |

### 6.4 Benchmark Comparisons

| Model | F1 Score | Notes |
|---|---|---|
| **Selected model (LLM-based, v7.0 prompt)** | **0.903** | Production model |
| Keyword / rule-based classifier (incumbent) | 0.712 | Status quo; 12% sampling coverage only |
| Fine-tuned DistilBERT | 0.841 | Best traditional ML alternative evaluated |
| Naive majority-class baseline | 0.000 (recall=0) | Labels everything non-complaint |

### 6.5 Sensitivity Analysis

**Top classification drivers (based on 500-contact interpretability sample with structured prompt analysis):**

1. Explicit complaint language ("I am filing a complaint", "I want to escalate this") — strongest positive signal
2. Request for resolution of a problem ("I need this fixed", "please correct this") — strong positive signal
3. Reference to a prior unresolved issue ("I already called about this") — moderate positive signal
4. Negative sentiment without resolution request ("this is frustrating but I just wanted to let you know") — low signal; model correctly classifies as non-complaint
5. ASR artifacts and incomplete sentences — reduces confidence score without changing label in most cases

**Sensitivity to input perturbations:** Tested by masking the explicit complaint language in 100 high-confidence complaints. Of these, 82 remained classified as complaints (from context); 18 flipped to non-complaint or low confidence, confirming that explicit language is a strong but not sole signal.

### 6.6 Explainability and Interpretability

Each model output includes up to three plain-English reason codes explaining the classification (e.g., "Customer explicitly requests resolution of a billing error" or "Customer expresses dissatisfaction with prior service interaction"). Reason codes are generated by the LLM as part of the structured output schema and reviewed by complaint specialists as part of the human review step. Internal testing confirmed that specialists found reason codes accurate and helpful in 91% of reviewed cases.

For regulatory adverse action purposes: this model does not make credit decisions and adverse action reason code requirements do not apply.

---

## Section 7: Assumptions and Limitations

### 7.1 Key Assumptions Registry

| # | Assumption | Sensitivity if Violated | Validation Evidence | Compensating Control |
|---|---|---|---|---|
| 1 | The LLM faithfully applies the CFPB complaint definition as specified in the system prompt | High | 23 prompt versions evaluated; edge case testing confirmed consistent application of the definition across paraphrase variations and ambiguous cases (Section 5.6) | Monthly QA sampling of non-complaint outputs (5% rate); prompt version control; human specialist review of all complaint classifications |
| 2 | ASR transcript quality is sufficient for classification on the phone channel | Medium | Phone channel F1 of 0.874 vs. 0.921 for email demonstrates acceptable performance degradation; contacts with >20% blank segments were excluded from training | Phone channel performance monitored separately; sustained F1 < 0.85 on phone channel triggers ASR quality investigation |
| 3 | The CFPB complaint definition remains stable | High | Currently stable; CFPB has not revised the complaint definition since 2013 | MRM monitors CFPB regulatory updates quarterly; any definition change triggers prompt update and re-validation |
| 4 | PII scrubbing does not materially degrade classification accuracy | Low | 200-contact A/B test comparing scrubbed vs. unmasked contacts showed F1 difference of 0.003 (within noise) | No active control needed; risk accepted as negligible |
| 5 | Temperature=0 produces sufficiently deterministic output for a governance-grade model | Medium | 99.8% reproducibility observed across 500 contacts run three times each (Section 5.6) | Monthly reproducibility check; any contact with inconsistent classifications flagged for manual review |

### 7.2 Known Limitations

| Limitation | Impact | Status |
|---|---|---|
| False negative rate of 11.7% — approximately 1 in 8.5 complaints not identified | Regulatory risk; missed complaints may not enter CFPB workflow | Mitigated — see 5% non-complaint sampling control and specialist escalation path in Section 10.5 |
| Lower performance on phone (ASR) channel (F1 = 0.874) due to ASR transcript quality | Higher false negative rate on phone channel; disproportionate share of missed complaints | Accepted — phone channel has lowest complaint rate (2.4%); absolute miss count is lower than email/chat despite lower F1 |
| LLM output is not 100% deterministic even at temperature=0 | 0.2% of contacts may classify differently across API calls; creates reproducibility question | Accepted — 0.2% rate is below noise threshold; production system uses single inference per contact |
| Model has not been tested on multilingual contacts | Non-English contacts will produce unreliable output | Mitigated — multilingual contacts routed to dedicated team outside model scope; out-of-scope use is explicitly prohibited (Section 2.4) |
| LLM provider may change model behavior in API updates | Undocumented model changes may degrade performance | Mitigated — model version is pinned in production config; version change requires change management record |

### 7.3 Compensating Controls for Known Limitations

**Limitation 1 (False negative rate):** A random 5% sample of all non-complaint-classified contacts is reviewed by a quality analyst each month. This sample is sized to detect a sustained false negative rate above 15% with 90% statistical power. Results are reported monthly to MRM. If the sampled false negative rate exceeds 12%, an expanded 15% sample is triggered for the following month. If the rate exceeds 15% for two consecutive months, a model review is initiated per Section 10.4.

**Limitation 2 (Phone channel performance):** Phone channel F1 is monitored separately in the monthly performance report. If phone channel F1 falls below 0.85 for two consecutive months, a root cause investigation is initiated — focused first on ASR quality before model performance.

**Limitation 5 (LLM version pinning):** The production inference service pins the LLM model version in the API configuration (`model = "claude-haiku-4-5-20251001"`). Any change to the pinned version requires a model change record in Appendix C and a targeted validation of performance on the most recent 500 labeled contacts before deployment.

### 7.4 Conditions Under Which the Model May Fail or Underperform

- **Novel complaint patterns:** A new product launch or regulatory event may generate complaint language patterns not represented in the training data. The model may classify these as non-complaints until the pattern is represented in the prompt or training data.
- **ASR quality degradation:** If the ASR vendor degrades in quality or changes its transcription model, phone channel performance may drop materially without any change to the complaint classifier.
- **CFPB regulatory language change:** Any change to the CFPB complaint definition would immediately invalidate the system prompt and require a full prompt update and re-validation.
- **Adversarial inputs:** A sophisticated customer who deliberately obfuscates complaint language to avoid detection is a theoretical failure mode. This is considered low-probability and low-impact given the regulatory context.

### 7.5 Appropriate Use Boundaries

This model is valid for inbound retail and small business customer contacts in English received through Pinnacle's email, chat, and phone channels from January 2026 onward. It is not valid for: commercial contacts, non-English contacts, contacts processed for backfill purposes outside the current production pipeline, or any use case other than CFPB complaint identification.

---

## Section 8: Third-Party and Vendor Components

### 8.1 Third-Party Components Inventory

| Component | Vendor / Source | Version | Type | Purpose |
|---|---|---|---|---|
| LLM API | Anthropic | claude-haiku-4-5-20251001 (pinned) | Hosted LLM API | Core classification engine |
| ASR Transcription | VoiceCapture Pro (fictional vendor) | v4.2 | Third-party service | Generates phone transcripts; upstream of model |
| `anthropic` Python SDK | Anthropic (open source) | 0.40.0 | Library | Python client for LLM API calls |
| `pydantic` | Pydantic (open source) | 2.7.1 | Library | Output schema validation |

### 8.2 Vendor Validation Approach

**Anthropic LLM API:** Anthropic provides model cards, safety documentation, and API reference documentation for the claude-haiku model family. Pinnacle's validation of the LLM component focused on behavioral validation — evaluating whether the model, when given the production prompt, produces the expected outputs across the labeled dataset — rather than internal architecture review (not available for hosted models). Performance results in Section 6 represent this behavioral validation. Anthropic's model documentation was reviewed and is retained in the model file.

**VoiceCapture Pro ASR:** ASR accuracy (word error rate) was measured on a 500-contact sample using human-transcribed ground truth. Measured WER of 7.9% is within the vendor's published SLA of ≤10% WER. VoiceCapture Pro provides quarterly accuracy reports; these are reviewed by the Contact Center Ops team.

### 8.3 Customizations and Adjustments

No customizations were made to the LLM model itself (it is used via API without fine-tuning). The production prompt (Section 9.2) represents Pinnacle's primary customization — encoding the CFPB definition and institutional complaint policy into the LLM's task context. The prompt is treated as a model component for governance purposes: it is versioned, documented, and subject to change management controls.

### 8.4 Ongoing Vendor Oversight

Anthropic model version changes are monitored via Anthropic's model deprecation and release announcements. The production configuration pins the model version; Pinnacle will not automatically inherit API model updates. When Anthropic announces a deprecation of the pinned version, the ML Engineering team initiates a targeted re-validation on the most recent 500 labeled contacts before migrating to the new version. VoiceCapture Pro ASR accuracy is reviewed quarterly; a drop below WER 12% triggers a vendor escalation.

---

## Section 9: GenAI and LLM Governance

### 9.1 LLM Model and Provider Details

| Field | Value |
|---|---|
| **LLM provider** | Anthropic |
| **Model name and version** | claude-haiku-4-5-20251001 (pinned) |
| **API or self-hosted** | Hosted API |
| **Fine-tuning applied** | No |
| **Context window size** | 200,000 tokens (inputs limited to 4,000 tokens by production configuration) |
| **Output format** | Structured JSON: `{"label": str, "confidence": float, "reason_codes": list[str]}` |

**Regulatory scope note:** SR 26-2 Section II explicitly excludes generative AI and agentic AI from its formal scope, noting they are "novel and rapidly evolving." However, SR 26-2 states that "a banking organization's risk management and governance practices should guide the determination of appropriate governance and controls for any tools, processes, or systems not covered in this document." This MDD applies SR 26-2 principles in the spirit of that guidance. Pinnacle's MRM team has assessed this approach as consistent with sound model risk management practice and consistent with the direction of emerging regulatory guidance on AI governance.

### 9.2 Prompt Engineering Documentation

- **System prompt location:** `src/prompts/complaint_classifier_system_v7.0.txt`
- **User prompt template location:** `src/prompts/complaint_classifier_user_v1.0.txt`
- **Prompt version:** System prompt v7.0 (production); User prompt template v1.0
- **Output schema:** Enforced via Pydantic model `ComplaintClassificationOutput` in `src/models/classification.py`

**System prompt structure:**
The system prompt contains: (1) the CFPB regulatory definition of a consumer complaint (verbatim from CFPB supervisory guidance); (2) Pinnacle's internal complaint policy definition; (3) three few-shot examples of complaint contacts and three of non-complaint contacts (all synthetic — no real customer data); (4) explicit negative instructions (what must NOT be classified as a complaint, including venting without resolution request, compliments with minor frustrations, and questions without expressed dissatisfaction); (5) the required JSON output schema with field descriptions.

**User prompt template structure:**
The user prompt injects two runtime variables: `{{channel_tag}}` (populated with `[CHANNEL: EMAIL]`, `[CHANNEL: CHAT]`, or `[CHANNEL: PHONE_TRANSCRIPT]`) and `{{contact_text}}` (the PII-scrubbed contact text).

**Negative instructions included:**
- Do not classify as a complaint a contact where the customer expresses general frustration but does not indicate an expectation of response or resolution
- Do not classify as a complaint a contact that is entirely a request for information or help without expressed dissatisfaction
- Do not classify as a complaint a contact that is a compliment, even if it includes minor frustrations

### 9.3 Output Validation and Error Handling

All LLM API responses are validated against the `ComplaintClassificationOutput` Pydantic model before downstream use. Validation enforces: `label` is one of `["complaint", "non_complaint"]`; `confidence` is a float in the range [0.0, 1.0]; `reason_codes` is a list of 1–3 non-empty strings.

**Failure handling:**
- If the API returns a response that fails Pydantic validation, the contact is automatically routed to the priority human review queue with flag `VALIDATION_FAILURE`.
- If the API returns an error (rate limit, timeout, server error), the contact is queued for retry up to 3 times with exponential backoff. After 3 failures, the contact is routed to the manual review queue.
- If the API is unavailable for more than 5 minutes, the contact pipeline pauses and the on-call ML Engineer is paged.

In production, validation failures occur at a rate of approximately 0.08% of contacts, consistent with the development-time rate of 0.09%.

### 9.4 Hallucination and Error Risk

**Hallucination risk for this use case:** Low-to-medium. The classification task is closed-label (binary output) with a fixed output schema. The LLM cannot "hallucinate" a third label. However, two hallucination-adjacent failure modes exist:

1. **Label hallucination:** The LLM assigns a label based on superficial linguistic features rather than the actual regulatory definition — e.g., classifying a contact as a complaint because it contains negative sentiment, regardless of whether a resolution is expected. This was the primary failure mode observed in early prompt versions and is mitigated by the explicit negative instructions in the system prompt.

2. **Reason code hallucination:** The LLM generates reason codes that do not accurately reflect the contact content — e.g., citing a billing error when the contact was about a branch closure. Tested on 500 contacts: reason codes were assessed as accurate and relevant by complaint specialists in 91% of cases; inaccurate in 4%; partially accurate in 5%.

**Business impact of undetected hallucination:** A mislabeled complaint entering the CFPB workflow creates unnecessary specialist work (false positive) but is caught by human review. A mislabeled non-complaint missing the CFPB workflow (false negative) creates regulatory risk. The human review layer and 5% non-complaint sampling control are the primary mitigations.

### 9.5 Human-in-the-Loop Controls

Every contact classified as a complaint by the model is reviewed by a human complaint specialist before entering the formal CFPB regulated workflow. This is a 100% review rate for complaint-labeled contacts — not a sample. Specialists use the model's label, confidence score, and reason codes as inputs to their review but make the final determination independently.

Additionally, a 5% random sample of non-complaint-classified contacts is reviewed by quality analysts monthly to estimate and monitor the false negative rate (missed complaints).

Specialist training includes: a 4-hour onboarding module covering the CFPB complaint definition, how to use the model output, when to override a model classification, and how to document overrides. Overrides are logged in Resolve360 and reviewed monthly by the Quality Monitoring Team.

### 9.6 Fine-Tuning Documentation

N/A — No fine-tuning applied. The model uses the base `claude-haiku-4-5-20251001` model via API.

---

## Section 10: Validation and Monitoring Plan

### 10.1 Pre-Production Validation Requirements

| Validation Activity | Responsible Party | Required Before Production? | Status |
|---|---|---|---|
| Conceptual soundness review | James Whitfield, Senior Model Validator | Yes | Complete |
| Data quality independent assessment | James Whitfield, Senior Model Validator | Yes | Complete |
| Performance replication on holdout set | James Whitfield, Senior Model Validator | Yes | Complete |
| Prompt version review and documentation audit | Diana Flores, MRM | Yes | Complete |
| Output validation and error handling review | Diana Flores, MRM | Yes | Complete |
| Ongoing monitoring plan review and approval | Diana Flores, MRM | Yes | Complete |
| Human-in-the-loop control design review | Sarah Kowalski, Model Owner | Yes | Complete |

### 10.2 Ongoing Monitoring Metrics and Thresholds

| Metric | Baseline (Development) | Warning Threshold | Action Threshold | Action Required |
|---|---|---|---|---|
| Recall (overall) | 0.883 | < 0.860 | < 0.840 | MRM notification within 2 business days; model review convened within 10 business days |
| Precision (overall) | 0.924 | < 0.890 | < 0.870 | MRM notification; investigate false positive rate increase |
| F1 Score (overall) | 0.903 | < 0.875 | < 0.855 | MRM notification; model review initiated |
| False Negative Rate (from 5% sampling) | 0.117 | > 0.140 | > 0.160 | Expanded sampling (15%); MRM escalation |
| F1 Score — Phone channel | 0.874 | < 0.850 | < 0.830 | ASR quality investigation before model review |
| PSI (score distribution) | ~0.0 at baseline | > 0.10 | > 0.25 | Input distribution shift investigation |
| Validation failure rate | 0.08% | > 0.50% | > 1.00% | API response quality investigation; Anthropic escalation |
| Override rate (specialist overrides) | ~3% at baseline | > 10% | > 15% | Model quality review; specialist feedback session |

### 10.3 Monitoring Frequency and Reporting

| Activity | Frequency | Owner | Report Delivered To |
|---|---|---|---|
| Performance monitoring report | Monthly | Marcus Chen / ML Team | Diana Flores (MRM), Sarah Kowalski (Model Owner) |
| Non-complaint sample QA review | Monthly | Quality Monitoring Team | Diana Flores (MRM) |
| Override rate review | Monthly | Quality Monitoring Team | Sarah Kowalski (Model Owner) |
| Vendor version / deprecation check | Monthly | ML Engineering | Marcus Chen |
| Full model review | Annual | James Whitfield (Validator) | MRM Committee |

### 10.4 Performance Deterioration Triggers and Escalation

If any Action Threshold in Section 10.2 is breached: the ML Team notifies the Model Owner and MRM within 2 business days. MRM convenes a model review within 10 business days. Possible outcomes: (1) root cause identified and resolved without model change (e.g., ASR quality issue); (2) prompt update initiated — requires prompt version change record in Appendix C and targeted re-validation; (3) model redevelopment initiated; (4) model use restricted or suspended pending review.

### 10.5 Override and Adjustment Policy

Complaint specialists may override the model classification during their human review step. All overrides are logged in Resolve360 with: the original model label, the specialist's determination, and a free-text reason code. Override data is exported monthly for analysis.

An override from `complaint` → `non_complaint` means the specialist determined the contact does not meet the CFPB definition despite the model's classification (false positive correction). An override from `non_complaint` → `complaint` is not possible in the current workflow (non-complaint contacts do not enter the specialist review queue except via the 5% sample).

Override rate above 15% triggers a model quality review — a high override rate signals the model is producing classifications the specialist team does not trust.

### 10.6 Redevelopment Criteria

Model redevelopment is triggered if: (1) F1 falls below 0.855 for three consecutive monthly reporting periods; (2) the CFPB complaint definition changes materially; (3) Pinnacle launches a new contact channel not represented in the training data; (4) the LLM provider discontinues the pinned model version and targeted re-validation on the replacement version shows F1 decline > 3 percentage points; (5) a regulatory examination finding specifically cites complaint identification failures traceable to model performance.

---

## Section 11: Implementation and Deployment

### 11.1 Production Architecture

The complaint classifier runs as a Python FastAPI microservice deployed on Pinnacle's internal Kubernetes cluster. Contact text arrives via three upstream event streams (email, chat, phone) through an internal Kafka topic (`customer-contacts-raw`). The inference service consumes from this topic, applies PII scrubbing and channel tagging, sends the processed text to the Anthropic API, validates the response with Pydantic, and publishes the structured result to a downstream Kafka topic (`complaint-classifications`). Resolve360 consumes from this topic and routes contacts based on the classification label and confidence score.

### 11.2 Input/Output Specifications

| Field | Description | Type | Format / Range | Required? |
|---|---|---|---|---|
| **Input: contact_text** | Full text of the customer contact (post-PII scrubbing) | string | 10–4,000 tokens | Yes |
| **Input: channel** | Originating channel | string | `"email"`, `"chat"`, `"phone"` | Yes |
| **Input: contact_id** | Unique contact identifier for traceability | string | UUID format | Yes |
| **Output: label** | Binary classification | string | `"complaint"` or `"non_complaint"` | — |
| **Output: confidence** | Model confidence score | float | 0.0–1.0 | — |
| **Output: reason_codes** | Plain-English classification rationale | list[str] | 1–3 items, each ≤ 200 characters | — |

If `contact_text` is missing or fewer than 10 tokens after PII scrubbing, the contact is routed directly to the manual review queue without model scoring. A `INSUFFICIENT_INPUT` flag is set in the output record.

### 11.3 Performance and Latency Requirements

| Requirement | Target | Notes |
|---|---|---|
| **Inference latency (p50)** | < 3 seconds | API median observed at 2.1 seconds |
| **Inference latency (p99)** | < 15 seconds | API p99 observed at 8.4 seconds; 15 seconds provides headroom for API fluctuation |
| **Throughput** | 200 contacts/minute sustained | Peak contact volume is approximately 180/minute during Monday morning rush |
| **Availability SLA** | 99.5% | Degraded mode (manual review queue) activates automatically if API unavailable > 5 minutes |

### 11.4 Integration Dependencies

| Dependency | System | Criticality | Fallback if Unavailable |
|---|---|---|---|
| Anthropic LLM API | Anthropic cloud | High | All contacts routed to manual review queue; on-call ML Engineer paged |
| Kafka (contact event stream) | Internal infrastructure | High | Inference service pauses; contacts held in Kafka until service recovers |
| Resolve360 API | Internal | Medium | Classifications cached locally; bulk-loaded to Resolve360 when available |
| VoiceCapture Pro ASR | External vendor | Medium | Phone contacts held in queue; specialist manual transcription for urgent cases |

### 11.5 Rollback Plan

Rollback to manual sampling process (status quo) can be executed within 2 hours by the ML Engineering on-call. Rollback is triggered by the Model Owner or MRM in response to: sustained API unavailability, a model review finding requiring immediate suspension, or a regulatory directive. During rollback, all contacts revert to the prior 12% manual sampling workflow. Rollback decision authority rests with the Model Owner in coordination with MRM.

---

## Section 12: Governance

### 12.1 Roles and Responsibilities

| Role | Name / Team | Responsibilities |
|---|---|---|
| **Model Owner** | Sarah Kowalski, SVP Customer Experience Operations | Accountable for model performance and appropriate use; approves changes; receives monthly performance report |
| **Lead Developer** | Marcus Chen, Senior Data Scientist | Responsible for model development, documentation, prompt versioning, and ongoing technical maintenance |
| **MRM / Validator** | Diana Flores (MRM); James Whitfield (Validator) | Independent validation; monthly performance oversight; approval of material changes |
| **Business User(s)** | Complaint Specialists, Customer Experience Operations | Consume model output; human review layer; responsible for appropriate use |
| **Data Owner** | Customer Experience Operations (email/chat); Contact Center Ops (phone) | Responsible for input data quality and access provisioning |
| **IT / MLOps** | ML Engineering Team | Production deployment, Kafka pipeline, Kubernetes hosting, API key management |
| **Internal Audit** | Internal Audit — Technology & Risk | Evaluates whether MRM practices for this model are rigorous and effective; does not duplicate validation activities |

### 12.2 Model Inventory Registration

| Field | Value |
|---|---|
| **Model inventory system** | Pinnacle ModelTrack (internal inventory system) |
| **Model ID in inventory** | MDL-2026-041 |
| **Inventory registration date** | 2026-03-15 |
| **Tier in model inventory** | Tier 2 — High Materiality |

### 12.3 Change Management

Material changes to this model require MRM review and approval before deployment. A material change is defined as: any change to the system prompt (including addition, removal, or modification of instructions or few-shot examples); any change to the pinned LLM model version; any change to the confidence threshold routing logic; any change to the input feature set or PII scrubbing logic; any change to the output schema.

Minor changes (bug fixes in non-classification logic, infrastructure updates, logging changes) require ML Engineering approval and are logged in Appendix C without requiring MRM review.

All changes are logged in Appendix C with a version increment.

### 12.4 Document Retention

This MDD and all supporting artifacts (prompt files, validation report, labeled training data manifest, experiment logs) shall be retained for a minimum of 7 years following model retirement, consistent with Pinnacle's Model Risk Management Policy and applicable regulatory retention requirements.

---

## Appendix A: Glossary

| Term | Definition |
|---|---|
| **AUC-ROC** | Area Under the Receiver Operating Characteristic Curve; measures discrimination ability across all classification thresholds |
| **CFPB** | Consumer Financial Protection Bureau — federal agency that regulates consumer financial products and services |
| **Consumer complaint** | Per CFPB guidance: any oral or written expression of dissatisfaction where the customer explicitly or implicitly expects a response or resolution |
| **False negative** | A complaint that the model classified as non-complaint; the most serious error type for this model given regulatory implications |
| **False positive** | A non-complaint that the model classified as a complaint; creates operational overhead but is caught by human specialist review |
| **F1 Score** | Harmonic mean of precision and recall; balanced measure of classification performance |
| **LLM** | Large Language Model — a neural network-based model trained on large text corpora, capable of text understanding and generation |
| **PII scrubbing** | Automated removal or tokenization of personally identifiable information before sending contact text to the third-party LLM API |
| **PSI** | Population Stability Index — measures distribution shift between model development and production populations |
| **Pydantic** | Python data validation library used to enforce the output schema on every LLM API response |
| **Recall (Sensitivity)** | The proportion of actual complaints correctly identified by the model; the primary performance metric given regulatory false negative risk |
| **SR 26-2** | Supervisory Guidance on Model Risk Management, issued April 17, 2026 by the Federal Reserve, OCC, and FDIC |
| **Temperature** | LLM inference parameter controlling output randomness; set to 0.0 for deterministic output in this model |
| **WER** | Word Error Rate — proportion of words incorrectly transcribed by the ASR system |

---

## Appendix B: References and Source Documents

| Reference | Description | Location |
|---|---|---|
| SR 26-2 | Supervisory Guidance on Model Risk Management (April 17, 2026) | `_mrm-mdd-kit/governance/sr26-2.md` |
| CFPB Complaint Definition | CFPB supervisory guidance on consumer complaint identification | Retained in model file (Pinnacle compliance SharePoint) |
| Anthropic Model Card — claude-haiku-4-5 | Model documentation provided by Anthropic | Retained in model file |
| Validation Report — MDL-2026-041 | Independent validation report by James Whitfield | Pinnacle ModelTrack document repository |
| Production prompt files | `complaint_classifier_system_v7.0.txt`, `complaint_classifier_user_v1.0.txt` | `src/prompts/` in model repository |
| `ComplaintClassificationOutput` Pydantic schema | Output validation model | `src/models/classification.py` in model repository |

---

## Appendix C: Model Change Log

| Version | Date | Author | Type of Change | MRM Approval Required? | Summary |
|---|---|---|---|---|---|
| 1.0.0 | 2026-05-01 | Marcus Chen | Initial production release | N/A | Initial deployment following full independent validation |

---

## Appendix D: Experiment Log

| Experiment ID | Date | Hypothesis | Approach | Result | Decision |
|---|---|---|---|---|---|
| EXP-001 | 2026-01-15 | Keyword classifier sufficient for complaint identification | Deployed keyword/rule-based classifier on 500-contact validation sample | F1 = 0.712; unacceptable false negative rate on paraphrase-heavy complaints | Rejected; moved to LLM-based approach |
| EXP-002 | 2026-01-28 | Fine-tuned DistilBERT can match LLM performance with lower latency | Fine-tuned DistilBERT on 30,009 training contacts | F1 = 0.841; below LLM performance; fine-tuning infrastructure overhead high | Rejected; LLM approach selected |
| EXP-003 | 2026-02-10 | Free-text LLM output parseable reliably | LLM with free-text (non-JSON) output | 4.2% API response parse failure rate; impractical for production | Rejected; structured JSON schema adopted |
| EXP-004 | 2026-02-18 | temperature=0 sufficient for deterministic governance-grade output | 500 contacts run 3x each at temperature=0 | 99.8% identical output across runs | Accepted; temperature=0 adopted as production setting |
| EXP-005 | 2026-02-25 | Prompt v7.0 achieves best F1 with recall ≥ 0.88 | 23 prompt versions evaluated on training/validation sets | Prompt v7.0: validation F1 = 0.908, recall = 0.896 | Selected as production prompt |
