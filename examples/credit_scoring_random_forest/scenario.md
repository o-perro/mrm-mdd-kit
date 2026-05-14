# Scenario: Credit Scoring Model — Cascade Consumer Lending

> **Fictional scenario.** All institution names, personnel, performance metrics, and business details are completely made up. This scenario exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## The Institution

**Cascade Consumer Lending** is the consumer lending division of a fictional regional bank holding company, Cascade Bancorp, headquartered in Portland, Oregon with approximately $28 billion in total assets. Cascade Consumer Lending originates personal installment loans ranging from $1,000 to $50,000 through its branch network and digital origination platform.

## The Business Problem

Cascade's personal loan underwriting team previously relied on a scorecard-based model from a third-party vendor (CreditView Analytics) that was last recalibrated in 2020. As Cascade's loan portfolio grew and its customer base shifted toward younger, thinner-file applicants through its digital channel, the vendor scorecard's discriminatory power degraded. Internal backtesting showed the vendor scorecard had an AUC of 0.721 on the current applicant population — below the 0.75 minimum threshold in Cascade's model risk policy.

## The Model

Cascade's Model Risk & Analytics team developed a Random Forest classifier that estimates each personal loan applicant's probability of 90-day default within 24 months of origination. The model score is used by underwriters as the primary input to the credit decision — approving, declining, or referring applicants to a senior underwriter for judgment-based review. Unlike the fraud model, this is a human-in-the-loop process: no application is declined by the model alone.

## The Team

- **Model Owner:** Rebecca Santos, SVP Consumer Lending
- **Lead Developer:** Nathan Park, Senior Data Scientist, Model Risk & Analytics
- **MRM Contact:** Helen Yu, Director of Model Risk Management
- **Primary Validator:** Carlos Reyes, Senior Model Validator

## Why This Model Is Interesting for MDD Purposes

This example demonstrates MDD documentation for a credit scoring model — one of the most regulated model types in banking. Key documentation challenges addressed: ECOA/Reg B adverse action reason code requirements, fair lending disparate impact assessment across protected class proxy variables, the random forest interpretability challenge (partially addressed with SHAP), and the specific SR 26-2 considerations for a model used in origination credit decisions. This is also an example of a model replacing a vendor scorecard — with documentation of the vendor validation approach for the prior model.
