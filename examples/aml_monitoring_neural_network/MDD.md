# Model Development Document (MDD)

> **Kit version:** mrm-mdd-kit v1.0.0 | **Regulatory basis:** SR 26-2 (April 17, 2026)
>
> **FICTIONAL EXAMPLE** — All institution names, personnel, performance metrics, and business details are completely made up. This document exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## Document Control

| Field | Value |
|---|---|
| **Model Name** | AML Suspicious Activity Scoring Model |
| **Model ID** | MDL-2025-008 |
| **Model Type** | Neural Network (Feedforward / Tabular) — Anomaly Scorer |
| **Model Version** | 1.2.0 |
| **Document Version** | 1.2 |
| **Document Status** | Approved |
| **Development Start Date** | 2024-09-01 |
| **Document Date** | 2025-04-10 |
| **Effective Date** | 2025-06-01 |
| **Next Review Date** | 2026-06-01 |
| **Model Owner** | Frank Nakamura, Chief BSA/AML Officer |
| **Lead Developer** | Aisha Oduya, Senior Data Scientist, Compliance Analytics |
| **Contributing Developers** | James Kwon, Data Engineer, Compliance Analytics |
| **Primary Validator** | Michael Torres, Senior Model Validator |
| **MRM Contact** | Sandra Lee, VP Model Risk Management |

### Approval Sign-offs

| Role | Name | Signature | Date |
|---|---|---|---|
| Lead Developer | Aisha Oduya | *(signed)* | 2025-04-10 |
| Model Owner | Frank Nakamura | *(signed)* | 2025-04-14 |
| MRM / Validation Lead | Michael Torres | *(signed)* | 2025-05-02 |
| Chief Risk Officer (or delegate) | Priya Chatterjee, CRO | *(signed)* | 2025-05-08 |

### Document Version History

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| 1.0 | 2025-02-15 | Aisha Oduya | Initial draft |
| 1.1 | 2025-03-20 | Aisha Oduya | MRM pre-review feedback incorporated; expanded Section 9 (N/A) and Section 7 |
| 1.2 | 2025-04-10 | Aisha Oduya | Updated to reflect SR 26-2 (supersedes SR 11-7 and SR 21-8); final pre-approval |

---

## Section 1: Executive Summary

### 1.1 Model Overview

The AML Suspicious Activity Scoring Model is a feedforward neural network that scores transactions for suspicious activity risk in support of Harborview Financial's Bank Secrecy Act (BSA) / Anti-Money Laundering (AML) compliance obligations. The model performs two functions: (1) it re-ranks the existing rule-based TMS alert queue, prioritizing high-risk alerts for earlier analyst review; and (2) it generates net-new risk scores for transactions that did not trigger a rule-based alert, surfacing behavioral anomalies for analyst review. All SAR filing decisions are made exclusively by licensed AML analysts — the model is a triage and prioritization tool, not a decision engine. Analysts use model scores as one input among several, retaining full discretion over SAR filings.

### 1.2 Model Risk Rating

| Dimension | Rating | Rationale |
|---|---|---|
| **Inherent Risk** | High | Neural network architecture has limited inherent interpretability; 112 behavioral features; complex temporal and network-based features; adversarial adaptation risk from money laundering typology evolution |
| **Model Exposure** | Medium | Model prioritizes analyst queue but does not make SAR filing decisions; human review is mandatory for all SAR determinations; however, low-priority queue items may receive less thorough review |
| **Model Purpose** | High | Directly supports BSA/AML regulatory compliance; FinCEN examiner scrutiny of SAR filing practices; failure to identify suspicious activity patterns creates significant regulatory and legal risk |
| **Overall Materiality** | High | High purpose risk and high inherent risk; human-in-the-loop partially mitigates exposure |

### 1.3 Key Risks and Findings

- **Neural network interpretability:** The model provides anomaly scores and top contributing features but does not produce fully transparent decision logic. AML analysts use scores as inputs to their own analysis, not as explanations. SAR narrative must be written by the analyst independently of the model score. Full treatment in Sections 6.6 and 7.2.
- **Adversarial adaptation:** Sophisticated money laundering operations may adapt their behavior to evade model detection over time — a risk inherent to any quantitative AML model. The model is retrained semi-annually to incorporate recent confirmed suspicious activity patterns. Full treatment in Section 7.4.
- **Net-new alert volume management:** The model's net-new alert capability (flagging transactions not caught by rules) must be calibrated carefully to avoid overwhelming the analyst team. A net-new alert precision target of ≥ 8% SAR conversion rate is set to ensure net-new alerts are meaningfully more suspicious than random transactions. Full treatment in Section 10.2.
- **SR 21-8 supersession:** SR 21-8 (AML-specific model risk guidance, April 2021) was explicitly superseded by SR 26-2 (April 2026). This MDD is written to the SR 26-2 standard. The principles of SR 21-8 (particularly the emphasis on human review of all SAR decisions) remain consistent with SR 26-2 and are incorporated throughout.

### 1.4 Regulatory Applicability

| Regulation | Applicable? | Notes |
|---|---|---|
| SR 26-2 — Model Risk Management | Yes | Full SR 26-2 applicability; supersedes SR 11-7 and SR 21-8. SR 26-2's risk-based, materiality-scaled approach governs this model. |
| BSA / Bank Secrecy Act | Yes | Model directly supports SAR filing obligations under 31 U.S.C. § 5318(g) |
| FinCEN SAR Filing Requirements | Yes | All SAR decisions made by human analysts; model output is supporting information, not the basis for filing |
| ECOA / Reg B | No | Not a credit decision model |
| CECL / FASB ASC 326 | No | Not a credit loss model |
| CFPB UDAAP | Conditional | AML monitoring should not produce discriminatory monitoring patterns that disproportionately surveil protected class customers without BSA justification |

---

## Section 2: Model Purpose and Business Context

### 2.1 Business Problem and Objectives

Harborview Financial's rule-based TMS generates approximately 4,800 alerts per month from approximately 2.1 million daily transactions. The AML analyst team of 18 investigators reviews all alerts within a 30-day review window. The investigation-to-SAR conversion rate under the prior triage process was approximately 4.1% — meaning 96% of analyst effort was expended on non-productive alerts. This creates two material problems: (1) analyst capacity is overwhelmed by low-risk alerts, increasing risk that genuinely suspicious patterns receive insufficient attention; and (2) novel money laundering typologies that do not match existing rule patterns go undetected by the rule-based system.

Objectives: (1) increase the investigation-to-SAR conversion rate for rule-based alerts to ≥ 12% by prioritizing the highest-risk alerts for first review; (2) surface net-new suspicious transaction patterns not captured by existing rules, with a SAR conversion rate ≥ 8% on net-new alerts; (3) reduce analyst time spent on lowest-risk alerts without reducing total SAR filing accuracy.

### 2.2 Intended Use

The model produces a suspicious activity risk score (0.0–1.0) for each transaction. For rule-based TMS alerts, scores re-rank the queue: high-score alerts (≥ 0.70) are tagged Priority 1 for same-day review; medium-score alerts (0.40–0.69) are tagged Priority 2 for review within 5 business days; low-score alerts (< 0.40) are tagged Priority 3 for review within 20 business days. For non-alert transactions, scores above 0.65 generate net-new analyst alerts for review. All alerts — whether rule-generated or model-generated — are reviewed by a licensed AML analyst before any SAR determination. The model score is displayed in the analyst workstation as one data element alongside transaction history, customer profile, and rule triggers.

### 2.3 Intended Users

| User / System | How Output Is Used |
|---|---|
| AML Analyst Workstation (ComplianceDesk) | Displays model score, priority tier, and top contributing features alongside alert details |
| AML Analysts (18 investigators) | Use model score as prioritization input; conduct independent investigation; make SAR determination |
| Frank Nakamura, Chief BSA/AML Officer | Reviews monthly performance report; SAR filing trend analysis |
| Sandra Lee, MRM | Reviews quarterly model performance report |

### 2.4 Out-of-Scope and Prohibited Uses

- The model score must never be used as the sole or primary basis for a SAR filing. SAR decisions must be grounded in the analyst's independent review of the transaction and customer file.
- The model must not be used to systematically de-prioritize or close alerts without analyst review. Every alert — regardless of model score — must be reviewed by a licensed analyst within the 30-day regulatory window.
- The model must not be used to make account closure, transaction blocking, or customer termination decisions.
- The model must not be applied to commercial or correspondent banking transactions; it was developed on consumer and small business transaction data.
- Net-new alert volume must not exceed 800 per month (analyst capacity constraint); the net-new alert threshold (currently 0.65) must be re-calibrated if net-new volume approaches this limit.

### 2.5 Downstream Decisions and Impact

The model affects the priority order in which analysts review the approximately 4,800 monthly rule-based alerts and generates up to 600 net-new alerts per month. No transaction is blocked, account closed, or SAR filed based on model output alone. However, alerts systematically deprioritized by the model (Priority 3) may receive less thorough investigation than Priority 1 alerts due to analyst time allocation. This creates a residual risk that genuinely suspicious activity in the low-score band may not be identified within the review window.

### 2.6 Model Dependencies

| Dependency | Type | Description |
|---|---|---|
| Rule-based TMS (existing system) | Upstream | Generates the base alert queue that the model re-ranks |
| Customer Behavioral Feature Store | Upstream | Pre-computed customer-level behavioral aggregates (30/90/180-day windows) |
| Transaction Ledger | Upstream | Real-time and historical transaction data |
| ComplianceDesk (analyst workstation) | Downstream | Displays model scores and priority tiers to analysts |

---

## Section 3: Model Risk Assessment

### 3.1 Inherent Risk Assessment

| Risk Factor | Assessment | Rationale |
|---|---|---|
| **Model complexity** | High | Feedforward neural network with 3 hidden layers (256/128/64 units); 112 input features including temporal sequences and network graph features; non-linear architecture; limited inherent interpretability |
| **Number and criticality of assumptions** | High | Assumes training data (confirmed SAR/non-SAR labels) accurately reflects true suspicious activity; assumes money laundering patterns stable over semi-annual retraining cycle; assumes model network features capture behavioral relationships |
| **Input data quality** | Medium | Internal transaction data is high quality; behavioral aggregate features are internally computed; confirmed SAR labels have known coverage gaps (suspicious activity not identified by prior system is unlabeled) |
| **Data constraints / availability** | Medium | 36 months of confirmed SAR labels (2022–2024); SAR label base rate approximately 0.18% of alerts; semi-annual retraining cycle accounts for pattern evolution |
| **Model interpretability** | High | Neural network provides top-feature attribution but not transparent logic; analysts cannot fully audit why a specific score was generated |
| **Novelty of approach** | Medium | ML-based AML scoring is increasingly industry-standard; FinCEN has provided guidance on ML use in AML; well-established use case with growing literature |
| **Overall Inherent Risk** | High | Black-box complexity, critical BSA assumptions, and adversarial adaptation risk drive High inherent risk |

### 3.2 Model Exposure Assessment

| Exposure Factor | Value | Notes |
|---|---|---|
| **Portfolio / business scope** | ~2.1M transactions/day; ~4,800 rule alerts/month; up to 600 net-new alerts/month | All consumer and small business transactions |
| **Frequency of model use** | Daily batch scoring for prioritization; real-time for net-new alert generation | — |
| **Business impact of incorrect output** | High (missed SAR) / Medium (misprioritized queue) | Missed SARs create FinCEN examination risk; misprioritized queue creates analyst efficiency risk |
| **Degree of automation** | Human-in-the-loop | No SAR filing without analyst decision; all alerts reviewed within regulatory window |
| **Overall Exposure** | Medium | Human-in-the-loop mitigates direct impact; but indirection (priority tiering affects analyst time allocation) creates residual risk |

### 3.3 Model Purpose Assessment

| Purpose Factor | Assessment | Notes |
|---|---|---|
| **Regulatory-facing use** | Yes | BSA/AML compliance; FinCEN examiner scrutiny |
| **Financial risk management use** | No | AML is a compliance function, not a financial risk management function in the credit/market risk sense |
| **Consumer-impacting decisions** | Conditional | Model does not directly take adverse action; however, differential monitoring intensity raises UDAAP considerations |
| **Overall Purpose Risk** | High | BSA regulatory compliance with FinCEN examiner scrutiny |

### 3.4 Overall Materiality Determination

| | |
|---|---|
| **Overall Materiality** | High |
| **Validation rigor required** | Full independent validation including conceptual soundness, data quality assessment, performance evaluation, and SAR label quality review. Mandatory human-in-the-loop verification. |
| **Monitoring frequency** | Quarterly |
| **Rationale** | High purpose risk (BSA/AML regulatory compliance, FinCEN) combined with High inherent risk (neural network complexity, adversarial adaptation). Human-in-the-loop partially mitigates exposure to Medium. Overall: High. |

---

## Section 4: Data

### 4.1 Data Sources and Provenance

| Data Source | System of Record | Data Owner | Data Type | Refresh Frequency |
|---|---|---|---|---|
| Transaction Ledger | Core Banking (FIS Modern Banking) | Transaction Operations | Transaction-level records: amount, type, channel, counterparty | Real-time |
| SAR Filing History | ComplianceDesk | BSA/AML Compliance | Historical SAR filings with subject transaction IDs | Monthly extract |
| TMS Alert History | Rule-based TMS | BSA/AML Compliance | Alert records with investigation outcomes (SAR / no SAR) | Monthly extract |
| Customer Profile Data | CRM | Customer Operations | Customer type, tenure, account structure, country of origin | Daily batch |
| Behavioral Feature Store | Internal (Spark/Delta Lake) | Compliance Analytics | Pre-computed 30/90/180-day behavioral aggregates | Daily batch |

### 4.2 Observation Period and Sample Design

| | |
|---|---|
| **Observation period** | January 1, 2022 – December 31, 2024 (36 months) |
| **Sample size** | 847,392 TMS alerts; 34,783 confirmed SAR subjects; 812,609 investigated non-SAR alerts |
| **Geographic scope** | All Harborview operating markets |
| **Product scope** | Consumer and small business deposit, transfer, and payment transactions |
| **Exclusion criteria** | Alerts with incomplete investigation records excluded (2.1%); correspondent banking alerts excluded (separate AML program); transactions from accounts closed prior to alert review excluded (0.4%) |
| **Rationale for observation period** | 36 months captures three distinct money laundering typology cycles observed in Harborview's customer base (2022 structured cash deposits, 2023 crypto off-ramp patterns, 2024 remittance layering). FinCEN SAR data from this period provides a sufficient positive class for neural network training. |

### 4.3 Data Quality Assessment

| Quality Dimension | Finding | Action Taken |
|---|---|---|
| **Completeness** | Transaction features: 99.2% complete. Behavioral aggregates: 97.4% complete (2.6% null due to new accounts with insufficient history). Customer profile: 98.7% complete. | Null behavioral features imputed with account-age-matched population medians; new account indicator flag added |
| **Accuracy** | SAR labels reviewed: confirmed SAR filings are authoritative; however, the label set has a known coverage gap — suspicious activity that was not detected by the prior TMS system (false negatives in the training data) is labeled "non-suspicious" in the training data. This creates label noise in the negative class. | Documented as a known limitation (Section 7.2); impact assessed as acceptable at this scale |
| **Timeliness** | SAR labels have up to 30-day lag (review window); training data uses labels as of 60 days post-transaction to ensure most labels are finalized | Accepted; 60-day label stabilization cutoff applied |
| **Consistency** | TMS rule set changed in March 2023 (17 new rules added); alert volume increased 22% post-change | Pre/post rule change flagged in training data; model evaluated separately on pre- and post-change periods; no significant performance difference |
| **Representativeness** | SAR base rate of 4.1% in TMS alert population; overall transaction SAR rate ~0.0027%. Positive class (SAR) is a small minority even in the alert population. | `class_weight` parameter used to address imbalance; net-new alert threshold separately calibrated |

### 4.4 Feature Engineering and Preprocessing

| Feature / Transformation | Description | Rationale |
|---|---|---|
| Transaction velocity features | Count and sum of transactions by account over 1d/7d/30d/90d windows, by transaction type | Structuring and layering patterns create velocity anomalies |
| Round-number transaction flag | Binary flag for transactions within 1% of a round number ($1,000, $5,000, $10,000) | Round-number transactions are a classic structuring signal |
| Counterparty network features | Number of unique counterparties in 30/90 days; proportion of transactions to new counterparties | Rapid expansion of counterparty network is a layering indicator |
| Geographic anomaly features | Distance between transaction location and customer's primary state; cross-border transaction flag | Unusual geographic patterns associated with money movement |
| Cash transaction features | Cash deposit/withdrawal count and amount in 7d/30d windows; CTR proximity flag (>$9,000) | Structuring detection; CTR proximity is a high-priority signal |
| Time-of-day anomaly | Transactions outside 6am–10pm local time as a proportion of recent activity | Unusual hour patterns associated with certain fraud and AML typologies |
| Account age at transaction | Days since account opening at time of transaction | New accounts used for money movement are higher risk |
| Customer risk tier | Existing customer risk tier from CRM (High/Medium/Low) | Prior BSA risk assessment provides context |

### 4.5 Train / Validation / Test Split

| Split | Size | Time Period | Notes |
|---|---|---|---|
| **Training set** | 593,174 (70%) | Jan 2022 – Jun 2024 | Time-based split; all behavioral features computed using data prior to transaction date |
| **Validation set** | 127,109 (15%) | Jul 2024 – Sep 2024 | Out-of-time; used for threshold calibration and architecture selection |
| **Test / holdout set** | 127,109 (15%) | Oct 2024 – Dec 2024 | Out-of-time; used only for final evaluation reported in Section 6 |

Out-of-time split is essential for this use case given adversarial adaptation — a random split would not test the model's ability to detect money laundering patterns in a period following training, which is the production scenario.

---

## Section 5: Model Development

### 5.1 Problem Framing and Conceptual Approach

AML suspicious activity scoring is framed as a binary classification problem: given a transaction (and associated account behavioral context), does this transaction belong to a suspicious activity pattern warranting analyst review and potential SAR filing (1) or not (0)? Ground truth labels come from confirmed SAR filings and investigated non-SAR alerts from the rule-based TMS.

The fundamental challenge in AML model development is label quality: the training "negative" class (non-SAR investigated alerts) may contain genuinely suspicious transactions that were not identified by the prior system. This creates label noise in the negative class. The neural network's ability to learn complex behavioral patterns from many features partially compensates for this noise at scale.

The conceptual approach is that money laundering activity — structuring, layering, integration — creates distinctive behavioral signatures in transaction data: unusual velocity, round-number patterns, counterparty network expansion, geographic anomalies. The neural network learns to recognize these signatures from the combination of 112 features, capturing non-linear interactions (e.g., moderate cash velocity combined with rapid counterparty expansion and geographic anomaly produces a compound signal stronger than any feature alone).

### 5.2 Methodology Selection

**Selected approach:** Feedforward neural network with 3 hidden layers (256/128/64 units), ReLU activations, batch normalization, dropout regularization. Trained with Adam optimizer, binary cross-entropy loss, `class_weight` to address imbalance.

**Rationale:** Neural network captures complex non-linear interactions among 112 features that ensemble tree methods partially capture but with less flexibility for behavioral sequence features. The primary objective is maximizing SAR conversion rate at the top of the alert queue — neural network ranked first on this metric across all models evaluated. Gradient boosting (XGBoost) was the closest competitor and is maintained as a benchmark.

**Alternatives considered:**

| Alternative Approach | Reason Not Selected |
|---|---|
| Rule-based TMS only (status quo) | 4.1% SAR conversion rate; no net-new alert capability; goal is to supplement, not replace |
| XGBoost | Priority 1 SAR conversion rate: 19.4% vs. neural network 22.1%; selected neural network for higher-risk queue precision |
| Isolation Forest (unsupervised) | No label utilization; SAR conversion rate on surfaced anomalies 6.2% — below 8% net-new target |
| Logistic Regression | SAR conversion rate Priority 1: 14.8%; insufficient for the prioritization objective |

### 5.3 Model Architecture and Specifications

| Specification | Value |
|---|---|
| **Algorithm / framework** | PyTorch 2.3.0; 3-layer feedforward network; Adam optimizer |
| **Number of features (inputs)** | 112 behavioral, transactional, and customer context features |
| **Output type** | Continuous suspicious activity probability (0.0–1.0) |
| **Output range** | 0.0–1.0 |
| **Decision threshold(s)** | Alert prioritization: ≥ 0.70 → Priority 1; 0.40–0.69 → Priority 2; < 0.40 → Priority 3. Net-new alerts: ≥ 0.65 on non-alerted transactions. |
| **Model size / complexity** | 256/128/64 hidden units; dropout 0.3; batch normalization; 68,000 parameters; serialized artifact: 1.2MB |
| **Inference latency** | Median 4.1ms; p99 9.8ms per transaction |

### 5.4 Training Procedure

- **Loss function:** Binary cross-entropy with `class_weight` (positive class weight = 23.3, reflecting ~4.1% positive rate in alert population)
- **Optimizer:** Adam, learning rate 0.001, weight decay 1e-4
- **Regularization:** Dropout 0.3 after each hidden layer; batch normalization for training stability
- **Training regime:** 50 epochs, batch size 512; early stopping on validation AUC (patience=5)
- **Hardware:** Single NVIDIA A100 GPU; training time approximately 22 minutes
- **Reproducibility:** `torch.manual_seed(42)`; PyTorch version pinned

### 5.5 Hyperparameter Selection and Calibration

| Hyperparameter | Search Range | Final Value | Selection Method |
|---|---|---|---|
| Hidden layer sizes | [128/64/32], [256/128/64], [512/256/128] | 256/128/64 | Grid search on validation AUC; diminishing returns above 256/128/64 |
| Dropout rate | 0.2, 0.3, 0.4 | 0.3 | Grid search; 0.3 optimal generalization |
| Learning rate | 0.0001, 0.001, 0.01 | 0.001 | Grid search; 0.001 standard for Adam |
| Priority 1 threshold | 0.60–0.80 | 0.70 | Calibrated to achieve ~15% of alert queue in Priority 1 while maximizing SAR conversion rate |
| Net-new alert threshold | 0.55–0.75 | 0.65 | Calibrated to produce ≤ 600 net-new alerts/month at ≥ 8% SAR conversion target |

### 5.6 Developmental Testing

- **5-fold time-series cross-validation:** Mean AUC 0.874 ± 0.011 across folds; moderate variance expected given adversarial pattern evolution across periods
- **Out-of-time validation (Jul–Sep 2024):** AUC 0.861; SAR conversion rate Priority 1 queue: 21.4%; net-new alert SAR conversion: 8.9%
- **Typology subgroup testing:** Model evaluated on confirmed SAR typologies: structuring (AUC 0.912), layering/wire transfers (AUC 0.847), remittance abuse (AUC 0.871), crypto off-ramp (AUC 0.834)
- **Net-new alert capacity test:** At threshold 0.65, net-new alert volume was 547/month on validation period — within the 600 analyst capacity limit

---

## Section 6: Model Performance

### 6.1 Evaluation Metrics and Rationale

| Metric | Type | Rationale |
|---|---|---|
| AUC-ROC | Primary | Discrimination measure across all thresholds |
| SAR Conversion Rate — Priority 1 Queue | Primary | Business performance metric: what % of Priority 1 alerts result in SAR filings? Target ≥ 12% |
| SAR Conversion Rate — Net-New Alerts | Primary | Measures net-new alert quality; target ≥ 8% |
| False Negative Rate (missed SARs) | Primary | Most critical regulatory metric: what % of actual SAR subjects receive Priority 3 score? |
| Precision @ Top 10% of Queue | Secondary | How well the model concentrates risk at the top of the queue |

### 6.2 Holdout Performance Results

| Metric | Training Set | Validation Set | Holdout / Test Set |
|---|---|---|---|
| AUC-ROC | 0.901 | 0.861 | 0.857 |
| SAR Conversion Rate — Priority 1 | 24.8% | 21.4% | 22.1% |
| SAR Conversion Rate — Net-New | 11.2% | 8.9% | 9.4% |
| False Negative Rate (SAR subjects → Priority 3) | 4.1% | 5.8% | 5.3% |
| Precision @ Top 10% | 0.341 | 0.298 | 0.312 |

The holdout AUC of 0.857 represents a 0.004 decline from validation (0.861), indicating minimal overfitting. The Priority 1 SAR conversion rate of 22.1% (vs. baseline 4.1%) demonstrates strong risk concentration. The false negative rate of 5.3% means approximately 1 in 19 SAR subjects receives a Priority 3 score; these subjects must still be reviewed within the 30-day window by regulation, but may receive less thorough investigation.

### 6.3 Performance Across Material Subgroups

| Subgroup | N (holdout) | AUC-ROC | SAR Conversion | Notes |
|---|---|---|---|---|
| Structuring alerts | 18,421 | 0.912 | 28.4% (P1) | Strongest performance; structured cash patterns are distinctive |
| Wire transfer / layering alerts | 31,847 | 0.847 | 19.7% (P1) | Good performance; complex cross-institutional patterns harder to capture |
| Remittance alerts | 24,319 | 0.871 | 21.3% (P1) | Strong; international remittance to high-risk corridors well-represented in training |
| Crypto off-ramp alerts | 11,204 | 0.834 | 16.8% (P1) | Adequate; crypto typologies are newer and less represented in training data |
| New accounts (< 90 days) | 8,847 | 0.848 | 23.6% (P1) | Slightly elevated SAR rate; new account AML risk well-captured |
| Net-new alerts (non-rule triggered) | 4,891 | N/A | 9.4% | Above 8% target; confirms net-new quality |

### 6.4 Benchmark Comparisons

| Model | AUC-ROC | Priority 1 SAR Conversion | Notes |
|---|---|---|---|
| **Selected model (Neural Network v1.2.0)** | **0.857** | **22.1%** | Production model |
| Rule-based TMS only (status quo) | N/A | 4.1% (undifferentiated) | No score-based prioritization |
| XGBoost benchmark | 0.841 | 19.4% | Maintained as ongoing benchmark |
| Logistic Regression | 0.798 | 14.8% | Baseline |

### 6.5 Sensitivity Analysis

**Top features by SHAP importance (holdout set):**
1. `cash_deposit_count_7d` — 0.1124 (structuring signal)
2. `ctr_proximity_flag` — 0.0987 (transactions approaching $10,000 CTR threshold)
3. `counterparty_new_ratio_30d` — 0.0843 (rapid counterparty expansion)
4. `cash_amount_7d_sum` — 0.0791
5. `round_number_transaction_flag` — 0.0712
6. `cross_border_flag` — 0.0634
7. `customer_risk_tier_high` — 0.0598
8. `geographic_anomaly_score` — 0.0521
9. `transaction_velocity_1d` — 0.0487
10. `account_age_at_transaction_days` — 0.0412

### 6.6 Explainability and Interpretability

SHAP values are computed for every scored alert and displayed in the ComplianceDesk analyst workstation as the top 5 contributing features to the model score. Analysts use these features as starting points for their investigation — not as conclusions. **Importantly, the SAR narrative must be written by the analyst based on their independent review of the full customer and transaction file, not derived from model features.**

The model does not produce adverse action reason codes (AML monitoring is not an adverse action context). Interpretability is provided purely to support analyst efficiency, not for regulatory reason-code compliance purposes.

---

## Section 7: Assumptions and Limitations

### 7.1 Key Assumptions Registry

| # | Assumption | Sensitivity if Violated | Validation Evidence | Compensating Control |
|---|---|---|---|---|
| 1 | Confirmed SAR filing history provides a representative sample of money laundering activity in Harborview's customer base | High | SAR history reviewed by Chief BSA/AML Officer for representativeness; typology coverage assessed against FinCEN SAR trend reports | Semi-annual retraining incorporates new SAR patterns; FinCEN SAR trend report review by Compliance Analytics quarterly |
| 2 | Money laundering behavioral patterns are sufficiently stable over the semi-annual retraining cycle | High | AUC stability across the 36-month training window (Section 5.6); moderate variance (±0.011) indicates some pattern evolution | Semi-annual retraining; monthly SAR conversion rate monitoring to detect sudden performance drops |
| 3 | The 30-day regulatory review window is sufficient for Priority 3 alerts to receive adequate investigation | High | All alerts reviewed within 30 days regardless of priority tier (regulatory requirement); Priority 3 review capacity analyzed and confirmed by Chief BSA/AML Officer | Monthly analyst queue capacity review; Priority 3 alert volume monitored; threshold recalibration if Priority 3 volume exceeds analyst capacity |
| 4 | The negative training class (investigated non-SAR alerts) does not contain a disproportionate number of genuinely suspicious transactions that were missed by the prior system | Medium | Cannot be fully validated without omniscient ground truth; estimated label noise of 3–7% in negative class based on analyst supervisor review of 500 low-score alerts | Accepted as inherent limitation of supervised AML model training; documented in Section 7.2 |
| 5 | Net-new alert threshold of 0.65 maintains analyst capacity within the 600/month ceiling | Medium | Validated on holdout period: 547 net-new alerts/month | Monthly net-new volume monitoring; threshold raised if volume approaches 600 |

### 7.2 Known Limitations

| Limitation | Impact | Status |
|---|---|---|
| Label noise in negative training class (non-SAR alerts may contain undetected suspicious activity) | Model may have learned to score some genuinely suspicious patterns low if they consistently evaded prior detection | Accepted — inherent limitation of supervised AML modeling; all Priority 3 alerts still reviewed within 30-day window |
| 5.3% false negative rate (SAR subjects scored Priority 3) | Approximately 1 in 19 SAR subjects may receive less thorough initial review | Mitigated — all alerts reviewed within regulatory window; Priority 3 analysts receive specific training on not under-investigating low-score alerts |
| Lower performance on crypto off-ramp typologies (AUC 0.834) | Emerging crypto AML patterns are underrepresented in training data | Mitigated — semi-annual retraining incorporates new confirmed SAR patterns; FinCEN crypto guidance monitored |
| Neural network provides feature attribution but not transparent logic | Analysts and examiners cannot fully audit the model's scoring rationale | Mitigated — human analyst review is mandatory; SAR narrative is analyst-authored; model score is supporting, not determinative |

### 7.3 Compensating Controls for Known Limitations

**False negative rate (5.3%):** Priority 3 analysts receive specific training on the limitation — the training emphasizes that a low model score does not indicate absence of suspicious activity, only that the model did not identify a behavioral pattern consistent with training data. The 30-day review window ensures all Priority 3 alerts receive analyst review regardless of model score. Monthly sampling of Priority 3 closed (no-SAR) alerts by senior analysts provides a quality check.

**Crypto typology underperformance:** The Compliance Analytics team monitors FinCEN SAR trend reports and Financial Crimes Enforcement Network advisories for emerging crypto-related typologies. New confirmed crypto SAR cases are incorporated into the next semi-annual retraining cycle.

### 7.4 Conditions Under Which the Model May Fail or Underperform

- **Novel money laundering typology:** A sophisticated new ML scheme designed to avoid the behavioral signatures in the training data (gradual velocity increases, diverse counterparty structure, below-threshold transaction sizing) could evade model detection. This is the fundamental adversarial adaptation risk.
- **Major change in customer base:** Significant growth in a new customer segment (e.g., international business accounts, cannabis banking) with different legitimate transaction patterns could create false positive spikes or calibration issues.
- **TMS rule set change:** Major additions or deletions to the rule-based alert triggers would change the composition of the input alert queue, potentially affecting model calibration.
- **Data pipeline failure:** Stale behavioral aggregate features (> 48 hours old) would degrade model performance on velocity and pattern features without triggering an obvious scoring failure.

### 7.5 Appropriate Use Boundaries

This model is valid for AML alert prioritization and net-new suspicious activity detection for consumer and small business transactions at Harborview Financial. It is not valid for: correspondent banking, commercial banking, or any SAR filing context outside of Harborview's BSA/AML program. The model may not be used to close alerts, block transactions, or take adverse action against customers without analyst review.

---

## Section 8: Third-Party and Vendor Components

### 8.1 Third-Party Components Inventory

| Component | Vendor / Source | Version | Type | Purpose |
|---|---|---|---|---|
| PyTorch | Open source (Meta) | 2.3.0 | Library | Neural network framework |
| SHAP | Open source | 0.46.0 | Library | Feature attribution for analyst workstation display |
| ComplianceDesk | Fictional vendor | v7.1 | Platform | AML analyst workstation; receives and displays model scores |

### 8.2 Vendor Validation Approach

**ComplianceDesk:** The integration between the scoring model and ComplianceDesk was tested via a 90-day parallel run (model scores computed but not displayed to analysts) to verify score display accuracy and alert routing logic. Results confirmed 100% accuracy in score-to-priority-tier routing logic.

### 8.3–8.4 Customizations and Ongoing Oversight

No vendor model components used. All modeling is internally developed. ComplianceDesk integration is maintained by the Technology team; any change to the score display or routing logic in ComplianceDesk requires ML Engineering review for potential model impact.

---

## Section 9: GenAI and LLM Governance

N/A — This is a traditional neural network model trained on tabular transaction data with no generative AI or LLM components.

---

## Section 10: Validation and Monitoring Plan

### 10.1 Pre-Production Validation Requirements

| Validation Activity | Responsible Party | Required Before Production? | Status |
|---|---|---|---|
| Conceptual soundness review | Michael Torres | Yes | Complete |
| Data quality independent assessment | Michael Torres | Yes | Complete |
| SAR label quality review | Michael Torres + Chief BSA/AML Officer | Yes | Complete |
| Performance replication on holdout set | Michael Torres | Yes | Complete |
| Analyst workflow integration review | Frank Nakamura (BSA/AML Officer) | Yes | Complete |
| Net-new alert capacity validation | Frank Nakamura | Yes | Complete |
| Ongoing monitoring plan review | Sandra Lee (MRM) | Yes | Complete |

### 10.2 Ongoing Monitoring Metrics and Thresholds

| Metric | Baseline (Development) | Warning Threshold | Action Threshold | Action Required |
|---|---|---|---|---|
| AUC-ROC (quarterly backtesting) | 0.857 | < 0.830 | < 0.810 | MRM notification; model review within 15 business days |
| Priority 1 SAR Conversion Rate | 22.1% | < 16.0% | < 12.0% | Alert quality review; potential threshold recalibration |
| Net-New Alert SAR Conversion Rate | 9.4% | < 7.0% | < 5.5% | Net-new threshold recalibration; MRM review |
| False Negative Rate (SAR subjects → P3) | 5.3% | > 8.0% | > 11.0% | MRM escalation; BSA/AML Officer notification; expanded P3 sampling |
| Net-New Alert Volume (monthly) | 547 | > 550 | > 600 | Threshold recalibration to maintain analyst capacity |
| PSI (score distribution) | ~0.0 | > 0.10 | > 0.25 | Input distribution investigation |
| Typology AUC — Crypto off-ramp | 0.834 | < 0.800 | < 0.770 | Crypto typology-specific retraining evaluation |

### 10.3 Monitoring Frequency and Reporting

| Activity | Frequency | Owner | Report Delivered To |
|---|---|---|---|
| Performance monitoring report | Quarterly | Aisha Oduya | Sandra Lee (MRM), Frank Nakamura (BSA/AML Officer) |
| SAR conversion rate tracking | Monthly | BSA/AML Compliance | Frank Nakamura, Sandra Lee |
| Net-new alert volume and quality | Monthly | Compliance Analytics | Frank Nakamura |
| Semi-annual model retraining | Semi-annual | Aisha Oduya | Sandra Lee (MRM) for retraining approval |
| Full model validation | Annual | Michael Torres | MRM Committee |

### 10.4 Performance Deterioration Triggers and Escalation

If Priority 1 SAR conversion rate falls below 12% Action Threshold: BSA/AML Officer and MRM notified within 2 business days. This is treated as a potential BSA compliance risk, not just a model performance issue. Root cause analysis conducted within 10 business days. Possible outcomes: (1) threshold recalibration; (2) out-of-cycle retraining; (3) suspension of net-new alert generation; (4) escalation to external BSA consultant review.

### 10.5 Override and Adjustment Policy

The BSA/AML Officer may adjust priority tier thresholds (0.70 and 0.40) as an operational control without triggering full model redevelopment, subject to MRM notification and documentation of the rationale and expected impact. Threshold adjustments are logged in Appendix C.

AML analysts may tag a Priority 3 alert as "analyst-escalated" if they believe the model score understates the risk. Analyst-escalated alerts are treated as Priority 1. The analyst-escalation rate is monitored monthly; a sustained escalation rate > 10% of Priority 3 alerts signals potential model underperformance and triggers a review.

### 10.6 Redevelopment Criteria

Model redevelopment triggered by: (1) Priority 1 SAR conversion rate below 12% for two consecutive quarters; (2) false negative rate (SAR subjects → Priority 3) above 11% for two consecutive quarters; (3) a FinCEN examination finding related to SAR identification that cites model performance; (4) emergence of a new major money laundering typology not adequately addressed by semi-annual retraining; (5) major change in Harborview's business mix creating a materially different transaction population.

---

## Section 11: Implementation and Deployment

### 11.1 Production Architecture

The model runs as a Python microservice (FastAPI) deployed on Harborview's on-premises Kubernetes cluster. Transaction scoring runs in two modes: (1) daily batch scoring of the prior day's TMS alert queue for prioritization (run at 6am daily); and (2) near-real-time scoring of all non-alerted transactions for net-new alert generation (lag ≤ 30 minutes from transaction processing). Scores are written to the ComplianceDesk database via REST API; priority tiers are assigned automatically in ComplianceDesk based on score thresholds.

### 11.2 Input/Output Specifications

| Field | Description | Type | Format / Range | Required? |
|---|---|---|---|---|
| **Input: alert_id / transaction_id** | Unique identifier | string | UUID | Yes |
| **Input: transaction_amount** | Amount in USD | float | > 0 | Yes |
| **Input: transaction_type** | Type code | string | Internal taxonomy | Yes |
| **Input: behavioral_features [array]** | 112 precomputed features | float array | Various ranges | Yes |
| **Output: suspicion_score** | Suspicious activity probability | float | 0.0–1.0 | — |
| **Output: priority_tier** | Queue priority | string | `"P1"`, `"P2"`, `"P3"` | — |
| **Output: top_shap_features** | Top 5 contributing features | list[str] | Feature names + direction | — |

### 11.3 Performance and Latency Requirements

| Requirement | Target | Notes |
|---|---|---|
| **Batch scoring latency** | < 2 hours for full alert queue | Daily batch run; SLA = alerts prioritized before analysts arrive at 8am |
| **Net-new alert latency** | < 30 minutes from transaction | Near-real-time; not a hard real-time constraint |
| **Availability SLA** | 99.0% | Degraded mode: rule-based FIFO queue used if model unavailable |

### 11.4 Integration Dependencies

| Dependency | System | Criticality | Fallback if Unavailable |
|---|---|---|---|
| Behavioral Feature Store | Internal (Spark/Delta Lake) | High | Scoring delayed until features available; max 6-hour wait before FIFO fallback |
| ComplianceDesk API | Vendor platform | High | Scores cached locally; bulk-loaded to ComplianceDesk when available |
| TMS Alert Export | Internal | High | Model waits for alert export; FIFO queue activated if > 2 hour delay |

### 11.5 Rollback Plan

Rollback to FIFO (chronological) alert queue can be executed within 15 minutes by disabling the model scoring service in the TMS configuration. All alerts remain in the queue for analyst review in filing order. Net-new alert generation is suspended during rollback. Rollback decision authority rests with the Chief BSA/AML Officer in coordination with MRM.

---

## Section 12: Governance

### 12.1 Roles and Responsibilities

| Role | Name / Team | Responsibilities |
|---|---|---|
| **Model Owner** | Frank Nakamura, Chief BSA/AML Officer | BSA program accountability; SAR quality oversight; alert threshold governance |
| **Lead Developer** | Aisha Oduya, Senior Data Scientist | Model development, semi-annual retraining, performance monitoring |
| **MRM / Validator** | Sandra Lee (MRM); Michael Torres (Validator) | Independent validation; quarterly performance oversight |
| **Business User(s)** | AML Analysts (18 investigators) | Alert investigation; SAR filing decisions; analyst-escalation authority |
| **Data Owner** | Transaction Operations (ledger); Compliance Analytics (feature store) | Data quality and pipeline integrity |
| **IT / MLOps** | Technology — Compliance Systems (James Kwon lead) | Deployment, ComplianceDesk integration, data pipeline |
| **Internal Audit** | Internal Audit — BSA/AML | Annual BSA/AML model risk effectiveness review |

### 12.2 Model Inventory Registration

| Field | Value |
|---|---|
| **Model inventory system** | Harborview ModelRegistry |
| **Model ID in inventory** | MDL-2025-008 |
| **Inventory registration date** | 2025-02-20 |
| **Tier in model inventory** | Tier 1 — High Materiality |

### 12.3 Change Management

Semi-annual retraining (new data, no architectural change) requires MRM notification and SAR conversion rate sign-off. Architecture changes (new features, new network structure) require full MRM review and targeted validation. Threshold changes (priority tier cutoffs) require BSA/AML Officer and MRM approval. All changes logged in Appendix C.

### 12.4 Document Retention

This MDD and all supporting artifacts shall be retained for a minimum of 7 years following model retirement per Harborview's BSA/AML Record Retention Policy and applicable Bank Secrecy Act retention requirements.

---

## Appendix A: Glossary

| Term | Definition |
|---|---|
| **AML** | Anti-Money Laundering — regulatory compliance program to detect and report money laundering activity |
| **AUC-ROC** | Area Under the Receiver Operating Characteristic Curve |
| **BSA** | Bank Secrecy Act — federal law requiring financial institutions to assist government agencies in detecting and preventing money laundering |
| **CTR** | Currency Transaction Report — required for cash transactions ≥ $10,000 |
| **False Negative Rate (AML context)** | Proportion of confirmed SAR subjects that received a Priority 3 (low) model score |
| **FinCEN** | Financial Crimes Enforcement Network — U.S. Treasury bureau that administers the BSA |
| **SAR** | Suspicious Activity Report — required filing under BSA when suspicious activity is detected |
| **SAR Conversion Rate** | Proportion of investigated alerts that result in a SAR filing |
| **SHAP** | SHapley Additive exPlanations — method for computing feature contributions to individual model predictions |
| **SR 26-2** | Supervisory Guidance on Model Risk Management (April 17, 2026) — supersedes SR 11-7 and SR 21-8 |
| **Structuring** | Money laundering technique: breaking large cash transactions into smaller amounts to avoid CTR filing thresholds |
| **TMS** | Transaction Monitoring System — rule-based system that generates AML alerts |

---

## Appendix B: References

| Reference | Description | Location |
|---|---|---|
| SR 26-2 | Supervisory Guidance on Model Risk Management (April 17, 2026) — supersedes SR 21-8 | `_mrm-mdd-kit/governance/sr26-2.md` |
| FinCEN SAR Trend Reports (2022–2024) | FinCEN published SAR statistics used for typology coverage assessment | FinCEN.gov; retained in model file |
| Validation Report — MDL-2025-008 v1.2 | Independent validation by Michael Torres | Harborview ModelRegistry |

---

## Appendix C: Model Change Log

| Version | Date | Author | Type of Change | MRM Approval Required? | Summary |
|---|---|---|---|---|---|
| 1.0.0 | 2025-06-01 | Aisha Oduya | Initial production release | N/A | Deployed as alert prioritization supplement to rule-based TMS |
| 1.1.0 | 2025-09-15 | Aisha Oduya | Semi-annual retraining | MRM notification | Retrained on Jan 2022 – Jun 2025 data; incorporated 2025 remittance layering patterns |
| 1.2.0 | 2025-12-01 | Aisha Oduya | Updated to SR 26-2; minor threshold adjustment | MRM review | Updated regulatory basis from SR 11-7/SR 21-8 to SR 26-2; net-new threshold adjusted 0.67 → 0.65 to increase net-new volume to target |

---

## Appendix D: Experiment Log

| Experiment ID | Date | Hypothesis | Approach | Result | Decision |
|---|---|---|---|---|---|
| EXP-2024-001 | 2024-10-01 | Isolation Forest (unsupervised) can surface novel AML patterns | Isolation Forest trained on full transaction population | Net-new SAR conversion rate 6.2% — below 8% target | Rejected; insufficient precision for analyst capacity |
| EXP-2024-002 | 2024-10-20 | XGBoost matches neural network with better interpretability | XGBoost with same feature set | Priority 1 SAR conversion 19.4% vs. neural network 22.1% | Neural network selected; maintained as ongoing benchmark |
| EXP-2024-003 | 2024-11-05 | Deeper neural network (4 layers) improves AUC | 512/256/128/64 architecture | AUC 0.858 vs. 3-layer 0.857; no meaningful improvement | 3-layer architecture retained; simpler model preferred |
| EXP-2024-004 | 2024-11-18 | Net-new alert threshold 0.65 achieves ≤ 600/month target | Threshold sweep 0.55–0.75 on validation period | 0.65 → 547 net-new alerts/month; 0.60 → 812/month (over capacity) | 0.65 adopted |
