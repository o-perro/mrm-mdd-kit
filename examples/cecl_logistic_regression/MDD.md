# Model Development Document (MDD)

> **Kit version:** mrm-mdd-kit v1.0.0 | **Regulatory basis:** SR 26-2 (April 17, 2026)
>
> **FICTIONAL EXAMPLE** — All institution names, personnel, performance metrics, and business details are completely made up. This document exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## Document Control

| Field | Value |
|---|---|
| **Model Name** | Consumer Portfolio CECL Expected Credit Loss Model |
| **Model ID** | MDL-2024-003 |
| **Model Type** | Logistic Regression — Probability of Default / CECL |
| **Model Version** | 2.0.0 |
| **Document Version** | 2.1 |
| **Document Status** | Approved |
| **Development Start Date** | 2024-01-15 |
| **Document Date** | 2024-09-30 |
| **Effective Date** | 2024-10-01 (Q4 2024 reporting) |
| **Next Review Date** | 2025-10-01 |
| **Model Owner** | Christine Hoang, CFO |
| **Lead Developer** | Robert Adeyemi, Senior Credit Risk Analyst |
| **Contributing Developers** | N/A |
| **Primary Validator** | Alan Marsh, Baker & Monroe LLP (External Validator) |
| **MRM Contact** | Patricia Kim, VP Internal Audit & Model Risk |

### Approval Sign-offs

| Role | Name | Signature | Date |
|---|---|---|---|
| Lead Developer | Robert Adeyemi | *(signed)* | 2024-09-30 |
| Model Owner / CFO | Christine Hoang | *(signed)* | 2024-10-02 |
| External Validator | Alan Marsh, Baker & Monroe LLP | *(signed)* | 2024-10-07 |
| Audit Committee Chair (Board) | Thomas Bergmann | *(signed)* | 2024-10-11 |

### Document Version History

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| 1.0 | 2022-09-01 | Robert Adeyemi | Initial CECL model for Q3 2022 CECL adoption |
| 1.1 | 2023-04-15 | Robert Adeyemi | Annual redevelopment: updated economic scenario weights |
| 2.0 | 2024-09-30 | Robert Adeyemi | Major redevelopment: retrained on 2020–2023 data, updated macro variables, added HELOC sub-model |
| 2.1 | 2024-09-30 | Robert Adeyemi | Final: incorporated external validator pre-review feedback |

---

## Section 1: Executive Summary

### 1.1 Model Overview

The Consumer Portfolio CECL Expected Credit Loss Model is a logistic regression-based model that estimates lifetime probability of default (PD) and 12-month PD for each consumer loan in Lakeshore Community Bank's portfolio at each quarter-end reporting date. The model produces loan-level PD estimates under three macroeconomic scenarios (base case, adverse, severely adverse), which are probability-weighted to produce the final expected credit loss (ECL) estimate. The ECL estimate feeds directly into Lakeshore's allowance for credit losses (ACL) reported in its quarterly and annual financial statements and regulatory filings (Call Report). This model is Lakeshore's primary compliance model for FASB ASC Topic 326 (CECL).

### 1.2 Model Risk Rating

| Dimension | Rating | Rationale |
|---|---|---|
| **Inherent Risk** | Medium | Logistic regression is deliberately chosen for interpretability; 18 features; key assumptions include macroeconomic scenario accuracy and historical relationship stability; reasonable complexity for a CECL model at a community bank scale |
| **Model Exposure** | High | Model output determines the ACL reported in audited financial statements and regulatory capital filings; $1.9B consumer portfolio; ACL changes directly affect reported earnings and regulatory capital ratios |
| **Model Purpose** | High | Primary regulatory capital reporting model; audited by external auditor; examined by FDIC and Federal Reserve; FASB ASC 326 compliance |
| **Overall Materiality** | High | High exposure and high purpose; financial reporting and regulatory capital implications drive High materiality regardless of medium inherent risk |

### 1.3 Key Risks and Findings

- **Macroeconomic scenario dependency:** The model ACL estimate changes significantly with each quarterly scenario update. A shift from base case to adverse scenario increases ACL by approximately 34%. The model's output is more sensitive to scenario assumptions than to model parameters. Full treatment in Sections 6.5 and 7.1.
- **Limited severe recession data:** The training data (2015–2023) includes the COVID-2020 stress period but not a prolonged severe recession with sustained unemployment > 9%. Performance in a severe recession scenario is extrapolated from the model's macroeconomic coefficients rather than empirically validated. Full treatment in Section 7.2.
- **Logistic regression linearity assumption:** The model assumes a linear relationship between log-odds of default and feature values. Non-linear interactions (e.g., the combined effect of high DTI and elevated unemployment) may not be fully captured. This is accepted as a deliberate trade-off for interpretability and auditability. Full treatment in Section 7.1.
- **External validator dependency:** Lakeshore uses an external validator (Baker & Monroe LLP) rather than a dedicated internal MRM team. External validation introduces coordination overhead and potential knowledge continuity risk. Full treatment in Section 12.3.

### 1.4 Regulatory Applicability

| Regulation | Applicable? | Notes |
|---|---|---|
| SR 26-2 — Model Risk Management | Yes — by spirit of guidance | Lakeshore's $4.8B total assets is below SR 26-2's $30B primary applicability threshold; however, the model's regulatory reporting purpose and ACL materiality make SR 26-2 principles applicable as sound practice |
| ECOA / Reg B — Fair Lending | No | CECL is a portfolio-level reserving model, not a credit decision model |
| CECL / FASB ASC 326 | Yes | Primary compliance requirement; model is the institution's primary CECL estimation methodology |
| BSA / AML | No | Not applicable |
| CFPB UDAAP | No | Not a consumer-facing decision model |
| Call Report (FFIEC) | Yes | ACL estimates from this model are reported in the quarterly Call Report; FDIC and Federal Reserve examiner scrutiny |

---

## Section 2: Model Purpose and Business Context

### 2.1 Business Problem and Objectives

FASB ASC Topic 326 requires Lakeshore to estimate lifetime expected credit losses on its consumer loan portfolio and maintain an allowance for credit losses (ACL) sufficient to cover those expected losses. This replaced the prior incurred loss model (ALLL) which recognized losses only when they were probable and estimable — a fundamentally backward-looking approach. CECL requires a forward-looking estimate that incorporates reasonable and supportable macroeconomic forecasts.

Lakeshore adopted CECL for fiscal year 2023 reporting. The v1.0 model used for initial adoption was a simplified cohort-based historical loss rate approach supplemented with qualitative adjustments. While regulatory-compliant for adoption, the v1.0 model lacked the loan-level segmentation and macroeconomic sensitivity required for robust ACL estimation as the portfolio grew. The v2.0 redevelopment objective: build a loan-level PD model with explicit macroeconomic scenario integration, fully aligned with ASC 326 guidance and sufficient for external auditor and FDIC examiner scrutiny.

### 2.2 Intended Use

The model runs quarterly (at each quarter-end) as a batch process. For each active consumer loan, the model produces: (1) 12-month PD under each scenario; (2) lifetime PD under each scenario; (3) loss given default (LGD) estimate (separately modeled; referenced but not re-documented here); (4) exposure at default (EAD). The product of PD × LGD × EAD under each scenario, probability-weighted (base 50%, adverse 30%, severe adverse 20%), produces the loan-level ECL estimate. Loan-level ECLs are aggregated to the portfolio level for ACL reporting. The CFO and Controller review the aggregate ACL estimate before financial statement disclosure; external auditor (Reinhardt Audit Group) reviews the methodology and results annually.

### 2.3 Intended Users

| User / System | How Output Is Used |
|---|---|
| Christine Hoang, CFO | Reviews ACL estimate; approves for financial statement disclosure |
| Finance / Accounting Team | Inputs ACL estimate into general ledger and financial statements |
| External Auditor (Reinhardt Audit Group) | Reviews model methodology and results as part of annual audit |
| FDIC / Federal Reserve Examiners | Review model documentation and results during examination |
| Board Audit Committee | Receives quarterly ACL update; reviews model governance |
| Patricia Kim, MRM | Reviews annual model validation report |

### 2.4 Out-of-Scope and Prohibited Uses

- This model estimates expected credit losses for the consumer installment and HELOC portfolio only. It must not be applied to commercial loans, SBA loans, or mortgage products without separate validation.
- The model produces a statistical estimate of expected losses; it must not be used as the sole basis for individual credit decisions, collections strategies, or loss forecasting outside the CECL reporting context.
- Macroeconomic scenario assumptions used in the model must be reviewed and approved by the CFO and MRM each quarter before model execution; stale scenario inputs must not be used.
- The model must not be modified (feature additions, parameter changes) between quarterly runs without MRM notification and documentation.

### 2.5 Downstream Decisions and Impact

The model's ACL estimate directly determines the quarterly provision for credit losses — the income statement charge that funds the ACL reserve. A 10 basis point change in the ACL ratio on the $1.9B consumer portfolio equals approximately $1.9 million in provision expense. In Q4 2024, the model-derived ACL is approximately $47.3 million (2.49% ACL ratio on the consumer portfolio). ACL changes affect reported net income, retained earnings, and regulatory capital ratios (CET1). External auditor and FDIC examiner scrutiny of the ACL methodology and adequacy is a primary model governance driver.

### 2.6 Model Dependencies

| Dependency | Type | Description |
|---|---|---|
| Macroeconomic Scenario Forecasts (Moody's Analytics) | Upstream | Base case, adverse, severely adverse economic forecasts (unemployment rate, GDP growth, HPI) sourced quarterly from Moody's Analytics |
| Core Banking System (Loan Module) | Upstream | Loan-level data: balance, interest rate, origination date, remaining term, product type |
| Credit Bureau Refresh (TransUnion) | Upstream | Quarterly credit bureau attribute refresh for active loans |
| LGD Model | Upstream | Separately developed loss given default estimates; referenced but not re-documented in this MDD |
| General Ledger / ACL Account | Downstream | Receives final ECL estimate for financial statement entry |

---

## Section 3: Model Risk Assessment

### 3.1 Inherent Risk Assessment

| Risk Factor | Assessment | Rationale |
|---|---|---|
| **Model complexity** | Low | Logistic regression; 18 features; transparent coefficients; fully auditable methodology — deliberately chosen for interpretability |
| **Number and criticality of assumptions** | High | Key assumptions: (1) log-linear relationship between features and default probability; (2) macroeconomic coefficients estimated from 2015–2023 data extrapolate to forecast scenarios; (3) LGD and prepayment assumptions (separately modeled); (4) macroeconomic scenario weights (50%/30%/20%) reflect reasonable and supportable forecasts |
| **Input data quality** | Medium | Internal loan data is high quality; quarterly bureau refresh may have 1–2 week lag; Moody's macro scenarios are externally sourced and subject to periodic revision |
| **Data constraints / availability** | Medium | 9-year training window (2015–2023) includes only one significant stress period (COVID-2020); limited data from sustained severe recession |
| **Model interpretability** | Low | Logistic regression coefficients are fully interpretable; external auditor and examiner can independently verify coefficient direction and magnitude |
| **Novelty of approach** | Low | Logistic regression PD model is a well-established, widely accepted CECL methodology; consistent with peer community bank approaches |
| **Overall Inherent Risk** | Medium | Low complexity and full interpretability partially offset by critical macroeconomic assumptions and limited severe recession data |

### 3.2 Model Exposure Assessment

| Exposure Factor | Value | Notes |
|---|---|---|
| **Portfolio / business scope** | $1.9B consumer loan portfolio; ~48,200 active loans | All consumer installment and HELOC accounts |
| **Frequency of model use** | Quarterly (at each quarter-end) | Batch run for each quarterly financial reporting cycle |
| **Business impact of incorrect output** | High | ACL under-estimation → regulatory capital deficiency, audit finding, restatement risk; ACL over-estimation → overstated provision, reduced earnings |
| **Degree of automation** | Human-reviewed | CFO reviews ACL estimate; external auditor reviews annually; qualitative overlay applied if needed |
| **Overall Exposure** | High | Financial statement and regulatory capital direct impact |

### 3.3 Model Purpose Assessment

| Purpose Factor | Assessment | Notes |
|---|---|---|
| **Regulatory-facing use** | Yes | FASB ASC 326 compliance; FDIC/Federal Reserve Call Report; external audit |
| **Financial risk management use** | Yes | ACL directly affects regulatory capital ratios |
| **Consumer-impacting decisions** | No | ACL is a portfolio-level reserve estimate; does not affect individual consumer decisions |
| **Overall Purpose Risk** | High | Dual regulatory and financial reporting use |

### 3.4 Overall Materiality Determination

| | |
|---|---|
| **Overall Materiality** | High |
| **Validation rigor required** | Full independent external validation (Baker & Monroe LLP) annually; macroeconomic scenario review quarterly; CFO sign-off on ACL estimate each quarter |
| **Monitoring frequency** | Quarterly |
| **Rationale** | High exposure (financial reporting, regulatory capital) and High purpose (FASB compliance, external audit) drive High materiality. Logistic regression's low inherent risk is offset by the critical importance of the macroeconomic assumptions. Lakeshore applies SR 26-2 principles as sound governance practice despite being below the $30B threshold, given the model's regulatory reporting purpose. |

---

## Section 4: Data

### 4.1 Data Sources and Provenance

| Data Source | System of Record | Data Owner | Data Type | Refresh Frequency |
|---|---|---|---|---|
| Loan Performance History | Core Banking (Jack Henry Silverlake) | Finance / Accounting | Monthly loan-level delinquency and charge-off records, 2015–2023 | Monthly historical extract for training; quarterly for production |
| Credit Bureau Attributes at Origination | TransUnion (historical) | Credit Risk | Credit bureau attributes at loan origination date | Historical extract |
| Macroeconomic Variables (historical) | Federal Reserve FRED database | Credit Risk | Unemployment rate, GDP growth rate, Wisconsin HPI: quarterly, 2015–2023 | Public data; downloaded quarterly |
| Macroeconomic Scenario Forecasts | Moody's Analytics | Finance | Base, adverse, severely adverse forecasts for unemployment, GDP, HPI | Quarterly subscription; updated before each ACL run |

### 4.2 Observation Period and Sample Design

| | |
|---|---|
| **Observation period** | January 1, 2015 – December 31, 2023 (9 years; 36 quarters) |
| **Sample size** | 94,847 loan-quarter observations (distinct loans: 31,284) |
| **Geographic scope** | Wisconsin and northern Illinois — Lakeshore's full operating footprint |
| **Product scope** | Consumer installment loans (personal, auto, boat) and home equity lines of credit (HELOC); separate sub-models for installment and HELOC |
| **Exclusion criteria** | Loans originated and paid off in fewer than 6 months excluded (early payoff reduces outcome observation window). Loans transferred to workout/collections prior to 90-day delinquency excluded from the PD model (separately managed). |
| **Rationale for observation period** | 9-year window captures: the pre-COVID baseline (2015–2019), the COVID-2020 stress period (only modern stress period available), and the post-COVID normalization and rate rise (2021–2023). Starts at 2015 to align with Jack Henry Silverlake implementation date (consistent loan data). |

### 4.3 Data Quality Assessment

| Quality Dimension | Finding | Action Taken |
|---|---|---|
| **Completeness** | Loan performance data: 99.6% complete. Credit bureau attributes at origination: 97.8% complete (2.2% of older loans missing origination bureau data). Macroeconomic variables: 100% complete (public FRED data). | Loans with missing origination bureau data imputed using portfolio-segment median values at origination quarter |
| **Accuracy** | Delinquency records spot-checked against servicing system on 500 loan-quarter sample: 99.4% accuracy. COVID forbearance period (March–September 2020): delinquency data suppressed due to CARES Act deferral; loans in forbearance excluded from the COVID stress quarter observations to avoid label contamination. | COVID forbearance exclusion applied; 3,847 forbearance loan-quarter observations removed |
| **Timeliness** | Historical data is static training data; timeliness not a concern for model training. Production: quarterly bureau refresh may have 1–2 week lag from quarter-end; macro scenarios received within 5 business days of quarter-end. | Production run scheduled for 15th business day of each month following quarter-end to ensure data freshness |
| **Consistency** | Loan product classification changed in 2019 (auto and boat loans consolidated into "consumer installment" category); pre-2019 data recoded to new taxonomy | Recoding applied; verified on 1,000-record sample |
| **Representativeness** | Annual charge-off rate in training data: 1.47% (personal installment), 0.34% (HELOC). Rates elevated in 2020 (2.89% personal installment) and suppressed in 2021–2022 (CARES Act effects). Training data includes two distinct economic periods. | Accepted; COVID period included as a stress calibration point despite CARES Act distortions |

### 4.4 Feature Engineering and Preprocessing

| Feature / Transformation | Description | Rationale |
|---|---|---|
| Credit score at origination (binned) | TransUnion score at origination, binned into 6 bands | Preserves origination credit quality signal; binning handles non-linearity |
| Debt-to-income ratio at origination | Originated DTI from application data | Core underwriting metric; historical predictor of default |
| Loan-to-value ratio (HELOC only) | Current balance / property value estimate | HELOC-specific collateral adequacy measure |
| Months on books | Months since loan origination | Default hazard peaks in years 2–4; captures loan seasoning effect |
| Delinquency status at start of period | 30-day delinquency flag at start of each quarter | Prior delinquency is the strongest predictor of subsequent 90-day default |
| Unemployment rate (quarterly, macro) | Wisconsin state unemployment rate from FRED | Macroeconomic sensitivity variable; primary macro driver |
| GDP growth rate (quarterly, macro) | U.S. real GDP growth rate, year-over-year from FRED | Second macroeconomic sensitivity variable |
| Interest rate environment (quarterly, macro) | 10-year Treasury yield; proxy for rate environment | Rate environment affects HELOC performance |

### 4.5 Train / Validation / Test Split

| Split | Size | Time Period | Notes |
|---|---|---|---|
| **Training set** | 66,393 (70%) | Q1 2015 – Q4 2020 | Captures baseline and COVID stress; panel structure with multiple observations per loan |
| **Validation set** | 14,227 (15%) | Q1 2021 – Q2 2022 | Post-COVID normalization; used for coefficient evaluation and model selection |
| **Test / holdout set** | 14,227 (15%) | Q3 2022 – Q4 2023 | Rate-rising environment; used only for final evaluation |

The training and validation sets include the COVID stress period, providing calibration against a genuine stress environment. The holdout set covers the rate-rising environment of 2022–2023 — an out-of-distribution period relative to the low-rate training environment — providing a meaningful stress test of model generalization.

### 4.6 Data Assumptions and Limitations

The model's macroeconomic coefficients are estimated from historical data that includes only one significant recession (COVID-2020, with significant government intervention). The model's response to a severe unassisted recession (e.g., sustained unemployment > 9% with limited fiscal stimulus) is extrapolated from these coefficients rather than empirically validated. This is the single most important assumption in the model and is explicitly addressed in the severely adverse scenario.

The COVID forbearance exclusion (March–September 2020) means the model was not trained on the mechanics of loan behavior during payment deferral periods. A future scenario with widespread forbearance would require qualitative overlay adjustments.

---

## Section 5: Model Development

### 5.1 Problem Framing and Conceptual Approach

CECL lifetime expected loss estimation is framed as a probability of default (PD) problem at the loan-quarter level: given the current loan attributes and macroeconomic conditions, what is the probability that this loan defaults (90+ days delinquent) within the next 12 months (12-month PD) and over its remaining life (lifetime PD)?

The conceptual foundation is a proportional hazard framework: the likelihood of default at any point in a loan's life is a function of loan-level credit risk attributes (established at origination and updated quarterly) modulated by the prevailing macroeconomic environment. Logistic regression captures this additive relationship transparently — each coefficient represents the marginal change in log-odds of default associated with one unit change in the predictor.

Logistic regression was chosen deliberately over more powerful algorithms (gradient boosting, neural network) because: (1) external auditors and regulators can independently verify and challenge individual coefficients; (2) coefficient signs must be theoretically consistent (negative coefficient on credit score, positive on unemployment rate); and (3) the linear additive structure integrates naturally with scenario-based forecasting.

### 5.2 Methodology Selection

**Selected approach:** Logistic regression with 18 features, estimated via maximum likelihood on the pooled loan-quarter panel. Separate sub-models for personal installment loans and HELOCs (materially different default drivers and loss characteristics).

**Rationale:** Logistic regression is the appropriate methodology for a regulatory-reporting CECL model at a community bank where interpretability, coefficient defensibility, and external auditor review are paramount. More powerful ML approaches would improve statistical fit at the cost of interpretability that is essential for regulatory scrutiny.

**Alternatives considered:**

| Alternative Approach | Reason Not Selected |
|---|---|
| Historical cohort loss rate (v1.0 approach) | Does not meet the ASC 326 requirement for loan-level segmentation and forward-looking macroeconomic integration |
| XGBoost / gradient boosting | Higher predictive power but coefficients not directly interpretable; external auditor and FDIC examiner cannot independently challenge individual feature contributions |
| Survival analysis (Cox proportional hazard) | Theoretically superior for lifetime PD but significantly more complex to implement, validate, and explain; community bank scale does not justify the additional complexity |
| Discounted cash flow / vintage curve approach | Considered for HELOC sub-model; rejected in favor of logistic regression for consistency of methodology across sub-models |

### 5.3 Model Architecture and Specifications

| Specification | Value |
|---|---|
| **Algorithm / framework** | Logistic regression (statsmodels 0.14.0); Python; separate sub-models for installment and HELOC |
| **Number of features (inputs)** | 18 features (12 loan-level, 6 macroeconomic / time-based) |
| **Output type** | Quarterly PD (0.0–1.0) per loan per scenario; aggregated to lifetime PD via survival product |
| **Output range** | 0.0–1.0 per period |
| **Decision threshold(s)** | No decision threshold — continuous PD used for ECL computation |
| **Model size / complexity** | 18-coefficient logistic regression; trivial computational footprint |
| **Inference latency** | Full portfolio run (48,200 loans × 3 scenarios): < 4 minutes |

### 5.4 Training Procedure

- **Estimation method:** Maximum likelihood estimation (MLE) on pooled quarterly panel data
- **Standard errors:** Clustered standard errors by loan ID to account for within-loan correlation across quarterly observations
- **Goodness-of-fit:** Hosmer-Lemeshow test (p = 0.412 — no evidence of poor fit); McFadden pseudo-R² = 0.214
- **Coefficient sign review:** All 18 coefficients reviewed for theoretical consistency; all signs are in the expected direction (higher credit score → lower PD; higher unemployment → higher PD; etc.)
- **Separate sub-models:** Installment and HELOC sub-models estimated independently; F-test for pooled vs. separate models confirms statistical justification for separation (p < 0.001)

### 5.5 Hyperparameter Selection and Calibration

Logistic regression has no hyperparameters requiring tuning via search. The primary methodological choices:

| Choice | Decision | Rationale |
|---|---|---|
| Regularization | None (MLE) | Regularization not appropriate for coefficient interpretability in a regulatory reporting model; sample size (94,847 obs.) is sufficient for stable estimation |
| Macro scenario weights | 50% base / 30% adverse / 20% severely adverse | Consistent with Moody's Analytics recommended weights for Lakeshore's risk profile; reviewed and approved by CFO quarterly |
| Lifetime PD computation | Monthly PD chain product: (1 - PD₁) × (1 - PD₂) × ... × (1 - PDₙ) | Standard survival approach; each period's PD conditioned on no prior default |
| Prepayment assumption | Separately modeled 12-month CPR applied to reduce EAD | Prepayment reduces lifetime exposure; separately documented |

### 5.6 Developmental Testing

- **Coefficient stability:** Jackknife resampling (removing one year of data at a time) showed coefficient stability across all 9 annual subsets; maximum coefficient variation < 15% for any variable — within acceptable bounds for a 9-year panel
- **Validation set (Q1 2021 – Q2 2022) AUC:** 0.784 — strong discrimination for a logistic regression CECL model
- **Scenario sensitivity analysis:** ACL estimate changes by approximately $15.9M (34%) between base case and severe downside scenario — confirming meaningful macroeconomic sensitivity
- **Back-testing against actual losses:** Model-implied loss rate for 2022–2023 holdout period (1.62% annualized) vs. actual charge-off rate (1.71%); within 9 basis points — indicates reasonable calibration

---

## Section 6: Model Performance

### 6.1 Evaluation Metrics and Rationale

| Metric | Type | Rationale |
|---|---|---|
| AUC-ROC | Primary | Discrimination ability; assesses whether model ranks defaults higher than non-defaults |
| Hosmer-Lemeshow Test | Primary | Calibration test; assesses whether predicted probabilities match observed default rates by decile |
| Back-test: predicted vs. actual loss rate | Primary | CECL-specific validation; compares model-implied loss rate to observed charge-off rate over holdout period |
| Coefficient sign and significance review | Primary | Regulatory requirement: all coefficients must be theoretically consistent and statistically significant |
| Scenario ACL sensitivity | Secondary | Validates model's macroeconomic scenario sensitivity is directionally reasonable |

### 6.2 Holdout Performance Results

| Metric | Training Set | Validation Set | Holdout / Test Set |
|---|---|---|---|
| AUC-ROC | 0.801 | 0.784 | 0.779 |
| Hosmer-Lemeshow p-value | 0.412 | 0.327 | 0.298 |
| Predicted 12-month loss rate | 1.61% | 1.58% | 1.62% |
| Actual charge-off rate | 1.54% | 1.49% | 1.71% |
| Predicted vs. actual difference | +0.07 pp | +0.09 pp | -0.09 pp | 

The holdout AUC of 0.779 is within 0.005 of the validation AUC (0.784), indicating no overfitting. The Hosmer-Lemeshow test p-value of 0.298 indicates no evidence of poor calibration at the 5% significance level. The back-test predicted loss rate of 1.62% vs. actual 1.71% represents a 9 basis point underestimation on the holdout period — acceptable given the rate-rising environment of 2022–2023 not well-represented in training data.

### 6.3 Performance Across Material Subgroups

| Subgroup | N (holdout obs.) | AUC-ROC | Predicted Annualized Loss Rate | Actual Charge-Off Rate |
|---|---|---|---|---|
| Personal installment sub-model | 9,847 | 0.791 | 1.87% | 1.94% |
| HELOC sub-model | 4,380 | 0.758 | 0.38% | 0.44% |
| Credit score 660+ at origination | 9,847 | 0.761 | 0.94% | 0.98% |
| Credit score < 660 at origination | 4,380 | 0.804 | 3.41% | 3.67% |
| Loans originated 2019–2021 (vintage risk) | 6,814 | 0.782 | 1.74% | 1.89% |
| Loans originated 2022+ (higher rate environment) | 3,412 | 0.773 | 1.41% | 1.44% |

### 6.4 Benchmark Comparisons

| Model | AUC-ROC | Notes |
|---|---|---|
| **Selected model (Logistic Regression v2.0)** | **0.779** | Production model |
| v1.0 cohort loss rate approach | ~0.650 (estimated) | Prior model; did not produce loan-level scores; estimated AUC from cohort-level comparison |
| XGBoost benchmark | 0.831 | Higher AUC; not selected due to interpretability requirements |

### 6.5 Sensitivity Analysis

**Macroeconomic scenario sensitivity (Q4 2024 ACL):**

| Scenario | Weight | 12-Month PD (weighted avg.) | Lifetime PD (weighted avg.) | Scenario ACL |
|---|---|---|---|---|
| Base case | 50% | 1.62% | 4.81% | $38.9M |
| Adverse | 30% | 2.34% | 6.93% | $56.1M |
| Severely adverse | 20% | 3.87% | 11.24% | $91.4M |
| **Probability-weighted (final ACL)** | 100% | **2.14%** | **6.31%** | **$47.3M** |

A 10 percentage point shift in the severe downside scenario weight (from 20% to 30%) would increase the ACL by approximately $4.4M, a 9.3% increase. This sensitivity analysis is presented to the Audit Committee quarterly.

**Coefficient sensitivity:** A 1 percentage point increase in the unemployment rate variable is associated with an 8.4% increase in predicted quarterly PD (exp(coefficient) = 1.084), consistent with the empirical relationship observed in training data.

### 6.6 Explainability and Interpretability

The logistic regression model provides complete transparency: all 18 coefficients, their standard errors, and marginal effects are published in Appendix D of this MDD and reviewed by the external validator annually. The coefficient table is presented to the Audit Committee as part of the annual model review.

For any loan with a model-predicted lifetime PD significantly different from the portfolio average, the contributing factors can be fully decomposed using the linear predictor — credit score, DTI, delinquency status, and macroeconomic variables each contribute an additive increment to log-odds of default.

---

## Section 7: Assumptions and Limitations

### 7.1 Key Assumptions Registry

| # | Assumption | Sensitivity if Violated | Validation Evidence | Compensating Control |
|---|---|---|---|---|
| 1 | The log-linear relationship between predictors and default probability is an adequate representation of the true relationship | Medium | Hosmer-Lemeshow test confirms calibration (p=0.298); residual analysis shows no systematic pattern | Accepted; deliberate trade-off for interpretability; qualitative overlay available for severe non-linearity |
| 2 | Macroeconomic coefficients estimated from 2015–2023 data extrapolate to the forecast scenarios | High | Holdout period (2022–2023 rate-rising) predicted loss rate 1.62% vs. actual 1.71% (+9 bps) — reasonable extrapolation to a mild out-of-distribution period | Macroeconomic scenario weights reviewed and approved by CFO quarterly; qualitative overlay applied if model-implied ACL appears unreasonable relative to management judgment |
| 3 | Moody's Analytics scenario forecasts represent "reasonable and supportable" forecasts as required by ASC 326 | High | CFO and external auditor review Moody's forecast methodology annually as part of model governance | Moody's Analytics engagement includes annual methodology documentation review; alternative scenario provider considered as backup |
| 4 | The 9-year training period (2015–2023) with one moderate stress event (COVID-2020) is sufficient for macroeconomic coefficient calibration | High | COVID back-test shows reasonable performance; severe recession not empirically testable | Severe downside scenario stress test compared against peer bank CECL estimates and regulatory stress scenarios annually |
| 5 | COVID forbearance exclusion does not bias macroeconomic coefficients | Medium | Sensitivity analysis with and without forbearance period shows < 5% coefficient change | Accepted; qualitative overlay documented for future forbearance scenarios |

### 7.2 Known Limitations

| Limitation | Impact | Status |
|---|---|---|
| Limited severe recession data: only one mild-to-moderate stress period in training data | Severely adverse scenario ACL may be understated in a sustained deep recession | Accepted — inherent limitation of community bank CECL modeling; qualitative overlay mechanism available; peer benchmarking used as sanity check |
| Linearity assumption may understate combined effects (e.g., high DTI + high unemployment interaction) | ACL may be slightly understated in combined stress scenarios | Accepted — deliberate trade-off for interpretability; external auditor has reviewed and accepted |
| Quarterly macro scenario updates create ACL volatility | ACL can change materially quarter-over-quarter due to scenario changes rather than actual portfolio change | Accepted — inherent in CECL forward-looking methodology; volatility explained in CFO earnings commentary |
| Back-test underestimation of 9 bps in holdout period | Model slightly under-predicted actual losses in the rate-rising environment of 2022–2023 | Accepted — within 10 bps; a qualitative adjustment (+5 bps) was applied by CFO for Q4 2024 as a conservative overlay |

### 7.3 Compensating Controls for Known Limitations

**Severe recession limitation:** The CFO applies a qualitative overlay adjustment to the model ACL estimate if the model-implied severe downside scenario appears inconsistent with peer bank CECL estimates or regulatory stress scenarios. The Q4 2024 qualitative overlay of +5 bps ($950K) was applied for conservatism given the rate-rising environment holdout underestimation. Overlay amounts and rationale are documented and reviewed by the external auditor.

**Macro scenario volatility:** A quarterly ACL bridge analysis (model vs. prior quarter; volume vs. credit quality vs. scenario change) is prepared by the Finance team and reviewed by the Audit Committee. This bridges the total ACL change into its components, making scenario-driven volatility transparent.

### 7.4 Conditions Under Which the Model May Fail or Underperform

- **Sustained severe recession:** Unemployment significantly above the range in training data (training max: 14.4% COVID peak; sustained severe recession scenario extrapolates significantly beyond this) would require management judgment overlay.
- **New product introduction:** The model covers personal installment and HELOC products only. New consumer products (e.g., credit card, personal secured lending) would require separate model development.
- **Regulatory/FASB guidance change:** ASC 326 implementation guidance continues to evolve; a material change to the standard (e.g., modified retrospective transition requirements, new segmentation requirements) could require model redesign.
- **Major credit policy change:** A significant loosening or tightening of underwriting criteria would shift the origination credit quality distribution; the model would require recalibration if the origination score distribution shifts meaningfully.

### 7.5 Appropriate Use Boundaries

This model is valid for quarterly CECL expected credit loss estimation for Lakeshore's consumer installment and HELOC portfolio. It is not valid for commercial loans, mortgage portfolio CECL estimation, individual credit decisions, stress testing outside the CECL context, or any use case that requires loan-level forecasting beyond the CECL reporting context without MRM review.

---

## Section 8: Third-Party and Vendor Components

### 8.1 Third-Party Components Inventory

| Component | Vendor / Source | Version | Type | Purpose |
|---|---|---|---|---|
| Macroeconomic Scenario Forecasts | Moody's Analytics | Quarterly subscription | Data / Scenarios | Base, adverse, severely adverse macro variable forecasts |
| statsmodels | Open source | 0.14.0 | Library | Logistic regression estimation, coefficient output |
| TransUnion Bureau | TransUnion | API v4.1 | Data | Quarterly credit attribute refresh for active loans |

### 8.2 Vendor Validation Approach

**Moody's Analytics:** Moody's provides detailed documentation of their scenario construction methodology and historical scenario accuracy reports. The CFO and external validator review the Moody's methodology documentation annually. Historical Moody's scenario accuracy (base case vs. actual) is reviewed as part of the annual validation. ASC 326 requires that the institution use "reasonable and supportable" forecasts — Moody's Analytics is widely accepted by auditors and regulators as meeting this standard.

**TransUnion:** Bureau attribute quality validated as described in Section 8.2 of the credit scoring example. Quarterly refresh completeness monitored by Credit Risk.

### 8.3 Customizations and Adjustments

No customizations to Moody's Analytics scenarios. The only adjustment is the CFO-approved qualitative overlay to the model ACL estimate, documented in Section 7.3 and reviewed by the external auditor.

### 8.4 Ongoing Vendor Oversight

Moody's Analytics quarterly scenario delivery is reviewed for reasonableness by the CFO within 5 business days of receipt. Scenarios that differ materially from prior quarter expectations (e.g., unemployment forecast shift > 1.5 percentage points) are discussed with Moody's before use.

---

## Section 9: GenAI and LLM Governance

N/A — This is a traditional logistic regression model with no generative AI or LLM components.

---

## Section 10: Validation and Monitoring Plan

### 10.1 Pre-Production Validation Requirements

| Validation Activity | Responsible Party | Required Before Production? | Status |
|---|---|---|---|
| Conceptual soundness review | Alan Marsh, Baker & Monroe LLP | Yes | Complete |
| Coefficient sign and significance review | Alan Marsh, Baker & Monroe LLP | Yes | Complete |
| Data quality independent assessment | Alan Marsh, Baker & Monroe LLP | Yes | Complete |
| Back-test (predicted vs. actual loss rates) | Alan Marsh, Baker & Monroe LLP | Yes | Complete |
| Macroeconomic scenario sensitivity review | Alan Marsh + Christine Hoang (CFO) | Yes | Complete |
| Audit Committee review and approval | Board Audit Committee | Yes | Complete (Q3 2024 Audit Committee meeting) |

### 10.2 Ongoing Monitoring Metrics and Thresholds

| Metric | Baseline (Development) | Warning Threshold | Action Threshold | Action Required |
|---|---|---|---|---|
| AUC-ROC (annual back-test) | 0.779 | < 0.750 | < 0.730 | MRM escalation; external validator immediate review |
| Back-test: predicted vs. actual loss rate | 9 bps underestimation | > 25 bps deviation (either direction) | > 40 bps deviation | CFO qualitative overlay review; potential model recalibration |
| Qualitative overlay as % of model ACL | 5 bps (Q4 2024) | > 25 bps | > 50 bps | Model recalibration required if overlay persistently large |
| PSI (loan portfolio score distribution) | ~0.0 | > 0.10 | > 0.25 | Credit quality distribution review; potential model recalibration |
| Scenario ACL sensitivity (adverse vs. base) | 34% difference | < 15% difference | N/A | May indicate insufficient macro sensitivity; external validator review |

### 10.3 Monitoring Frequency and Reporting

| Activity | Frequency | Owner | Report Delivered To |
|---|---|---|---|
| Quarterly ACL bridge analysis | Quarterly | Robert Adeyemi / Finance | Christine Hoang (CFO), Audit Committee |
| Qualitative overlay documentation | Quarterly | Christine Hoang (CFO) | External auditor, MRM |
| Macro scenario reasonableness review | Quarterly | Christine Hoang (CFO) | Audit Committee |
| Annual model back-test | Annual | Alan Marsh (Baker & Monroe) | Board Audit Committee, MRM, external auditor |
| Full model validation | Annual | Alan Marsh (Baker & Monroe) | Board Audit Committee |

### 10.4 Performance Deterioration Triggers and Escalation

If the annual back-test shows predicted vs. actual loss rate deviation > 40 bps, or AUC below 0.730: external validator immediately notified; CFO and Audit Committee informed within 5 business days. Root cause analysis within 20 business days. If caused by a structural shift in the portfolio (credit quality, product mix), model recalibration is initiated. If caused by a macroeconomic regime change (scenario assumptions materially wrong), qualitative overlay adjustment made pending model redevelopment.

### 10.5 Override and Adjustment Policy

The CFO may apply a qualitative overlay to the model ACL estimate each quarter after model execution. Overlay amounts and rationale must be documented in the quarterly ACL memo, reviewed by external auditor annually, and disclosed in financial statement footnotes as required by ASC 326. The overlay represents management's judgment on model limitations (Section 7.2) and is itself subject to auditor scrutiny. Overlays larger than 50 bps of the model ACL trigger a model recalibration review.

### 10.6 Redevelopment Criteria

Model redevelopment triggered by: (1) AUC below 0.730 in annual back-test; (2) predicted vs. actual loss rate deviation > 40 bps for two consecutive annual back-tests; (3) a material change to FASB ASC 326 guidance requiring methodology changes; (4) a major change in Lakeshore's product mix requiring new sub-models; (5) Audit Committee or external auditor directing model redevelopment.

---

## Section 11: Implementation and Deployment

### 11.1 Production Architecture

The model runs as a Python batch process executed manually by the Credit Risk analyst (Robert Adeyemi) on the 15th business day of each month following quarter-end. The process reads loan-level data from the core banking system export (CSV), fetches the Moody's scenario file (Excel), executes the logistic regression scoring function, computes ECL by loan, aggregates to portfolio, and outputs: (1) loan-level ECL table (CSV) for Finance team; (2) aggregate ACL summary (Excel) for CFO review; (3) scenario sensitivity table. Results are reviewed and approved by the CFO before financial statement entry.

### 11.2 Input/Output Specifications

| Field | Description | Type | Format | Required? |
|---|---|---|---|---|
| **Input: loan_id** | Unique loan identifier | string | Internal format | Yes |
| **Input: credit_score_at_origination** | TransUnion score at origination | int | 300–850 | Yes |
| **Input: dti_at_origination** | Debt-to-income ratio | float | 0.0–1.0 | Yes |
| **Input: months_on_books** | Loan age in months | int | 0–360 | Yes |
| **Input: delinquency_status** | 30-day delinquency flag | binary | 0/1 | Yes |
| **Input: unemployment_rate_forecast** | Moody's quarterly forecast per scenario | float | 0.0–0.25 | Yes |
| **Output: pd_12mo_[scenario]** | 12-month PD per scenario | float | 0.0–1.0 | — |
| **Output: pd_lifetime_[scenario]** | Lifetime PD per scenario | float | 0.0–1.0 | — |
| **Output: ecl_[scenario]** | Loan-level ECL per scenario | float | $ amount | — |
| **Output: ecl_weighted** | Probability-weighted ECL | float | $ amount | — |

### 11.3 Performance and Latency Requirements

| Requirement | Target | Notes |
|---|---|---|
| **Full portfolio run time** | < 30 minutes | Batch process; not a real-time constraint |
| **Availability** | Manual execution on quarterly schedule | Not a 24/7 production system |

### 11.4 Integration Dependencies

| Dependency | System | Criticality | Fallback if Unavailable |
|---|---|---|---|
| Core banking export | Jack Henry Silverlake | High | Manual data pull from servicing system |
| Moody's Analytics scenario file | External subscription | High | CFO may use prior quarter scenarios with adjustment as documented overlay if Moody's delivery is delayed |
| TransUnion quarterly refresh | External | Medium | Prior quarter bureau data used with a stale-data overlay disclosed in ACL memo |

### 11.5 Rollback Plan

If the model produces an anomalous ACL estimate (e.g., > 20% change from prior quarter without portfolio-level explanation), the CFO may direct use of the prior quarter ACL with a qualitative adjustment while the current quarter model run is investigated. This is documented as a manual override in the ACL memo. Rollback to the v1.0 cohort approach is available as a fallback but would require disclosure as a methodology change in financial statement footnotes.

---

## Section 12: Governance

### 12.1 Roles and Responsibilities

| Role | Name / Team | Responsibilities |
|---|---|---|
| **Model Owner / CFO** | Christine Hoang | Financial statement accountability; quarterly ACL approval; scenario weight governance |
| **Lead Developer** | Robert Adeyemi, Senior Credit Risk Analyst | Model development, quarterly execution, documentation |
| **External Validator** | Alan Marsh, Baker & Monroe LLP | Annual independent validation; pre-approval validation; quarterly back-test review |
| **MRM Contact** | Patricia Kim, VP Internal Audit & Model Risk | Model risk governance; annual validation oversight; change management |
| **External Auditor** | Reinhardt Audit Group | Annual audit of ACL methodology and financial statement results |
| **Board Audit Committee** | Thomas Bergmann (Chair) + 3 members | Annual model approval; quarterly ACL oversight |

### 12.2 Model Inventory Registration

| Field | Value |
|---|---|
| **Model inventory system** | Lakeshore ModelLog (internal spreadsheet-based inventory) |
| **Model ID in inventory** | MDL-2024-003 |
| **Inventory registration date** | 2024-01-20 |
| **Tier in model inventory** | Tier 1 — Critical (financial reporting) |

### 12.3 Change Management

Any change to model features, coefficients, scenario weights, or sub-model structure requires: MRM notification (Patricia Kim), external validator review, and CFO approval before the change takes effect in a quarterly run. Changes must be disclosed in the quarterly ACL memo. The use of an external validator (Baker & Monroe LLP) for primary validation rather than an internal team is a recognized limitation for a community bank of Lakeshore's size — knowledge continuity is maintained through comprehensive MDD documentation and the requirement for annual full re-documentation.

### 12.4 Document Retention

This MDD and all supporting artifacts shall be retained for a minimum of 7 years following model retirement per Lakeshore's records retention policy and FASB ASC 326 audit trail requirements.

---

## Appendix A: Glossary

| Term | Definition |
|---|---|
| **ACL** | Allowance for Credit Losses — balance sheet reserve for expected credit losses under CECL |
| **AUC-ROC** | Area Under the Receiver Operating Characteristic Curve |
| **CECL** | Current Expected Credit Loss — FASB ASC Topic 326 accounting standard requiring lifetime expected loss estimation |
| **EAD** | Exposure at Default — outstanding balance at the time of default |
| **ECL** | Expected Credit Loss — product of PD × LGD × EAD |
| **FASB ASC 326** | Financial Accounting Standards Board Accounting Standards Codification Topic 326 — the CECL standard |
| **HELOC** | Home Equity Line of Credit |
| **LGD** | Loss Given Default — proportion of EAD lost in the event of default |
| **PD** | Probability of Default — estimated probability a borrower defaults within a specified time horizon |
| **Provision for Credit Losses** | Income statement charge that funds the ACL reserve |
| **SR 26-2** | Supervisory Guidance on Model Risk Management (April 17, 2026) |

---

## Appendix B: References

| Reference | Description | Location |
|---|---|---|
| SR 26-2 | Supervisory Guidance on Model Risk Management (April 17, 2026) | `_mrm-mdd-kit/governance/sr26-2.md` |
| FASB ASC Topic 326 | Current Expected Credit Loss accounting standard | FASB.org |
| Moody's Analytics CECL Scenario Documentation | Quarterly scenario construction methodology | Moody's Analytics client portal; retained in model file |
| Validation Report — MDL-2024-003 v2.0 | External validation by Alan Marsh, Baker & Monroe LLP | Lakeshore ModelLog repository |

---

## Appendix C: Model Change Log

| Version | Date | Author | Type of Change | MRM Approval Required? | Summary |
|---|---|---|---|---|---|
| 1.0.0 | 2022-09-01 | Robert Adeyemi | Initial CECL adoption model | N/A | Cohort-based historical loss rate approach for CECL adoption |
| 1.1.0 | 2023-04-15 | Robert Adeyemi | Scenario weight update | MRM notification | Adverse scenario weight increased from 25% to 30% |
| 2.0.0 | 2024-10-01 | Robert Adeyemi | Major redevelopment | Full external validation | Loan-level logistic regression replacing cohort approach; HELOC sub-model added |

---

## Appendix D: Model Coefficients (Logistic Regression)

> The full coefficient table is provided here for external auditor and regulatory examiner review. All coefficients are statistically significant at the 5% level. Signs are consistent with credit risk theory.

| Feature | Coefficient | Std. Error | p-value | Marginal Effect | Direction |
|---|---|---|---|---|---|
| Credit score at origination (bin 660–719) | -0.412 | 0.048 | < 0.001 | -0.031 | Expected ↓ risk |
| Credit score at origination (bin 720–779) | -0.847 | 0.052 | < 0.001 | -0.063 | Expected ↓ risk |
| Credit score at origination (bin ≥ 780) | -1.284 | 0.061 | < 0.001 | -0.096 | Expected ↓ risk |
| Debt-to-income ratio | +0.624 | 0.071 | < 0.001 | +0.047 | Expected ↑ risk |
| Months on books (12–36 months) | +0.218 | 0.039 | < 0.001 | +0.016 | Peak hazard period |
| Months on books (37–72 months) | -0.104 | 0.041 | 0.011 | -0.008 | Declining hazard |
| Months on books (> 72 months) | -0.391 | 0.058 | < 0.001 | -0.029 | Seasoned loan lower risk |
| 30-day delinquency flag | +1.847 | 0.094 | < 0.001 | +0.138 | Strongest individual predictor |
| Unemployment rate (macro) | +0.803 | 0.112 | < 0.001 | +0.060 | Macro sensitivity |
| GDP growth rate (macro) | -0.347 | 0.089 | < 0.001 | -0.026 | Macro sensitivity |
| 10-year Treasury yield (macro) | +0.214 | 0.067 | 0.001 | +0.016 | Rate environment |
| Loan-to-value (HELOC only) | +0.538 | 0.083 | < 0.001 | +0.040 | HELOC collateral risk |
| [Additional features: 6 remaining] | [See model file] | — | — | — | — |
| **Intercept** | -4.218 | 0.124 | < 0.001 | — | Base log-odds |

---

## Appendix E: Experiment Log

| Experiment ID | Date | Hypothesis | Approach | Result | Decision |
|---|---|---|---|---|---|
| EXP-2024-001 | 2024-02-15 | Survival analysis (Cox PH) superior to logistic regression for lifetime PD | Cox PH model estimated on loan-quarter panel | AUC 0.791 vs. logistic 0.779; marginal improvement; significantly more complex to implement and explain | Rejected; logistic regression preferred for interpretability |
| EXP-2024-002 | 2024-03-10 | XGBoost improves discrimination for HELOC sub-model | XGBoost trained on HELOC subset | AUC 0.812 vs. logistic 0.758; significant gain but not interpretable for auditor review | Rejected; logistic regression retained for consistency and auditability |
| EXP-2024-003 | 2024-04-01 | Separate sub-models for installment and HELOC justified statistically | Likelihood ratio test for pooled vs. separate | Chi-square 47.3, p < 0.001; separate models statistically justified | Adopted; separate installment and HELOC sub-models |
| EXP-2024-004 | 2024-05-20 | Scenario weights 60%/25%/15% more conservative than 50%/30%/20% | Sensitivity analysis on alternative weights | ACL difference of +$2.1M under conservative weights | Rejected; CFO adopted Moody's recommended 50%/30%/20% as more defensible to auditors |
