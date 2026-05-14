# Model Development Document (MDD)

> **Kit version:** mrm-mdd-kit v1.0.0 | **Regulatory basis:** SR 26-2 (April 17, 2026)
>
> **FICTIONAL EXAMPLE** — All institution names, personnel, performance metrics, and business details are completely made up. This document exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## Document Control

| Field | Value |
|---|---|
| **Model Name** | Personal Loan Probability of Default Scorecard |
| **Model ID** | MDL-2025-031 |
| **Model Type** | Random Forest Binary Classifier |
| **Model Version** | 1.0.0 |
| **Document Version** | 1.1 |
| **Document Status** | Approved |
| **Development Start Date** | 2025-02-01 |
| **Document Date** | 2025-07-10 |
| **Effective Date** | 2025-09-01 |
| **Next Review Date** | 2026-09-01 |
| **Model Owner** | Rebecca Santos, SVP Consumer Lending |
| **Lead Developer** | Nathan Park, Senior Data Scientist, Model Risk & Analytics |
| **Contributing Developers** | N/A |
| **Primary Validator** | Carlos Reyes, Senior Model Validator |
| **MRM Contact** | Helen Yu, Director of Model Risk Management |

### Approval Sign-offs

| Role | Name | Signature | Date |
|---|---|---|---|
| Lead Developer | Nathan Park | *(signed)* | 2025-07-10 |
| Model Owner | Rebecca Santos | *(signed)* | 2025-07-14 |
| MRM / Validation Lead | Carlos Reyes | *(signed)* | 2025-07-28 |
| Chief Risk Officer (or delegate) | David Okonkwo, Deputy CRO | *(signed)* | 2025-08-04 |

### Document Version History

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| 1.0 | 2025-06-20 | Nathan Park | Initial draft |
| 1.1 | 2025-07-10 | Nathan Park | Expanded fair lending section per MRM pre-review; updated Section 7 assumptions registry |

---

## Section 1: Executive Summary

### 1.1 Model Overview

The Personal Loan Probability of Default Scorecard is a Random Forest classifier that estimates the probability that a personal loan applicant will become 90+ days delinquent within 24 months of origination. The model scores each application at underwriting time, producing a probability score (0.0–1.0) that is converted to a 300–850 scorecard scale for underwriter use. Underwriters use the score as their primary credit risk input: applicants scoring below a decline threshold are recommended for decline; applicants in a referral band are reviewed by a senior underwriter; applicants above the approval threshold proceed to standard terms and conditions. No application is declined without a human underwriter decision — the model informs but does not replace human credit judgment.

### 1.2 Model Risk Rating

| Dimension | Rating | Rationale |
|---|---|---|
| **Inherent Risk** | Medium | Random Forest is moderately complex; partially interpretable via SHAP; 47 features, primarily credit bureau and application data; key assumption of credit bureau data accuracy |
| **Model Exposure** | High | Model score is the primary determinant of approve/decline recommendations for all personal loan applications; approximately 8,400 applications per month; average loan size $12,400 |
| **Model Purpose** | High | Origination credit decisions; directly affects consumer access to credit; ECOA/Reg B applicable; adverse action reason codes required |
| **Overall Materiality** | High | High exposure and high purpose risk; credit decision model with ECOA applicability requires full independent validation and annual review |

### 1.3 Key Risks and Findings

- **Fair lending / disparate impact risk:** The model uses proxy variables (zip code, income-to-debt ratio, employment type) that may correlate with protected class characteristics. A pre-production disparate impact assessment found no statistically significant disparate impact at production thresholds, but ongoing monitoring is required. Full treatment in Sections 7.1 and 10.2.
- **Random Forest interpretability gap:** Random Forest does not produce natively interpretable individual predictions. SHAP values are computed for each decision but require domain expertise to interpret. Adverse action reason codes are derived from SHAP but involve a mapping step that introduces minor approximation. Full treatment in Section 6.6.
- **Thin-file applicants:** The model performs less well on applicants with fewer than 12 months of credit history (AUC 0.744 vs. 0.831 overall). Thin-file applicants represent approximately 18% of the application volume through the digital channel. Full treatment in Sections 6.3 and 7.2.
- **Economic cycle sensitivity:** The model was trained on 2020–2024 data which includes a post-COVID credit normalization period. Performance in a severe recession scenario is estimated through stress testing but has not been empirically validated. Full treatment in Section 7.1.

### 1.4 Regulatory Applicability

| Regulation | Applicable? | Notes |
|---|---|---|
| SR 26-2 — Model Risk Management | Yes | Traditional ML model; $28B institution (applies by spirit of guidance given significant model exposure) |
| ECOA / Reg B — Fair Lending | Yes | Model output directly determines credit approval recommendations; adverse action reason codes required for declines |
| CECL / FASB ASC 326 | No | This is an origination scorecard, not a portfolio loss reserve model |
| BSA / AML — Bank Secrecy Act | No | Not applicable |
| CFPB UDAAP | Yes | Consumer credit product; disparate impact monitoring required |

---

## Section 2: Model Purpose and Business Context

### 2.1 Business Problem and Objectives

Cascade Consumer Lending originates approximately 8,400 personal loan applications per month across branch and digital channels. Prior to this model, Cascade relied on a vendor scorecard (CreditView Analytics, last recalibrated 2020) as its primary credit risk assessment tool. Internal backtesting on the 2023–2024 applicant population showed the vendor scorecard AUC had declined to 0.721 — below Cascade's 0.75 minimum MRM policy threshold — primarily due to the growth of thin-file and young-credit applicants through the digital origination channel, a population underrepresented in the vendor scorecard's 2018–2019 training data.

Objectives: (1) develop an internally owned credit scoring model achieving AUC ≥ 0.80 on the current applicant population; (2) improve thin-file applicant discrimination from AUC 0.689 (vendor scorecard) to ≥ 0.74; (3) produce interpretable SHAP-based adverse action reason codes compliant with ECOA/Reg B; (4) eliminate vendor dependency for the primary credit decision tool.

### 2.2 Intended Use

The model produces a fraud probability score (0.0–1.0) converted to a 300–850 point scorecard scale at application time. Three score bands drive three underwriting outcomes: (1) score < 580 → decline recommendation (senior underwriter reviews before final decision); (2) score 580–659 → referral to senior underwriter for judgment-based review; (3) score ≥ 660 → standard approval recommendation (underwriter confirms completeness and approves). All declines require a human underwriter decision — no application is declined by model output alone. Adverse action notices are generated for all declined applications with SHAP-derived reason codes.

### 2.3 Intended Users

| User / System | How Output Is Used |
|---|---|
| Loan Origination System (LoanPath) | Receives model score; presents to underwriter with score band and recommendation |
| Underwriters | Use score as primary credit risk input; make final approve/decline/refer decision |
| Senior Underwriters | Review all referral band and decline-recommended applications |
| Compliance Team | Reviews adverse action notices and reason codes for Reg B compliance |
| Helen Yu, MRM | Reviews quarterly performance report |

### 2.4 Out-of-Scope and Prohibited Uses

- This model must not be used as the sole basis for a credit decline; every decline requires a human underwriter decision.
- This model must not be used for loan renewals, credit limit increases, or collections decisions; it was developed on origination data only.
- This model must not be applied to commercial or small business loan applications.
- This model must not be used for any purpose other than Cascade personal loan origination without MRM review and re-validation.
- Score outputs must not be shared externally or used for marketing segmentation without Compliance review.

### 2.5 Downstream Decisions and Impact

Approximately 8,400 applications per month with an average requested loan amount of $12,400 (total monthly origination exposure: approximately $104 million at full approval). Historical approval rate of approximately 42% means approximately 3,528 loans per month are approved. Model score is the primary determinant of approve/decline/refer recommendation for each application. Incorrect declines (false positives for default) result in Cascade losing creditworthy borrowers; incorrect approvals (false negatives for default) result in credit losses.

### 2.6 Model Dependencies

| Dependency | Type | Description |
|---|---|---|
| Credit Bureau Pull (TransUnion) | Upstream | Primary credit bureau data; 31 of 47 features sourced from bureau pull at application time |
| Loan Origination System (LoanPath) | Upstream / Downstream | Provides application data inputs; receives score and recommendation |
| Adverse Action Notice Generator | Downstream | Receives SHAP reason codes; generates Reg B-compliant adverse action notices |

---

## Section 3: Model Risk Assessment

### 3.1 Inherent Risk Assessment

| Risk Factor | Assessment | Rationale |
|---|---|---|
| **Model complexity** | Medium | Random Forest with 300 trees is moderately complex; more interpretable than deep learning; less interpretable than logistic regression; 47 features is manageable |
| **Number and criticality of assumptions** | Medium | Key assumptions: credit bureau data accuracy, stability of credit behavior relationship over time, sufficient outcome observation window |
| **Input data quality** | Medium | Credit bureau data is high quality for thick-file applicants; thin-file applicants have sparse bureau data, creating higher uncertainty in scores for that segment |
| **Data constraints / availability** | Medium | 4.5 years of origination data (2020–2024) with 24-month outcome window means 2022 originations are the most recent with complete outcomes; captures COVID normalization but limited severe recession data |
| **Model interpretability** | Medium | SHAP values provide post-hoc interpretability; but Random Forest does not natively produce reason codes; SHAP-to-reason-code mapping involves a rule-based translation layer |
| **Novelty of approach** | Low | Random Forest is a well-established and widely validated algorithm; well-accepted by banking regulators |
| **Overall Inherent Risk** | Medium | Established algorithm partially offset by credit bureau data quality variation and interpretability limitations |

### 3.2 Model Exposure Assessment

| Exposure Factor | Value | Notes |
|---|---|---|
| **Portfolio / business scope** | ~8,400 applications/month; ~$104M monthly origination exposure | All Cascade personal loan applications |
| **Frequency of model use** | Per application; approximately 280 per business day | Batch-scored at underwriting |
| **Business impact of incorrect output** | High | False negatives (missed defaults) → credit losses; false positives (declined creditworthy borrowers) → lost revenue, ECOA risk |
| **Degree of automation** | Human-in-the-loop | All decline recommendations reviewed by human underwriter before final decision |
| **Overall Exposure** | High | High dollar exposure; all applications; credit access implications |

### 3.3 Model Purpose Assessment

| Purpose Factor | Assessment | Notes |
|---|---|---|
| **Regulatory-facing use** | Yes | ECOA/Reg B applies; adverse action reason codes required |
| **Financial risk management use** | Yes | Primary credit risk control for personal loan origination |
| **Consumer-impacting decisions** | Yes | Access to credit is a direct consumer financial impact |
| **Overall Purpose Risk** | High | All three factors apply |

### 3.4 Overall Materiality Determination

| | |
|---|---|
| **Overall Materiality** | High |
| **Validation rigor required** | Full independent validation including conceptual soundness, data quality, performance replication, fair lending assessment, and SHAP reason code validation |
| **Monitoring frequency** | Quarterly |
| **Rationale** | High exposure (all personal loan originations, $104M/month) and high purpose risk (ECOA/Reg B credit decisions) drive High overall materiality. Quarterly monitoring with annual full validation. |

---

## Section 4: Data

### 4.1 Data Sources and Provenance

| Data Source | System of Record | Data Owner | Data Type | Refresh Frequency |
|---|---|---|---|---|
| Credit Bureau Pull (TransUnion) | TransUnion API | Credit Risk | 31 credit bureau attributes at application time | Per application (real-time pull) |
| Loan Application Data | LoanPath LOS | Consumer Lending Ops | Application-level: loan amount, purpose, income, employment, assets | Per application |
| Loan Performance Data (outcomes) | Core Banking System | Credit Risk Reporting | 24-month performance outcomes: days past due, charge-off, prepayment | Monthly extract |

### 4.2 Observation Period and Sample Design

| | |
|---|---|
| **Observation period** | January 1, 2020 – December 31, 2022 (origination cohorts with 24-month observation window complete through Dec 2024) |
| **Sample size** | 186,420 originated loans |
| **Geographic scope** | All Cascade operating states (Oregon, Washington, Idaho, Montana, Nevada) |
| **Product scope** | Unsecured personal installment loans $1,000–$50,000 only |
| **Exclusion criteria** | Loans with incomplete outcome windows excluded (originations after Dec 2022); loans with missing bureau data at origination excluded (1.2%); fraudulently obtained loans excluded from good/bad definition (0.3%) |
| **Rationale for observation period** | 3-year origination window provides 186,420 loans with complete 24-month outcomes, capturing the post-COVID credit normalization period (2020–2021), rate normalization (2022). Pre-2020 data excluded due to change in underwriting policy and origination criteria that make earlier cohorts non-representative. |

### 4.3 Data Quality Assessment

| Quality Dimension | Finding | Action Taken |
|---|---|---|
| **Completeness** | Credit bureau features: 98.8% complete overall; 84.3% complete for thin-file applicants (< 12 months history). Application data: 99.4% complete. | Thin-file missing bureau features imputed with population median; a thin-file indicator flag added as a feature |
| **Accuracy** | Bureau attributes spot-checked against 500 manually reviewed credit files; 99.1% accuracy for thick-file applicants; 97.4% for thin-file | Accepted as within data quality standards |
| **Timeliness** | Bureau data pulled at origination time; no timeliness concern for training data | N/A |
| **Consistency** | Loan purpose categories recoded in June 2021 (5 categories → 8 categories); pre-2021 purposes mapped to new taxonomy | Mapping applied and validated on 1,000-record sample |
| **Representativeness** | Default rate varies by channel: branch 4.7%, digital 7.3%. Digital channel grew from 31% to 54% of volume over the observation period. | Channel flag included as feature; model trained on combined population; channel-level performance monitored separately |

### 4.4 Feature Engineering and Preprocessing

| Feature / Transformation | Description | Rationale |
|---|---|---|
| Debt-to-income ratio | Total monthly debt obligations / gross monthly income | Core creditworthiness indicator; derived from application income + bureau trade line data |
| Revolving utilization | Total revolving balance / total revolving limit | High utilization is a strong default predictor independent of score |
| Derogatory mark count | Count of derogatory marks (collections, charge-offs, public records) in past 24 months | Recency-weighted derogatory history |
| Payment-to-income ratio | Requested monthly payment / gross monthly income | Affordability measure; not directly in bureau data |
| Employment tenure (binned) | Employment tenure in months, binned: 0–6, 7–24, 25–60, 60+ | Non-linear relationship with default; binning captures diminishing effect |
| Loan purpose encoding | One-hot encoding of 8 loan purpose categories | Debt consolidation and medical purposes have different default patterns |
| Thin-file indicator | Binary flag: fewer than 12 months of credit history | Enables model to differentiate behavior for sparse-data applicants |
| Bureau score (TransUnion) | TransUnion credit score as a feature (not the output) | Strong prior; included alongside behavioral features for model context |

### 4.5 Train / Validation / Test Split

| Split | Size | Time Period | Notes |
|---|---|---|---|
| **Training set** | 130,494 (70%) | Jan 2020 – Sep 2021 | Random stratified split within origination cohort; stratified by default/non-default and channel |
| **Validation set** | 27,963 (15%) | Jan 2020 – Sep 2021 | Same cohort as training; stratified holdout within the cohort for hyperparameter tuning |
| **Test / holdout set** | 27,963 (15%) | Oct 2021 – Dec 2022 | Out-of-time holdout; all loans with complete 24-month outcomes from later cohorts |

An out-of-time test set (Oct 2021 – Dec 2022 originations) provides a meaningful real-world validation since these applicants were scored by a different underwriting environment than the training cohort, and their performance outcomes are independent of the training data period.

### 4.6 Data Assumptions and Limitations

The outcome definition (90+ days delinquent within 24 months) assumes 24 months is a sufficient performance window for a personal installment loan. Loans with 12–24 month maturities may be fully paid off before a default opportunity arises, right-censoring the outcome. The training data does not include a severe recession vintage (the 2020 COVID vintage represents a mild stress period with significant government support); model performance in a severe recession is estimated via stress testing but not empirically validated.

---

## Section 5: Model Development

### 5.1 Problem Framing and Conceptual Approach

Personal loan default prediction is framed as a binary classification: does the applicant default (90+ days delinquent) within 24 months of origination (1) or not (0)? Default rate in the training population is 6.1%. The 24-month outcome window was chosen to balance completeness (sufficient time to observe defaults) with data recency (longer windows require older origination cohorts).

The conceptual approach follows the established credit scoring theory that default probability is a function of willingness and ability to repay — captured respectively by credit history attributes (willingness) and income, debt burden, and affordability measures (ability). Random Forest was chosen over logistic regression to capture non-linear interactions (e.g., the interaction between high utilization and thin credit history is more predictive than either feature alone).

### 5.2 Methodology Selection

**Selected approach:** Random Forest classifier with 300 trees, max features = sqrt(n_features), balanced class weight. SHAP values computed post-prediction for adverse action reason code generation.

**Rationale:** Random Forest captures non-linear feature interactions important for thin-file applicant discrimination without the black-box opacity of deep learning. It outperformed logistic regression on thin-file applicants (the primary improvement objective) while remaining within acceptable interpretability bounds through SHAP. Gradient boosting (XGBoost) achieved marginally higher overall AUC but had lower thin-file AUC, making Random Forest the preferred choice for this use case.

**Alternatives considered:**

| Alternative Approach | Reason Not Selected |
|---|---|
| CreditView Analytics vendor scorecard (incumbent) | AUC 0.721 on current population; below MRM 0.75 minimum threshold; thin-file AUC 0.689 |
| Logistic Regression | Overall AUC 0.801; thin-file AUC 0.734 — below the 0.74 target; cannot capture non-linear interactions |
| XGBoost Gradient Boosting | Overall AUC 0.836 (vs. Random Forest 0.831); thin-file AUC 0.741 (vs. Random Forest 0.751) — Random Forest preferred for thin-file performance |
| Neural Network (MLP) | AUC 0.829; thin-file AUC 0.748; lower explainability than Random Forest; insufficient improvement to justify added complexity |

### 5.3 Model Architecture and Specifications

| Specification | Value |
|---|---|
| **Algorithm / framework** | Random Forest (scikit-learn 1.5.0); Python scoring service |
| **Number of features (inputs)** | 47 features across credit bureau, application, and derived categories |
| **Output type** | Continuous probability (0.0–1.0), converted to 300–850 scorecard scale |
| **Output range** | 300–850 (scaled) |
| **Decision threshold(s)** | < 580 → decline recommendation; 580–659 → referral; ≥ 660 → approval recommendation |
| **Model size / complexity** | 300 trees; max depth 15; serialized artifact: 24MB |
| **Inference latency** | Median 12ms; p99 28ms per application |

### 5.4 Training Procedure

- **Algorithm:** Random Forest with `n_estimators=300`, `max_depth=15`, `max_features='sqrt'`, `class_weight='balanced'`
- **Class imbalance:** `class_weight='balanced'` applies inverse-frequency weighting to address 6.1% default rate
- **No probability calibration applied:** Random Forest probabilities were assessed via reliability diagrams on the validation set; calibration was within acceptable bounds (Brier score 0.0521) without additional calibration
- **Hardware:** CPU training on 32-core server; training time approximately 18 minutes
- **Reproducibility:** `random_state=42`; scikit-learn version pinned

### 5.5 Hyperparameter Selection and Calibration

| Hyperparameter | Search Range | Final Value | Selection Method |
|---|---|---|---|
| `n_estimators` | 100–500 | 300 | 5-fold CV on validation set; diminishing returns above 300 |
| `max_depth` | 8–20 | 15 | Grid search; depth 15 optimal on validation AUC |
| `max_features` | 'sqrt', 'log2', 0.5 | 'sqrt' | Grid search; 'sqrt' standard and performed best |
| `min_samples_leaf` | 5, 10, 20 | 10 | Controls overfitting; 10 optimal |
| Decline threshold (scorecard) | 540–620 | 580 (scorecard) | Threshold sweep; 580 achieves target approval rate and default rate targets |
| Referral threshold | 620–680 | 660 (scorecard) | Sized to refer approximately 15% of applications to senior underwriter review |

### 5.6 Developmental Testing

- **5-fold stratified cross-validation (training set):** Mean AUC 0.828 ± 0.006; stable performance across folds
- **Out-of-time test set:** AUC 0.831 — slightly above cross-validation mean, indicating no overfitting to the training cohort
- **Thin-file applicant subgroup (validation):** AUC 0.751 — meets the 0.74 objective; significant improvement over the vendor scorecard's 0.689
- **Score distribution stability:** Kolmogorov-Smirnov test comparing score distributions of 2020–2021 originations vs. 2022 originations: D-statistic 0.047 (p > 0.20), indicating stable score distribution across cohorts

---

## Section 6: Model Performance

### 6.1 Evaluation Metrics and Rationale

| Metric | Type | Rationale |
|---|---|---|
| AUC-ROC | Primary | Industry standard for credit scoring discrimination; threshold-independent |
| Gini Coefficient | Primary | 2 × AUC – 1; widely used in credit risk as a discrimination measure |
| KS Statistic | Primary | Industry standard; maximum separation between default and non-default score distributions |
| Default Rate by Score Band | Primary | Business performance metric; validates that score bands produce the expected risk stratification |
| Brier Score | Secondary | Probability calibration quality |

### 6.2 Holdout Performance Results

| Metric | Training Set | Validation Set | Holdout / Test Set |
|---|---|---|---|
| AUC-ROC | 0.851 | 0.828 | 0.831 |
| Gini Coefficient | 0.702 | 0.656 | 0.662 |
| KS Statistic | 0.487 | 0.461 | 0.468 |
| Brier Score | 0.0489 | 0.0521 | 0.0514 |

The holdout AUC of 0.831 represents a 0.003 improvement over the validation set (0.828), indicating no overfitting. The Gini coefficient of 0.662 represents a significant improvement over the vendor scorecard baseline of 0.442 (measured on the same holdout population).

### 6.3 Performance Across Material Subgroups

| Subgroup | N (holdout) | AUC-ROC | Default Rate | Notes |
|---|---|---|---|---|
| Branch channel | 12,864 | 0.847 | 4.9% | Strong performance; branch applicants are typically thicker-file |
| Digital channel | 15,099 | 0.819 | 7.1% | Lower AUC expected; higher proportion of thin-file; meets 0.80 minimum |
| Thick-file (≥ 24 months history) | 22,891 | 0.854 | 5.7% | Strong discrimination |
| Thin-file (< 12 months history) | 5,072 | 0.751 | 9.4% | Known weaker subgroup; above 0.74 objective; documented in limitations |
| Score band < 580 (decline band) | 4,189 | N/A | 19.8% | Default rate confirms appropriate risk stratification of decline band |
| Score band 580–659 (referral band) | 4,608 | N/A | 8.9% | Intermediate risk; appropriate for senior underwriter judgment |
| Score band ≥ 660 (approval band) | 19,166 | N/A | 3.1% | Low default rate confirms approval band quality |

### 6.4 Benchmark Comparisons

| Model | AUC-ROC | Gini | Notes |
|---|---|---|---|
| **Selected model (Random Forest v1.0.0)** | **0.831** | **0.662** | Production model |
| CreditView Analytics vendor scorecard (incumbent) | 0.721 | 0.442 | Prior production system |
| Logistic Regression baseline | 0.801 | 0.602 | Internal benchmark |
| XGBoost | 0.836 | 0.672 | Slightly higher overall; lower thin-file AUC (0.741) |

### 6.5 Sensitivity Analysis

**Top 10 features by mean absolute SHAP value (holdout set):**
1. `revolving_utilization` — 0.0912
2. `derogatory_mark_count_24m` — 0.0847
3. `debt_to_income_ratio` — 0.0789
4. `bureau_score_transunion` — 0.0734
5. `payment_to_income_ratio` — 0.0681
6. `months_since_most_recent_derogatory` — 0.0523
7. `employment_tenure_bin` — 0.0412
8. `thin_file_indicator` — 0.0387
9. `loan_purpose_debt_consolidation` — 0.0341
10. `open_trade_count` — 0.0298

**Key interaction effects:** Revolving utilization > 85% combined with a derogatory mark in the past 12 months produces a compound default signal approximately 1.8× stronger than either feature alone (assessed via SHAP interaction values).

### 6.6 Explainability and Interpretability

SHAP values are computed for every application at scoring time. For declined applications, the top 4 SHAP features with negative impact on the score are mapped to Reg B-compliant adverse action reason codes via a lookup table maintained by the Compliance team. Mapping was validated by the Compliance team against a 200-application sample.

Adverse action reason codes generated by the model are reviewed for Reg B compliance by the Compliance team on a quarterly sample basis. The SHAP-to-reason-code mapping table is treated as a model component; any change requires MRM review.

Example SHAP-to-reason-code mappings:
- `revolving_utilization` (negative) → "Proportion of balances to credit limits is too high on revolving accounts"
- `derogatory_mark_count_24m` (negative) → "Delinquent past or present credit obligations"
- `debt_to_income_ratio` (negative) → "Excessive obligations in relation to income"

---

## Section 7: Assumptions and Limitations

### 7.1 Key Assumptions Registry

| # | Assumption | Sensitivity if Violated | Validation Evidence | Compensating Control |
|---|---|---|---|---|
| 1 | The relationship between credit bureau attributes and 90-day default probability is stable over the economic cycle | High | Out-of-time testing across 2022 cohorts (rate rising environment) shows stable AUC (0.831 holdout vs. 0.828 validation); however, severe recession scenario not tested empirically | Annual stress testing on portfolio; recession scenario performance estimated via macro variable shift sensitivity; model review triggered if actual default rate deviates > 1.5 percentage points from model-implied rate |
| 2 | Credit bureau data (TransUnion) accurately reflects applicant credit history at origination | Medium | 99.1% spot-check accuracy for thick-file applicants; 97.4% for thin-file | TransUnion data quality SLA monitoring; dispute resolution process for borrowers claiming bureau errors |
| 3 | The 24-month outcome window captures materially all default events for the loan products in scope | Medium | Empirical survival analysis of historical portfolio: 91% of eventual defaults occur within 24 months of origination | Accepted as within industry norms for personal installment lending; outcome window not a constraint for loans with 24-month maturities |
| 4 | The digital channel applicant population does not shift materially over time | High | PSI monitoring; digital channel has grown from 31% to 54% of volume, creating distribution shift risk | Quarterly PSI monitoring; channel mix shift > 10 percentage points from training period triggers model review |
| 5 | SHAP-derived adverse action reason codes are materially accurate representations of credit decline drivers | High | Compliance validated 200-application sample: 94% of reason codes assessed as accurate and responsive to credit file | Quarterly Reg B compliance audit by Compliance team; mapping table treated as model component requiring MRM change approval |

### 7.2 Known Limitations

| Limitation | Impact | Status |
|---|---|---|
| Lower performance on thin-file applicants (AUC 0.751) | Higher false positive and false negative rates for a segment that represents 18% of digital volume and has limited credit history data | Accepted — AUC 0.751 exceeds the 0.74 objective and represents significant improvement over vendor scorecard (0.689); thin-file performance monitored quarterly |
| Limited severe recession data in training | Model performance in a prolonged recession with unemployment > 8% has not been empirically validated; stress testing estimates used | Accepted — stress test results documented in model file; macro performance monitoring added to quarterly report |
| 24-month outcome window right-censoring for short-term loans | Loans with 12–18 month maturities may prepay before a default opportunity arises | Accepted — loans with maturities < 18 months represent < 8% of origination volume; documented in model file |
| SHAP reason code mapping introduces minor approximation | SHAP values are post-hoc approximations; the top 4 features may not perfectly represent the model's decision boundary in all cases | Mitigated — Compliance quarterly audit catches material mapping errors; mapping table updated annually |

### 7.3 Compensating Controls for Known Limitations

**Thin-file performance:** Thin-file applicants (score 580–659) are routed to the referral band with a lower approval rate than equivalent thick-file scores, providing an additional human judgment layer for this segment. Thin-file AUC is monitored separately in quarterly reports.

**Recession sensitivity:** Monthly actual default rate is tracked against the model's predicted default rate by origination cohort. A deviation of > 1.5 percentage points for two consecutive months triggers a model sensitivity review with macro scenario analysis.

### 7.4 Conditions Under Which the Model May Fail or Underperform

- **Severe macroeconomic stress:** A rapid rise in unemployment (> 3 percentage point increase in 12 months) not represented in the training data would likely shift the relationship between credit bureau attributes and default probability, reducing model discrimination.
- **Product mix change:** Introduction of a materially new personal loan product (e.g., secured personal loans, BNPL hybrid products) would require re-validation of the model on the new product population.
- **Credit bureau policy change:** Changes in how TransUnion computes or reports bureau attributes (e.g., changes to scoring algorithms affecting raw attributes) could introduce model input drift without a model change.
- **Underwriting policy shift:** A material tightening or loosening of underwriting criteria would change the applicant population and may reduce model performance through distribution shift.

### 7.5 Appropriate Use Boundaries

This model is valid for unsecured personal installment loan origination decisions for consumer applicants at Cascade Consumer Lending, applied to applications received after September 1, 2025. It is not valid for loan renewals, credit limit decisions, collections, commercial lending, or any application outside the $1,000–$50,000 personal loan product range without MRM review.

---

## Section 8: Third-Party and Vendor Components

### 8.1 Third-Party Components Inventory

| Component | Vendor / Source | Version | Type | Purpose |
|---|---|---|---|---|
| TransUnion Credit Bureau Data | TransUnion | API v4.1 | Data | 31 credit bureau features at origination |
| scikit-learn | Open source | 1.5.0 | Library | Random Forest algorithm and pipeline |
| SHAP | Open source | 0.46.0 | Library | Post-hoc feature importance and reason code generation |

### 8.2 Vendor Validation Approach

**TransUnion:** Bureau data quality validated via spot-check of 500 randomly sampled application files against bureau report printouts; 98.9% field-level accuracy. TransUnion provides a data quality guarantee under the Fair Credit Reporting Act. Dispute resolution process documented in Section 12.

### 8.3 Customizations and Adjustments

No customizations to the TransUnion API. All feature engineering described in Section 4.4 is applied to raw bureau outputs by the model pipeline.

### 8.4 Ongoing Vendor Oversight

TransUnion attribute availability and quality monitored monthly via data pipeline completeness report. Any change to TransUnion bureau attribute definitions is communicated via vendor data dictionary update; changes are reviewed by Nathan Park within 10 business days of notification for potential feature impact.

---

## Section 9: GenAI and LLM Governance

N/A — This is a traditional Random Forest model with no generative AI or LLM components.

---

## Section 10: Validation and Monitoring Plan

### 10.1 Pre-Production Validation Requirements

| Validation Activity | Responsible Party | Required Before Production? | Status |
|---|---|---|---|
| Conceptual soundness review | Carlos Reyes | Yes | Complete |
| Data quality independent assessment | Carlos Reyes | Yes | Complete |
| Performance replication on holdout set | Carlos Reyes | Yes | Complete |
| Fair lending / disparate impact assessment | Carlos Reyes | Yes | Complete |
| Adverse action reason code Reg B review | Compliance Team | Yes | Complete |
| SHAP reason code mapping validation | Compliance Team | Yes | Complete |
| Ongoing monitoring plan review | Helen Yu (MRM) | Yes | Complete |

### 10.2 Ongoing Monitoring Metrics and Thresholds

| Metric | Baseline (Development) | Warning Threshold | Action Threshold | Action Required |
|---|---|---|---|---|
| AUC-ROC (quarterly backtesting) | 0.831 | < 0.810 | < 0.790 | MRM notification; model review within 20 business days |
| Gini Coefficient | 0.662 | < 0.620 | < 0.580 | MRM escalation |
| Actual default rate vs. predicted (deviation) | < 0.5 pp deviation | > 1.5 pp deviation | > 2.5 pp deviation | Macro scenario sensitivity review; potential model overlay |
| PSI (score distribution) | ~0.0 | > 0.10 | > 0.25 | Input distribution investigation |
| Thin-file applicant AUC | 0.751 | < 0.730 | < 0.710 | Thin-file subgroup review |
| Disparate impact ratio (approval rate by demographic proxy) | < 1.15x across quintiles | > 1.20x | > 1.25x | ECOA/UDAAP legal review |
| Override rate (underwriter overrides of model recommendation) | ~8% at baseline | > 18% | > 25% | Model quality review; underwriter feedback session |

### 10.3 Monitoring Frequency and Reporting

| Activity | Frequency | Owner | Report Delivered To |
|---|---|---|---|
| Performance monitoring report | Quarterly | Nathan Park | Helen Yu (MRM), Rebecca Santos (Model Owner) |
| Default rate vs. prediction tracking | Monthly | Credit Risk Reporting | Nathan Park, Helen Yu (MRM) |
| Fair lending disparate impact report | Semi-annual | Carlos Reyes (Validator) | Helen Yu (MRM), Compliance, Legal |
| Full model validation | Annual | Carlos Reyes | MRM Committee |

### 10.4 Performance Deterioration Triggers and Escalation

If any Action Threshold is breached: MRM notified within 3 business days. Model review convened within 20 business days. For a significant default rate deviation: macro scenario analysis conducted first to determine whether deterioration is model failure vs. expected macro-driven performance. Possible outcomes: (1) model overlay applied for specific segments; (2) model recalibration; (3) expedited retraining; (4) model retirement and replacement.

### 10.5 Override and Adjustment Policy

Underwriters may override model decline recommendations based on additional information not captured by the model (e.g., business income not reflected in bureau data, documented employment offer). Overrides are logged in LoanPath with a reason code and reviewed monthly by the Credit Risk team. An override rate above 25% triggers a model quality review — high override rates indicate model recommendations are not aligned with underwriter judgment.

Senior underwriter judgment-based approvals in the referral band are not classified as overrides (the referral band is designed to accommodate human judgment).

### 10.6 Redevelopment Criteria

Model redevelopment triggered by: (1) AUC below 0.790 for two consecutive quarterly reports; (2) actual default rate deviation > 2.5 percentage points from model-implied rate for 3 consecutive months; (3) ECOA/fair lending finding requiring structural model changes; (4) material change in Cascade's personal loan product design or underwriting criteria; (5) TransUnion attribute definitions changing in a way that affects > 5 of the 31 bureau features.

---

## Section 11: Implementation and Deployment

### 11.1 Production Architecture

The model is deployed as a Python microservice (FastAPI) called by the LoanPath loan origination system at underwriting time. When an underwriter opens an application for review, LoanPath calls the scoring API, which retrieves the pre-fetched bureau data from a temporary application cache, assembles the 47-feature vector, scores with the Random Forest model, computes SHAP values, maps SHAP to reason codes, and returns the score and recommendation to LoanPath within 200ms. The score and reason codes are displayed to the underwriter in the LoanPath UI.

### 11.2 Input/Output Specifications

| Field | Description | Type | Format / Range | Required? |
|---|---|---|---|---|
| **Input: application_id** | Unique application identifier | string | UUID | Yes |
| **Input: bureau_pull_id** | TransUnion bureau pull reference | string | TransUnion format | Yes |
| **Input: loan_amount_requested** | Requested loan amount | float | 1,000–50,000 | Yes |
| **Input: gross_monthly_income** | Self-reported monthly income | float | > 0 | Yes |
| **Output: probability_of_default** | PD probability | float | 0.0–1.0 | — |
| **Output: scorecard_score** | Scaled score | int | 300–850 | — |
| **Output: recommendation** | Underwriting recommendation | string | `"approve"`, `"refer"`, `"decline"` | — |
| **Output: adverse_action_reason_codes** | Reg B reason codes (top 4) | list[str] | 4 items | — |

### 11.3 Performance and Latency Requirements

| Requirement | Target | Notes |
|---|---|---|
| **Inference latency (p50)** | < 80ms | Includes SHAP computation |
| **Inference latency (p99)** | < 200ms | Underwriter SLA; not a hard real-time constraint |
| **Throughput** | 50 concurrent applications | Peak underwriting volume |
| **Availability SLA** | 99.5% | Underwriters can proceed with manual review if model unavailable |

### 11.4 Integration Dependencies

| Dependency | System | Criticality | Fallback if Unavailable |
|---|---|---|---|
| LoanPath LOS | Internal | High | N/A — model is called by LoanPath |
| TransUnion Bureau Cache | Internal (pre-fetched at application entry) | High | Model cannot score without bureau data; application held for bureau re-pull |

### 11.5 Rollback Plan

Rollback to manual underwriting (without model score) can be executed within 30 minutes by disabling the model service in LoanPath configuration. Manual underwriting using the prior vendor scorecard (CreditView Analytics) is available as an interim fallback for up to 90 days post-implementation while redevelopment is underway. Rollback decision authority rests with the Model Owner in coordination with MRM.

---

## Section 12: Governance

### 12.1 Roles and Responsibilities

| Role | Name / Team | Responsibilities |
|---|---|---|
| **Model Owner** | Rebecca Santos, SVP Consumer Lending | Credit decision policy; approve/decline threshold governance; model change approval |
| **Lead Developer** | Nathan Park, Senior Data Scientist | Model development, documentation, annual retraining |
| **MRM / Validator** | Helen Yu (MRM); Carlos Reyes (Validator) | Independent validation; quarterly performance oversight; fair lending compliance |
| **Business User(s)** | Consumer Loan Underwriters | Use model score as primary credit risk input; final approve/decline decision authority |
| **Data Owner** | Credit Risk Reporting (loan performance); TransUnion (bureau) | Data quality, bureau pull management |
| **IT / MLOps** | Technology — Lending Systems | LoanPath integration, model deployment, bureau data pipeline |
| **Compliance** | Compliance Team | Reg B adverse action notice review; fair lending oversight |
| **Internal Audit** | Internal Audit — Credit Risk | Annual credit model risk management effectiveness review |

### 12.2 Model Inventory Registration

| Field | Value |
|---|---|
| **Model inventory system** | Cascade ModelTrack |
| **Model ID in inventory** | MDL-2025-031 |
| **Inventory registration date** | 2025-07-15 |
| **Tier in model inventory** | Tier 1 — High Materiality |

### 12.3 Change Management

Threshold changes (approve/decline/refer cutoffs) require MRM review and Compliance notification (ECOA impact assessment). Feature additions or algorithm changes require full MRM review. Annual retraining on new data requires MRM notification and performance sign-off. All changes logged in Appendix C.

### 12.4 Document Retention

This MDD and all supporting artifacts shall be retained for a minimum of 7 years following model retirement per Cascade's Model Risk Policy and Reg B record retention requirements.

---

## Appendix A: Glossary

| Term | Definition |
|---|---|
| **AUC-ROC** | Area Under the Receiver Operating Characteristic Curve |
| **ECOA** | Equal Credit Opportunity Act — prohibits credit discrimination on protected class basis |
| **Gini Coefficient** | 2 × AUC − 1; credit risk discrimination measure |
| **KS Statistic** | Kolmogorov-Smirnov statistic; maximum separation between default and non-default score distributions |
| **Probability of Default (PD)** | Estimated probability that a borrower will default within a specified time window |
| **Reg B** | Regulation B — implements ECOA; requires adverse action notices with reason codes for credit denials |
| **SHAP** | SHapley Additive exPlanations — method for computing feature contributions to model predictions |
| **SR 26-2** | Supervisory Guidance on Model Risk Management (April 17, 2026) |
| **Thin-file** | Applicant with fewer than 12 months of credit history; limited bureau data available |

---

## Appendix B: References and Source Documents

| Reference | Description | Location |
|---|---|---|
| SR 26-2 | Supervisory Guidance on Model Risk Management (April 17, 2026) | `_mrm-mdd-kit/governance/sr26-2.md` |
| Validation Report — MDL-2025-031 | Independent validation report by Carlos Reyes | Cascade ModelTrack document repository |
| Fair Lending Assessment | Pre-production disparate impact analysis | Cascade ModelTrack document repository |
| SHAP Reason Code Mapping Table | Compliance-approved SHAP-to-Reg B reason code mapping | `src/compliance/reason_code_mapping.json` in model repository |

---

## Appendix C: Model Change Log

| Version | Date | Author | Type of Change | MRM Approval Required? | Summary |
|---|---|---|---|---|---|
| 1.0.0 | 2025-09-01 | Nathan Park | Initial production release | N/A | Replaced CreditView Analytics vendor scorecard |

---

## Appendix D: Experiment Log

| Experiment ID | Date | Hypothesis | Approach | Result | Decision |
|---|---|---|---|---|---|
| EXP-2025-001 | 2025-02-20 | Logistic regression sufficient and more interpretable | LR trained on full feature set | AUC 0.801 overall; thin-file AUC 0.734 — below 0.74 target | Rejected; insufficient thin-file performance |
| EXP-2025-002 | 2025-03-10 | XGBoost outperforms Random Forest | XGBoost trained and evaluated | Overall AUC 0.836; thin-file AUC 0.741 — below Random Forest 0.751 | Rejected; Random Forest preferred for thin-file performance despite slightly lower overall AUC |
| EXP-2025-003 | 2025-03-25 | Neural network (MLP) captures interactions better | MLP (3 layers, 128/64/32 units) | AUC 0.829; thin-file AUC 0.748; lower interpretability; insufficient gain to justify complexity | Rejected |
| EXP-2025-004 | 2025-04-08 | Isotonic calibration needed for probability score | Calibration reliability diagram assessment | Brier score 0.0521 without calibration — within acceptable bounds | Calibration not applied; Random Forest probabilities used directly |
| EXP-2025-005 | 2025-05-01 | Decline threshold at 580 achieves target default rate in decline band | Threshold sweep 540–620 | Threshold 580 → 19.8% default rate in decline band, 3.1% in approval band | Adopted |
