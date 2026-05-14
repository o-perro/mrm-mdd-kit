# Model Development Document (MDD)

> **Kit version:** mrm-mdd-kit v1.0.0 | **Regulatory basis:** SR 26-2 (April 17, 2026)
>
> **FICTIONAL EXAMPLE** — All institution names, personnel, performance metrics, and business details are completely made up. This document exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## Document Control

| Field | Value |
|---|---|
| **Model Name** | CNP Transaction Fraud Scoring Model |
| **Model ID** | MDL-2025-017 |
| **Model Type** | Gradient Boosting (XGBoost) Binary Classifier |
| **Model Version** | 2.1.0 |
| **Document Version** | 2.1 |
| **Document Status** | Approved |
| **Development Start Date** | 2025-03-01 |
| **Document Date** | 2025-08-15 |
| **Effective Date** | 2025-10-01 |
| **Next Review Date** | 2026-10-01 |
| **Model Owner** | Patricia Vance, SVP Fraud Risk Management |
| **Lead Developer** | Kevin Osei, Lead Data Scientist, Fraud Analytics |
| **Contributing Developers** | Anita Sharma, Data Engineer; Tyler Brooks, ML Engineer |
| **Primary Validator** | Stephanie Morales, Senior Model Validator |
| **MRM Contact** | Lawrence Kim, VP Model Risk Management |

### Approval Sign-offs

| Role | Name | Signature | Date |
|---|---|---|---|
| Lead Developer | Kevin Osei | *(signed)* | 2025-08-15 |
| Model Owner | Patricia Vance | *(signed)* | 2025-08-19 |
| MRM / Validation Lead | Stephanie Morales | *(signed)* | 2025-09-08 |
| Chief Risk Officer (or delegate) | Margaret Chen, CRO | *(signed)* | 2025-09-12 |

### Document Version History

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| 1.0 | 2023-06-01 | Kevin Osei | Initial model (v1.0.0) — replaced FraudGuard Pro rules engine |
| 2.0 | 2025-06-10 | Kevin Osei | Major redevelopment: expanded feature set, retrained on 2023–2025 data, added SHAP explainability |
| 2.1 | 2025-08-15 | Kevin Osei | Updated performance results; expanded Section 7 per MRM pre-review feedback |

---

## Section 1: Executive Summary

### 1.1 Model Overview

The CNP Transaction Fraud Scoring Model is an XGBoost gradient boosting classifier that scores each card-not-present transaction across Westbrook National Bank's consumer credit and debit card portfolios for probability of fraud in real time. The model generates a fraud probability score (0.0–1.0) within 85 milliseconds of each authorization request. Transactions scoring above 0.82 are automatically declined; transactions scoring 0.55–0.82 trigger step-up SMS authentication; transactions scoring below 0.55 proceed normally. The model processes approximately 4.2 million transactions per day and directly replaced a legacy vendor rules engine (FraudGuard Pro) as Westbrook's primary CNP fraud detection mechanism.

### 1.2 Model Risk Rating

| Dimension | Rating | Rationale |
|---|---|---|
| **Inherent Risk** | High | Complex ensemble model with 187 features including behavioral, network, and velocity features; significant feature engineering; non-linear interactions difficult to fully audit; adversarial drift risk from fraud typology evolution |
| **Model Exposure** | High | ~4.2M transactions/day across 9.4M consumer accounts; automated decline decisions with no human review for high-score transactions; estimated $47M annual fraud exposure addressed by the model |
| **Model Purpose** | High | Primary fraud control for CNP channel; directly affects customer account access; automatic declines constitute adverse actions subject to UDAAP scrutiny |
| **Overall Materiality** | Critical | All three dimensions High; automated high-stakes consumer decisions at scale |

### 1.3 Key Risks and Findings

- **False positive impact on customers:** At the production threshold of 0.82, the model declines approximately 0.31% of legitimate transactions. At 4.2M daily transactions, this affects approximately 13,000 legitimate transactions per day. Customer friction and potential UDAAP exposure are the primary false positive risks. Full treatment in Sections 7.2 and 10.2.
- **Adversarial drift:** Fraud typologies evolve rapidly as fraudsters adapt to detection. The model was trained on data through February 2025; emerging fraud patterns post-training may not be detected. Full treatment in Section 7.4.
- **Feature staleness risk:** 14 of the 187 features depend on external data providers (device intelligence, IP geolocation). Degradation in provider data quality will affect model performance without a change in the model itself. Full treatment in Section 8.
- **Fairness / disparate impact monitoring:** Automatic declines affect customers differentially across demographic segments. While the model does not use protected class variables, proxy variables may create disparate impact. Fairness monitoring is included in the ongoing monitoring plan (Section 10.2).
- **Latency risk:** The model must score transactions within 85ms p99 to meet the authorization network SLA. Under peak load, latency occasionally approaches the threshold. Infrastructure capacity planning is documented in Section 11.3.

### 1.4 Regulatory Applicability

| Regulation | Applicable? | Notes |
|---|---|---|
| SR 26-2 — Model Risk Management | Yes | Traditional ML model at a $210B institution; full SR 26-2 applicability |
| ECOA / Reg B — Fair Lending | Conditional | Model does not make credit decisions, but automatic declines may constitute adverse actions under UDAAP; disparate impact monitoring required |
| CECL / FASB ASC 326 | No | Not a credit loss estimation model |
| BSA / AML — Bank Secrecy Act | No | Not a transaction monitoring model for AML purposes |
| CFPB UDAAP | Yes | Automatic transaction declines affect consumer account access; disparate impact potential |

---

## Section 2: Model Purpose and Business Context

### 2.1 Business Problem and Objectives

Westbrook National Bank processes approximately 4.2 million CNP transactions per day. CNP fraud — primarily e-commerce transactions using stolen card credentials — represented approximately $47 million in annual gross fraud losses before model implementation, net of chargebacks recovered. The prior system, FraudGuard Pro (a vendor rules engine), had two material deficiencies: (1) a 22% false positive rate among flagged transactions, creating customer friction and approximately $4.2 million annually in dispute handling costs; and (2) slow adaptation to emerging fraud typologies — new fraud patterns typically required 6–8 weeks to reflect in updated rules, during which loss rates spiked.

Model v2.0 objectives: (1) reduce fraud losses by at least 30% relative to the FraudGuard Pro baseline at equivalent false positive rate; (2) reduce the false positive rate on declined transactions from 22% to below 12%; (3) enable real-time adaptation to fraud pattern shifts through model retraining on a quarterly cadence rather than a 6–8 week rules deployment cycle.

### 2.2 Intended Use

The model produces a continuous fraud probability score (0.0–1.0) for each CNP transaction at authorization time. Three score bands drive three outcomes: (1) score ≥ 0.82 → automatic decline, fraud operations queue entry; (2) score 0.55–0.81 → step-up SMS OTP authentication challenge sent to cardholder; (3) score < 0.55 → transaction proceeds normally. The entire scoring and routing decision occurs within the 85ms authorization window with no human involvement in the real-time decision. Fraud operations reviews automatic declines in batch for potential customer outreach.

### 2.3 Intended Users

| User / System | How Output Is Used |
|---|---|
| Authorization Engine (internal) | Routes transaction based on score band in real time |
| Step-Up Authentication Service | Triggered for score band 0.55–0.81; sends SMS OTP |
| Fraud Operations Team | Reviews declined transactions; handles customer disputes and cardholder outreach |
| Lawrence Kim, MRM | Reviews monthly performance report |

### 2.4 Out-of-Scope and Prohibited Uses

- This model must not be used for card-present (in-person) transaction fraud scoring; it was developed exclusively on CNP transaction data.
- This model must not be used to assess creditworthiness or make credit limit decisions.
- This model must not be applied to commercial card transactions; scope is consumer credit and debit only.
- Automatic declines must not be treated as permanent account suspensions; a declined transaction does not constitute account closure or adverse credit action.
- Model scores must not be shared externally, used for marketing segmentation, or applied to non-fraud use cases.

### 2.5 Downstream Decisions and Impact

The model scores 4.2 million transactions per day across 9.4 million consumer accounts. At the production decline threshold (0.82), approximately 0.31% of transactions receive automatic declines — approximately 13,000 transactions per day. Of these, an estimated 11.7% are legitimate transactions (false positives) — approximately 1,500 legitimate transaction declines per day. Each false positive creates potential customer friction and a dispute handling cost of approximately $8. Step-up authentication is triggered for approximately 3.4% of transactions daily — approximately 142,800 transactions.

### 2.6 Model Dependencies

| Dependency | Type | Description |
|---|---|---|
| Device Intelligence API (DeviceIQ Pro) | Upstream | Provides device fingerprint features; 14 features sourced from this vendor |
| IP Geolocation Service (GeoRisk) | Upstream | Provides IP-based geolocation and VPN/proxy detection features |
| Customer Behavior Feature Store | Upstream | Internal feature store providing 30-day and 90-day behavioral aggregates |
| Fraud Operations Case Management | Downstream | Receives declined transactions for review; feeds back confirmed fraud labels for model retraining |

---

## Section 3: Model Risk Assessment

### 3.1 Inherent Risk Assessment

| Risk Factor | Assessment | Rationale |
|---|---|---|
| **Model complexity** | High | XGBoost ensemble with 500 trees, max depth 7, 187 features including interaction terms and velocity features; non-linear decision boundaries; significant complexity |
| **Number and criticality of assumptions** | High | Assumes fraud patterns in training data (Jan 2022 – Feb 2025) are representative of current fraud; assumes feature quality from third-party providers; assumes stable relationship between behavioral features and fraud probability |
| **Input data quality** | Medium | Internal transaction data is high quality; third-party device intelligence and geolocation data has occasional gaps (< 2% null rate) |
| **Data constraints / availability** | Low | 3 years of labeled transaction data with 847,000 confirmed fraud cases provides robust training corpus |
| **Model interpretability** | High | XGBoost is partially interpretable via SHAP values, but real-time individual decision explanation is limited; 187-feature complexity makes full auditability challenging |
| **Novelty of approach** | Low | XGBoost is a well-established industry standard for fraud detection; extensive published literature and industry benchmarks exist |
| **Overall Inherent Risk** | High | Complexity, critical assumptions, and limited individual-decision interpretability drive High inherent risk |

### 3.2 Model Exposure Assessment

| Exposure Factor | Value | Notes |
|---|---|---|
| **Portfolio / business scope** | 4.2M transactions/day; 9.4M consumer accounts | Entire CNP transaction portfolio |
| **Frequency of model use** | Real-time; 4.2M times per day | Every CNP authorization request |
| **Business impact of incorrect output** | High | False negatives = fraud losses (~$47M annual exposure); false positives = customer friction + dispute costs |
| **Degree of automation** | Fully automated | No human review before automatic decline; human review occurs post-hoc only |
| **Overall Exposure** | High | Maximum-volume real-time fully-automated consumer decisions |

### 3.3 Model Purpose Assessment

| Purpose Factor | Assessment | Notes |
|---|---|---|
| **Regulatory-facing use** | Yes | UDAAP applies to automatic declines; disparate impact monitoring required |
| **Financial risk management use** | Yes | Primary control for CNP fraud loss management; directly affects the P&L |
| **Consumer-impacting decisions** | Yes | Automatic declines directly affect consumer account access in real time |
| **Overall Purpose Risk** | High | All three factors apply |

### 3.4 Overall Materiality Determination

| | |
|---|---|
| **Overall Materiality** | Critical |
| **Validation rigor required** | Full independent validation including conceptual soundness, data quality, performance replication, fairness assessment, and outcomes analysis. Ongoing monthly monitoring required. |
| **Monitoring frequency** | Monthly performance; real-time latency monitoring |
| **Rationale** | All three materiality dimensions are High. Fully automated real-time adverse consumer decisions at 4.2M transactions/day with $47M annual financial exposure and UDAAP regulatory applicability. This is Westbrook's highest-materiality production model. |

---

## Section 4: Data

### 4.1 Data Sources and Provenance

| Data Source | System of Record | Data Owner | Data Type | Refresh Frequency |
|---|---|---|---|---|
| CNP Transaction Ledger | Core Banking System (Fiserv DNA) | Card Operations | Transaction-level records: amount, merchant, timestamp, channel | Real-time |
| Confirmed Fraud Labels | Fraud Case Management System | Fraud Operations | Binary fraud / legitimate label from chargeback adjudication | Daily batch |
| Customer Behavioral Aggregates | Internal Feature Store (Spark/Delta Lake) | Fraud Analytics | 7/30/90-day behavioral aggregates per account | Hourly refresh |
| Device Intelligence | DeviceIQ Pro API | Vendor (DeviceIQ Inc.) | Device fingerprint, device age, device risk score, 14 features | Real-time per transaction |
| IP Geolocation / Risk | GeoRisk API | Vendor (GeoRisk Ltd.) | IP location, ISP, VPN/proxy flag, country risk score | Real-time per transaction |

### 4.2 Observation Period and Sample Design

| | |
|---|---|
| **Observation period** | January 1, 2022 – February 28, 2025 (38 months) |
| **Sample size** | 312,847,000 transactions (847,213 confirmed fraud; 311,999,787 legitimate) |
| **Geographic scope** | All U.S. states where Westbrook operates |
| **Product scope** | Consumer CNP credit and debit transactions only |
| **Exclusion criteria** | Transactions with missing merchant category code excluded (0.3%); transactions from accounts in bankruptcy or fraud suspension excluded (0.1%); test transactions excluded |
| **Rationale for observation period** | 38-month window captures three distinct fraud typology cycles (post-COVID e-commerce surge, 2023 credential stuffing wave, 2024 synthetic identity-adjacent CNP patterns). Sufficient history for behavioral feature computation (90-day lookback features require 90+ days of prior data). |

### 4.3 Data Quality Assessment

| Quality Dimension | Finding | Action Taken |
|---|---|---|
| **Completeness** | Transaction core fields: 99.8% complete. Device intelligence features: 98.1% complete (1.9% null — primarily transactions from older devices with no DeviceIQ record). IP geolocation: 97.4% complete. | Null device/IP features imputed with account-level median values for continuous features; binary flags set to "unknown" category for categorical features |
| **Accuracy** | Fraud label accuracy: chargeback-adjudicated labels estimated at 94–96% accuracy (industry standard for CNP chargeback labels); approximately 4–6% of fraud cases are recovered as chargebacks but remain labeled fraudulent | Label noise accepted as within industry norms; model robustly handles moderate label noise at this training scale |
| **Timeliness** | Chargeback labels have an average 45-day lag between fraud occurrence and label assignment | Training used labels available as of April 2025 for the Feb 2025 end of observation period; labels with lag > 60 days excluded from recent months |
| **Consistency** | Minor taxonomy changes in merchant category codes occurred in March 2023; recoded to consistent 2023+ taxonomy for full training period | Recoding applied; verified on random 10,000-transaction sample |
| **Representativeness** | Severe class imbalance: 0.271% fraud rate. Training distribution reflects true population; no upsampling applied to preserve calibration | Class imbalance addressed via `scale_pos_weight` XGBoost parameter (set to 368, reflecting class ratio) |

### 4.4 Feature Engineering and Preprocessing

| Feature / Transformation | Description | Rationale |
|---|---|---|
| Transaction velocity features | Count of transactions by account in past 1h, 6h, 24h, 7d, 30d | Velocity spikes are a strong fraud signal; multiple windows capture both burst and sustained unusual activity |
| Amount deviation features | Transaction amount vs. account 30-day mean and std | Unusually large or small transactions relative to account behavior |
| Merchant risk features | Merchant category fraud rate (30-day rolling, internal), merchant country risk score | High-risk merchant categories and geographies are strong fraud predictors |
| Time-of-day / day-of-week cyclical encoding | Hour and day encoded as sin/cos pairs | Preserves cyclical nature of time features without artificial discontinuity |
| Account tenure feature | Days since account opening | New accounts have higher fraud rates; captures account age risk |
| Device novelty flag | First use of this device on this account (binary) | New device = elevated fraud probability |
| Geographic distance | Great-circle distance between IP geolocation and account's most common transaction location | Unusual geography is a classic fraud signal |
| IP risk indicators | VPN/proxy flag, Tor exit node flag, data center IP flag | High-risk IP types strongly associated with CNP fraud |
| Missing value indicators | Binary flags for null device intelligence and IP geolocation fields | Preserves information value of missingness pattern |

### 4.5 Train / Validation / Test Split

| Split | Size | Time Period | Notes |
|---|---|---|---|
| **Training set** | 218,993,000 (70%) | Jan 2022 – Apr 2024 | Time-based split; all behavioral features computed on data prior to the transaction date |
| **Validation set** | 46,927,000 (15%) | May 2024 – Oct 2024 | Out-of-time; used for hyperparameter tuning and threshold calibration |
| **Test / holdout set** | 46,927,000 (15%) | Nov 2024 – Feb 2025 | Out-of-time; used only for final evaluation reported in Section 6 |

Out-of-time split is critical for this use case — random splitting would create data leakage through behavioral aggregate features that span the split boundary. The 6-month holdout period contains two distinct fraud pattern shifts (November 2024 holiday season spike, January 2025 credential stuffing campaign) providing a rigorous real-world evaluation.

### 4.6 Data Assumptions and Limitations

The training data reflects the fraud patterns, transaction mix, and behavioral norms of the 2022–2025 period. A significant macroeconomic event, major product change (e.g., new payment rails), or novel fraud typology can shift the underlying distribution without any change to the model. The 45-day chargeback label lag means the most recent 45 days of training data have incomplete fraud labels; this is a known limitation managed by the observation period cutoff.

---

## Section 5: Model Development

### 5.1 Problem Framing and Conceptual Approach

CNP fraud detection is framed as a binary classification problem: each transaction is either fraudulent (1) or legitimate (0). The prediction target is a chargeback-adjudicated fraud label assigned an average of 45 days post-transaction. The model produces a fraud probability score, which is then mapped to three action bands via calibrated thresholds.

The conceptual foundation is that fraudulent transactions differ systematically from legitimate ones across behavioral, device, geographic, and temporal dimensions — patterns that emerge at the individual transaction level and in aggregate behavioral context. XGBoost captures non-linear interactions among these dimensions more effectively than linear models, making it appropriate for a task where fraud signals are complex and interdependent (e.g., an unusual amount is not suspicious by itself, but combined with a new device at an unusual hour from an unfamiliar IP creates a strong compound signal).

### 5.2 Methodology Selection

**Selected approach:** XGBoost 2.1.0 gradient boosting classifier with 500 trees, max depth 7, trained with `scale_pos_weight` to address severe class imbalance. SHAP values computed for each production prediction to enable post-hoc explainability.

**Rationale for selection:** XGBoost is the industry standard for CNP fraud detection due to its strong performance on tabular data with class imbalance, fast inference (< 5ms for scoring, well within the 85ms authorization window), and partial interpretability via SHAP. It outperformed all alternatives evaluated on the validation set. Deep learning alternatives (LightGBM neural network hybrid, TabNet) offered marginal performance gains with significantly higher inference latency and deployment complexity.

**Alternatives considered:**

| Alternative Approach | Reason Not Selected |
|---|---|
| FraudGuard Pro rules engine (incumbent) | Validation AUC 0.891; 22% false positive rate among flagged transactions; 6–8 week adaptation lag to new fraud patterns |
| LightGBM | Validation AUC 0.967 vs. XGBoost 0.971; marginal difference; XGBoost preferred for team familiarity and validation precedent |
| Neural network (TabNet) | Validation AUC 0.968; p99 latency 47ms vs. XGBoost 8ms; insufficient latency margin for the 85ms authorization window under peak load |
| Logistic Regression | Validation AUC 0.941; insufficient for the non-linear interaction complexity of CNP fraud patterns |

### 5.3 Model Architecture and Specifications

| Specification | Value |
|---|---|
| **Algorithm / framework** | XGBoost 2.1.0; Python scoring service with scikit-learn 1.5.0 pipeline wrapper |
| **Number of features (inputs)** | 187 features across transaction, behavioral, device, and network categories |
| **Output type** | Continuous fraud probability score (0.0–1.0) |
| **Output range** | 0.0–1.0 (calibrated probability) |
| **Decision threshold(s)** | ≥ 0.82 → decline; 0.55–0.81 → step-up auth; < 0.55 → approve. Thresholds calibrated on validation set to optimize fraud loss reduction at target false positive rate. |
| **Model size / complexity** | 500 trees, max depth 7; serialized model artifact: 47MB |
| **Inference latency** | Median 3.2ms; p99 7.8ms (scoring only; network overhead excluded) |

### 5.4 Training Procedure

- **Loss function:** Binary cross-entropy (log loss) with `scale_pos_weight=368` to address 0.271% fraud rate class imbalance
- **Optimization:** Gradient boosting with learning rate 0.05, 500 estimators, early stopping on validation log loss (patience=20)
- **Regularization:** `reg_alpha=0.1` (L1), `reg_lambda=1.0` (L2), `subsample=0.8`, `colsample_bytree=0.7` — tuned via Bayesian optimization (Section 5.5)
- **Probability calibration:** Isotonic regression calibration applied post-training on the validation set to ensure score is a well-calibrated probability rather than a raw ensemble score
- **Hardware:** Training on 8x NVIDIA A100 GPUs; training time approximately 4.5 hours on full dataset
- **Reproducibility:** Random seed fixed at 42; XGBoost version pinned in `pyproject.toml`

### 5.5 Hyperparameter Selection and Calibration

| Hyperparameter | Search Range | Final Value | Selection Method |
|---|---|---|---|
| `n_estimators` | 200–800 | 500 | Bayesian optimization (Optuna, 150 trials) on validation AUC |
| `max_depth` | 4–9 | 7 | Bayesian optimization |
| `learning_rate` | 0.01–0.2 | 0.05 | Bayesian optimization |
| `subsample` | 0.6–1.0 | 0.8 | Bayesian optimization |
| `colsample_bytree` | 0.5–1.0 | 0.7 | Bayesian optimization |
| `scale_pos_weight` | Class ratio (fixed) | 368.0 | Set to negative/positive class ratio per XGBoost documentation |
| Decline threshold | 0.70–0.90 | 0.82 | Threshold sweep on validation set; 0.82 achieved target <12% false positive rate among declines while maximizing fraud capture |
| Step-up threshold | 0.45–0.65 | 0.55 | Threshold sweep; 0.55 captures incremental fraud with acceptable step-up volume |

Probability calibration (isotonic regression) was applied on the validation set and reduced the Brier score from 0.00487 (uncalibrated) to 0.00312 (calibrated), confirming meaningful improvement in score reliability.

### 5.6 Developmental Testing

- **5-fold time-series cross-validation:** Mean AUC 0.969 ± 0.004 across folds; stable performance across the three-year training window
- **Out-of-time validation (May–Oct 2024):** AUC 0.967 — within 0.002 of cross-validation mean, confirming low overfitting
- **Fraud typology subgroup testing:** Model evaluated separately on the three primary fraud typologies in the training data: credential stuffing (AUC 0.981), account takeover (AUC 0.962), card-not-present new account fraud (AUC 0.944). Lower performance on new account fraud is expected and documented as a known limitation.
- **Latency stress test:** 10,000 concurrent scoring requests processed with p99 latency of 7.8ms (scoring only), well within the 85ms authorization window constraint.

---

## Section 6: Model Performance

### 6.1 Evaluation Metrics and Rationale

| Metric | Type | Rationale |
|---|---|---|
| AUC-ROC | Primary | Threshold-independent measure of fraud/legitimate discrimination; industry standard for fraud models |
| Fraud Detection Rate @ 0.1% False Positive Rate | Primary | Regulatory and business standard: how much fraud is caught while keeping false positives below 0.1% of volume |
| False Positive Rate at production threshold (0.82) | Primary | Directly measures customer friction from incorrect declines |
| KS Statistic | Secondary | Measures maximum separation between fraud and legitimate score distributions |
| Brier Score (calibration) | Secondary | Measures reliability of the probability score |

### 6.2 Holdout Performance Results

| Metric | Training Set | Validation Set | Holdout / Test Set |
|---|---|---|---|
| AUC-ROC | 0.984 | 0.967 | 0.963 |
| Fraud Detection Rate @ 0.1% FPR | 72.4% | 68.9% | 67.8% |
| False Positive Rate @ threshold 0.82 | 9.1% | 10.8% | 11.7% |
| KS Statistic | 0.847 | 0.821 | 0.814 |
| Brier Score | 0.00271 | 0.00312 | 0.00341 |

The holdout AUC of 0.963 represents a 0.004 decline from the validation set (0.967), indicating minimal overfitting. The false positive rate of 11.7% at the production decline threshold meets the objective of < 12%, representing a significant improvement over the FraudGuard Pro baseline of 22%.

### 6.3 Performance Across Material Subgroups

| Subgroup | N (holdout) | AUC-ROC | False Positive Rate @ 0.82 | Notes |
|---|---|---|---|---|
| Credit card transactions | 28,471,000 | 0.971 | 10.9% | Higher fraud rate (0.38%) enables better discrimination |
| Debit card transactions | 18,456,000 | 0.948 | 13.1% | Lower fraud rate (0.18%) makes discrimination harder |
| Transaction amount < $50 | 27,812,000 | 0.941 | 14.2% | Lower-value transactions have noisier fraud signals |
| Transaction amount $50–$500 | 14,293,000 | 0.971 | 10.1% | Core fraud band; strongest model performance |
| Transaction amount > $500 | 4,822,000 | 0.978 | 8.7% | High-value transactions have clearer fraud signals |
| New device (first transaction) | 2,140,000 | 0.956 | 9.3% | Device novelty is a strong feature |
| New account (< 90 days old) | 1,847,000 | 0.931 | 16.8% | Known weaker subgroup; documented in limitations |

### 6.4 Benchmark Comparisons

| Model | AUC-ROC | FPR @ Threshold | Notes |
|---|---|---|---|
| **Selected model (XGBoost v2.1.0)** | **0.963** | **11.7%** | Production model |
| FraudGuard Pro rules engine (prior system) | 0.891 | 22.0% | Prior production system |
| Logistic Regression baseline | 0.941 | 18.4% | Internal benchmark |
| XGBoost v1.0 (prior model version) | 0.948 | 14.9% | Prior ML model version (2023) |

### 6.5 Sensitivity Analysis

**Top 10 features by mean absolute SHAP value (holdout set):**
1. `ip_vpn_proxy_flag` — 0.0847
2. `txn_velocity_1h` — 0.0712
3. `device_novelty_flag` — 0.0683
4. `amount_vs_30d_mean_zscore` — 0.0621
5. `merchant_category_fraud_rate_30d` — 0.0589
6. `ip_country_risk_score` — 0.0534
7. `geographic_distance_km` — 0.0498
8. `txn_velocity_24h` — 0.0461
9. `account_tenure_days` — 0.0387
10. `device_risk_score` — 0.0341

**Sensitivity to feature perturbation:** Setting `ip_vpn_proxy_flag` to 0 (no VPN/proxy detected) for all high-scoring VPN transactions reduced model AUC by 0.008 — confirming this feature's material contribution but also demonstrating the model's resilience through feature redundancy.

### 6.6 Explainability and Interpretability

SHAP values are computed for every transaction scored at or above the step-up threshold (0.55) and stored in the fraud operations case management system. Fraud operations analysts use SHAP visualizations to understand why a transaction was flagged when conducting post-decline review and customer outreach. The top 3 SHAP features for each flagged transaction are surfaced in the fraud analyst UI.

For UDAAP adverse action purposes: declined transactions generate an automatic adverse action notice with reason codes derived from the top SHAP features for that transaction, converted to consumer-readable language (e.g., "Transaction originated from an unusual location" rather than "geographic_distance_km = 847").

---

## Section 7: Assumptions and Limitations

### 7.1 Key Assumptions Registry

| # | Assumption | Sensitivity if Violated | Validation Evidence | Compensating Control |
|---|---|---|---|---|
| 1 | Fraud patterns in the 2022–2025 training window are representative of current fraud typologies | High | Quarterly retraining cadence limits staleness; holdout period includes two distinct fraud pattern shifts (Nov 2024, Jan 2025) without significant performance degradation | Quarterly model retraining; monthly PSI monitoring; fraud typology change alerts from Fraud Ops team |
| 2 | Third-party device intelligence (DeviceIQ Pro) data quality remains stable | Medium | Vendor SLA requires ≥ 98% availability and ≤ 2% null rate; measured at 98.1% in training data | Monthly vendor data quality check; null rate increase > 5% triggers vendor escalation and model review |
| 3 | Chargeback labels accurately reflect true fraud (~94–96% accuracy) | Medium | Industry standard; label noise at this scale and rate is well-tolerated by ensemble methods (validated in literature) | Accepted as within industry norms; fraud operations conducts periodic manual audit of chargeback accuracy |
| 4 | The relationship between behavioral velocity features and fraud probability is stable over time | High | Out-of-time testing across 6-month holdout containing two fraud pattern shifts shows stable AUC (Section 5.6) | Quarterly retraining; PSI monitoring on velocity feature distributions |
| 5 | Model does not produce disparate impact on protected class proxy variables at a rate that would constitute UDAAP violation | High | Pre-production fairness assessment conducted by Stephanie Morales (Validator): no statistically significant disparate impact found at production thresholds | Quarterly disparate impact monitoring; FPR monitored by demographic proxy quintiles |

### 7.2 Known Limitations

| Limitation | Impact | Status |
|---|---|---|
| Lower performance on new accounts (< 90 days): AUC 0.931, FPR 16.8% | Higher false positive rate on new accounts creates friction for legitimate new customers | Accepted — new account fraud rate is also higher (0.61%); fraud capture tradeoff is favorable; new account performance monitored separately |
| 45-day chargeback label lag means the most recent 45 days of training labels are incomplete | Recent fraud patterns may be underrepresented in training data | Mitigated — 45-day cutoff applied to observation period end; accepted as industry-standard limitation |
| Novel fraud typologies not in training data may evade detection | New fraud methods (deepfake voice, novel credential harvesting) may not generate the expected behavioral signals | Mitigated — quarterly retraining; Fraud Ops feeds new confirmed fraud cases back into training data; fraud intelligence subscription monitored by Fraud Analytics |
| 11.7% false positive rate at decline threshold generates ~1,500 incorrect declines per day | Customer friction, potential UDAAP exposure, dispute handling costs | Mitigated — SHAP-based adverse action reason codes; customer-friendly dispute resolution process; false positive rate monitored monthly with action threshold |

### 7.3 Compensating Controls for Known Limitations

**New account false positive rate (16.8%):** New accounts (< 90 days) that receive a declined transaction are automatically flagged for outbound fraud operations contact within 4 hours. New account FPR is monitored separately in the monthly report with a lower action threshold (14%) than the overall model threshold (15%).

**Novel fraud typologies:** The Fraud Analytics team subscribes to two external fraud intelligence feeds (FraudIntel Network, FS-ISAC alerts). New typologies identified via these feeds are evaluated against the current model within 5 business days of alert. If a new typology is generating losses above $500K without model detection, an emergency model update is initiated outside the quarterly retraining cycle.

**False positive customer impact:** All automatically declined transactions generate a real-time notification to the cardholder and include a one-tap dispute initiation link. Dispute handling target resolution time is 2 business days.

### 7.4 Conditions Under Which the Model May Fail or Underperform

- **Novel fraud typology emergence:** A coordinated fraud campaign using techniques not represented in the training data (e.g., AI-generated synthetic transaction behavior designed to mimic legitimate cardholder patterns) could significantly reduce detection rates before the quarterly retraining cycle.
- **Upstream data outage:** Outage of the DeviceIQ Pro or GeoRisk APIs causes 14 device and IP features to fall back to null, reducing model performance. At 100% null on these features, estimated AUC decreases to approximately 0.941.
- **Behavioral feature store staleness:** If the feature store refresh fails, behavioral velocity features become stale. Features more than 6 hours old are flagged; model continues to score but with reduced accuracy on velocity signals.
- **Adversarial gaming:** Sophisticated fraudsters who identify and specifically avoid model trigger patterns (e.g., always using VPN-free IPs, maintaining low velocity before a burst) could systematically evade detection. This is monitored via fraud intelligence feeds.

### 7.5 Appropriate Use Boundaries

This model is valid for consumer CNP credit and debit card transactions processed through Westbrook's authorization network. It is not valid for: card-present transactions, commercial card transactions, ACH or wire transactions, account opening decisions, or any purpose other than real-time CNP fraud scoring.

---

## Section 8: Third-Party and Vendor Components

### 8.1 Third-Party Components Inventory

| Component | Vendor / Source | Version | Type | Purpose |
|---|---|---|---|---|
| DeviceIQ Pro | DeviceIQ Inc. | API v3.1 | Data / API | 14 device intelligence features per transaction |
| GeoRisk | GeoRisk Ltd. | API v2.4 | Data / API | IP geolocation, VPN/proxy detection features |
| XGBoost | Open source (DMLC) | 2.1.0 | Library | Core gradient boosting algorithm |
| scikit-learn | Open source | 1.5.0 | Library | Pipeline wrapper, isotonic calibration |
| SHAP | Open source | 0.46.0 | Library | Feature importance and individual prediction explanation |

### 8.2 Vendor Validation Approach

**DeviceIQ Pro:** Vendor provided API documentation, methodology overview (device fingerprinting approach), and a data quality SLA (≥ 98% availability, ≤ 2% null rate). Westbrook validated device feature quality by comparing a 30-day production sample against the vendor's published field definitions. Feature null rate measured at 1.9%, within SLA. No access to underlying device fingerprinting algorithm — validated behaviorally via feature contribution to model performance (14 device features represent 18% of total SHAP importance).

**GeoRisk:** Similar vendor validation approach. IP geolocation accuracy spot-checked against known-location test transactions (bank-initiated test cards at known locations): 97.2% city-level accuracy. VPN/proxy detection verified against a curated list of known VPN exit nodes: 94.1% detection rate.

### 8.3 Customizations and Adjustments

No customizations to vendor APIs. The primary customization is the feature engineering applied to vendor outputs before model training (combining raw vendor scores with internal behavioral data, computing derived features, handling nulls). These transformations are documented in Section 4.4.

### 8.4 Ongoing Vendor Oversight

DeviceIQ Pro and GeoRisk null rates are monitored in real time via the production data pipeline dashboard. A null rate spike > 5% above baseline triggers an automated alert to the ML Engineering on-call. Quarterly vendor business reviews include data quality SLA compliance checks. Vendor contract renewals include re-evaluation of data quality against model performance contribution.

---

## Section 9: GenAI and LLM Governance

N/A — This is a traditional gradient boosting model with no generative AI or LLM components.

---

## Section 10: Validation and Monitoring Plan

### 10.1 Pre-Production Validation Requirements

| Validation Activity | Responsible Party | Required Before Production? | Status |
|---|---|---|---|
| Conceptual soundness review | Stephanie Morales | Yes | Complete |
| Data quality independent assessment | Stephanie Morales | Yes | Complete |
| Performance replication on holdout set | Stephanie Morales | Yes | Complete |
| Fairness / disparate impact assessment | Stephanie Morales | Yes | Complete |
| Vendor component validation review | Stephanie Morales | Yes | Complete |
| Latency / infrastructure validation | Tyler Brooks (ML Eng) | Yes | Complete |
| Threshold calibration review | Lawrence Kim (MRM) | Yes | Complete |
| Ongoing monitoring plan review | Lawrence Kim (MRM) | Yes | Complete |

### 10.2 Ongoing Monitoring Metrics and Thresholds

| Metric | Baseline (Development) | Warning Threshold | Action Threshold | Action Required |
|---|---|---|---|---|
| AUC-ROC (monthly backtesting) | 0.963 | < 0.950 | < 0.940 | MRM notification within 2 business days; model review within 10 |
| False Positive Rate @ threshold 0.82 | 11.7% | > 14.0% | > 16.0% | Customer impact review; threshold re-calibration evaluation |
| Fraud Detection Rate @ 0.1% FPR | 67.8% | < 63.0% | < 59.0% | MRM escalation; model update evaluation |
| PSI (score distribution) | ~0.0 | > 0.10 | > 0.25 | Distribution shift investigation |
| DeviceIQ Pro null rate | 1.9% | > 5.0% | > 10.0% | Vendor escalation; feature imputation review |
| New account FPR | 16.8% | > 19.0% | > 22.0% | New account subgroup model review |
| Disparate impact ratio (FPR by demographic proxy) | < 1.10x across quintiles | > 1.20x | > 1.30x | UDAAP legal review; threshold adjustment evaluation |
| Override rate (fraud ops) | 11.7% | > 18.0% | > 22.0% | Model quality review; analyst feedback session |
| p99 inference latency | 7.8ms | > 40ms | > 65ms | Infrastructure scaling; ML Engineering escalation |

### 10.3 Monitoring Frequency and Reporting

| Activity | Frequency | Owner | Report Delivered To |
|---|---|---|---|
| Performance monitoring report | Monthly | Kevin Osei / Fraud Analytics | Lawrence Kim (MRM), Patricia Vance (Model Owner) |
| Disparate impact report | Quarterly | Stephanie Morales (Validator) | Lawrence Kim (MRM), Legal/Compliance |
| Vendor data quality check | Monthly | ML Engineering | Kevin Osei |
| Model quarterly retraining | Quarterly | Kevin Osei | Lawrence Kim (MRM) for retraining approval |
| Full model validation | Annual | Stephanie Morales | MRM Committee |
| Real-time latency monitoring | Continuous | ML Engineering (automated alerts) | On-call ML Engineer |

### 10.4 Performance Deterioration Triggers and Escalation

If any Action Threshold is breached: MRM notified within 2 business days. Model review convened within 10 business days. Emergency retraining may be initiated for severe fraud typology shifts (> $1M unexplained loss increase in 30 days). Possible outcomes: (1) threshold recalibration; (2) out-of-cycle retraining with expedited validation; (3) temporary reversion to FraudGuard Pro rules engine as fallback; (4) model retirement and redevelopment.

### 10.5 Override and Adjustment Policy

The fraud operations team may escalate declined transactions for manual review, effectively overriding the model decline decision. Overrides are logged in the case management system with reason code. Override rates above 22% trigger a model quality review — high override rates indicate analyst disagreement with model decisions.

Score threshold adjustments (calibration changes) are treated as material model changes requiring MRM review and updated documentation in Appendix C.

### 10.6 Redevelopment Criteria

Model redevelopment triggered by: (1) AUC below 0.940 for two consecutive months; (2) a novel fraud typology causing > $2M in undetected losses over 30 days that cannot be addressed through retraining on existing features; (3) a major change in Westbrook's card product mix or transaction processing infrastructure; (4) a UDAAP finding related to disparate impact that cannot be resolved through threshold adjustment.

---

## Section 11: Implementation and Deployment

### 11.1 Production Architecture

The scoring service is a Python FastAPI application deployed on Westbrook's on-premises Kubernetes cluster (3 pods, auto-scaling to 8 under peak load). Each authorization request from the Fiserv DNA core banking system passes through the API Gateway to the scoring service. The service retrieves real-time device intelligence and IP geolocation from vendor APIs (< 10ms each), fetches precomputed behavioral aggregates from the internal feature store (Redis cache, < 5ms), assembles the 187-feature vector, scores with the XGBoost model, and returns the result to the authorization engine — all within 85ms.

### 11.2 Input/Output Specifications

| Field | Description | Type | Format / Range | Required? |
|---|---|---|---|---|
| **Input: transaction_id** | Unique transaction identifier | string | UUID | Yes |
| **Input: account_id** | Masked account identifier | string | Internal format | Yes |
| **Input: transaction_amount** | Transaction amount in USD | float | 0.01–99,999.99 | Yes |
| **Input: merchant_category_code** | ISO 18245 MCC | string | 4-digit code | Yes |
| **Input: transaction_timestamp** | UTC timestamp | datetime | ISO 8601 | Yes |
| **Input: device_fingerprint_id** | Device fingerprint token | string | DeviceIQ Pro format | No (null handled) |
| **Input: ip_address** | Masked IP address | string | IPv4/IPv6 | No (null handled) |
| **Output: fraud_score** | Fraud probability | float | 0.0–1.0 | — |
| **Output: action** | Routing decision | string | `"decline"`, `"step_up"`, `"approve"` | — |
| **Output: top_shap_features** | Top 3 SHAP features | list[str] | Feature names | — |

### 11.3 Performance and Latency Requirements

| Requirement | Target | Notes |
|---|---|---|
| **End-to-end latency (p50)** | < 35ms | Measured from authorization request receipt to score return |
| **End-to-end latency (p99)** | < 85ms | Authorization network SLA; exceeding this causes authorization timeout |
| **Throughput** | 5,000 transactions/second sustained | Peak observed: 4,100/sec (Cyber Monday 2024); 5,000 target provides 22% headroom |
| **Availability SLA** | 99.95% | < 4.4 hours downtime per year; scored as critical infrastructure |

### 11.4 Integration Dependencies

| Dependency | System | Criticality | Fallback if Unavailable |
|---|---|---|---|
| DeviceIQ Pro API | Vendor | High | Null imputation for device features; model continues with reduced feature set |
| GeoRisk API | Vendor | High | Null imputation for IP features; model continues with reduced feature set |
| Internal Feature Store (Redis) | Internal | Critical | Fallback to cold feature computation (adds ~15ms latency); acceptable within 85ms SLA |
| Fiserv DNA Authorization Engine | Internal | Critical | Not applicable — if authorization engine is down, no transactions are processed |

### 11.5 Rollback Plan

Rollback to FraudGuard Pro rules engine can be executed within 15 minutes by the ML Engineering on-call by updating the authorization engine routing configuration. Rollback is triggered by the Model Owner or MRM for: p99 latency sustained > 85ms, AUC breach of Action Threshold, or regulatory directive. During rollback, FraudGuard Pro assumes full fraud scoring responsibility. Rollback decision authority rests with the Model Owner in coordination with MRM and the CRO.

---

## Section 12: Governance

### 12.1 Roles and Responsibilities

| Role | Name / Team | Responsibilities |
|---|---|---|
| **Model Owner** | Patricia Vance, SVP Fraud Risk Management | Accountable for fraud performance outcomes; approves threshold changes and material model changes |
| **Lead Developer** | Kevin Osei, Lead Data Scientist | Model development, quarterly retraining, documentation |
| **MRM / Validator** | Lawrence Kim (MRM); Stephanie Morales (Validator) | Independent validation; quarterly disparate impact report; ongoing oversight |
| **Business User(s)** | Fraud Operations Team | Reviews declined transactions; provides confirmed fraud feedback for retraining |
| **Data Owners** | Card Operations (transactions); Vendor Management (DeviceIQ, GeoRisk) | Data quality and vendor contract oversight |
| **IT / MLOps** | ML Engineering (Tyler Brooks lead) | Production deployment, Kubernetes infrastructure, latency monitoring, vendor API integration |
| **Internal Audit** | Internal Audit — Technology & Risk | Annual model risk management effectiveness review |

### 12.2 Model Inventory Registration

| Field | Value |
|---|---|
| **Model inventory system** | Westbrook ModelRegistry (internal) |
| **Model ID in inventory** | MDL-2025-017 |
| **Inventory registration date** | 2025-03-15 |
| **Tier in model inventory** | Tier 1 — Critical Materiality |

### 12.3 Change Management

Quarterly retraining (new data only, no structural changes) requires MRM notification and performance sign-off but not full re-validation. Structural changes (new features, new algorithm, threshold changes, architecture changes) require full MRM review and targeted or full re-validation depending on the scope of change. All changes logged in Appendix C.

### 12.4 Document Retention

This MDD and all supporting artifacts shall be retained for a minimum of 7 years following model retirement per Westbrook's Model Risk Management Policy.

---

## Appendix A: Glossary

| Term | Definition |
|---|---|
| **AUC-ROC** | Area Under the Receiver Operating Characteristic Curve; threshold-independent discrimination measure |
| **Card-Not-Present (CNP)** | Transaction where the physical card is not used at a terminal — primarily e-commerce and phone orders |
| **Chargeback** | A transaction reversal initiated by the cardholder's bank following a fraud dispute |
| **Class imbalance** | Severe disproportion between fraud (0.271%) and legitimate (99.729%) transactions; addressed via `scale_pos_weight` |
| **False positive** | A legitimate transaction scored above the decline threshold and incorrectly declined |
| **KS Statistic** | Kolmogorov-Smirnov statistic; maximum separation between fraud and legitimate score distributions |
| **PSI** | Population Stability Index; measures score distribution shift between development and production |
| **SHAP** | SHapley Additive exPlanations — method for computing feature contributions to individual model predictions |
| **Step-up authentication** | Additional verification (SMS OTP) triggered for medium-risk transactions |
| **SR 26-2** | Supervisory Guidance on Model Risk Management (April 17, 2026) |
| **UDAAP** | Unfair, Deceptive, or Abusive Acts or Practices — CFPB consumer protection standard |

---

## Appendix B: References and Source Documents

| Reference | Description | Location |
|---|---|---|
| SR 26-2 | Supervisory Guidance on Model Risk Management (April 17, 2026) | `_mrm-mdd-kit/governance/sr26-2.md` |
| DeviceIQ Pro API Documentation | Vendor methodology and field definitions | Westbrook ModelRegistry document repository |
| GeoRisk API Documentation | Vendor methodology and accuracy benchmarks | Westbrook ModelRegistry document repository |
| Validation Report — MDL-2025-017 v2.1 | Independent validation report by Stephanie Morales | Westbrook ModelRegistry |
| Model artifact | Serialized XGBoost model + calibration object | `models/fraud_xgboost_v2.1.0.pkl` in model repository |
| SHAP library documentation | Explainability methodology | https://shap.readthedocs.io |

---

## Appendix C: Model Change Log

| Version | Date | Author | Type of Change | MRM Approval Required? | Summary |
|---|---|---|---|---|---|
| 1.0.0 | 2023-07-01 | Kevin Osei | Initial production release (replaced FraudGuard Pro) | N/A | First ML-based fraud model; 147 features; AUC 0.948 |
| 1.1.0 | 2023-10-15 | Kevin Osei | Quarterly retraining — new data only | MRM notification | Retrained on Jan 2022 – Aug 2023 data; AUC 0.951 |
| 1.2.0 | 2024-01-20 | Kevin Osei | Quarterly retraining — new data only | MRM notification | Retrained; incorporated Nov 2023 credential stuffing patterns; AUC 0.956 |
| 2.0.0 | 2025-06-20 | Kevin Osei | Major redevelopment | Full re-validation | Expanded feature set (147 → 187 features); SHAP explainability added; retrained on 2022–2024 data; AUC 0.967 |
| 2.1.0 | 2025-10-01 | Kevin Osei | Threshold recalibration + quarterly retraining | MRM review | Decline threshold adjusted 0.80 → 0.82 to reduce FPR; retrained on 2022–Feb 2025 data |

---

## Appendix D: Experiment Log

| Experiment ID | Date | Hypothesis | Approach | Result | Decision |
|---|---|---|---|---|---|
| EXP-2025-001 | 2025-03-15 | LightGBM matches XGBoost with lower latency | LightGBM trained on full feature set | AUC 0.967 vs. XGBoost 0.971; p99 latency 12ms vs. 7.8ms | XGBoost retained; marginal AUC advantage + lower latency |
| EXP-2025-002 | 2025-04-02 | Neural network (TabNet) outperforms XGBoost | TabNet trained on full feature set | AUC 0.968; p99 latency 47ms — exceeds budget | Rejected; latency constraint binding |
| EXP-2025-003 | 2025-04-18 | Behavioral features alone sufficient (remove vendor features) | XGBoost trained on 173 internal features only | AUC 0.951 — 0.012 decline | Rejected; vendor features provide material lift |
| EXP-2025-004 | 2025-05-10 | Isotonic calibration improves score reliability | Applied isotonic regression on validation set | Brier score improved 0.00487 → 0.00312 | Adopted; calibration applied to production model |
| EXP-2025-005 | 2025-06-01 | Decline threshold 0.82 achieves <12% FPR target | Threshold sweep 0.70–0.90 on validation set | 0.82 achieves 10.8% FPR on validation; 11.7% on holdout | Adopted as production threshold |
