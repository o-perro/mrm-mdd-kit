# Scenario: Fraud Detection Model — Westbrook National Bank

> **Fictional scenario.** All institution names, personnel, performance metrics, and business details are completely made up. This scenario exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## The Institution

**Westbrook National Bank** is a fictional large retail bank headquartered in Charlotte, North Carolina with approximately $210 billion in total assets. It operates nationally with approximately 9.4 million consumer credit and debit card accounts.

## The Business Problem

Westbrook's Fraud Risk Management team processes approximately 4.2 million card-not-present (CNP) transactions per day across its consumer credit and debit card portfolios. CNP fraud — fraudulent purchases made without the physical card present, typically via e-commerce — represents Westbrook's largest single fraud loss category at approximately $47 million annually before this model.

The prior fraud detection system was a third-party vendor rules engine (FraudGuard Pro) that applied approximately 300 static rules and velocity checks. While effective for known fraud patterns, the rules engine was slow to adapt to emerging fraud typologies and produced a false positive rate of 22% — meaning 22% of flagged transactions were legitimate, creating significant customer friction and a dedicated dispute resolution workload.

## The Model

Westbrook's Fraud Analytics team developed an XGBoost gradient boosting model that scores each CNP transaction in real time for probability of fraud. The model generates a fraud probability score from 0.0 to 1.0 within 85 milliseconds of transaction authorization request. Based on the score: low-scoring transactions proceed normally; mid-scoring transactions trigger a step-up authentication challenge (SMS OTP); high-scoring transactions are automatically declined and flagged for the fraud operations team.

## The Team

- **Model Owner:** Patricia Vance, SVP Fraud Risk Management
- **Lead Developer:** Kevin Osei, Lead Data Scientist, Fraud Analytics
- **Contributing Developers:** Anita Sharma, Data Engineer; Tyler Brooks, ML Engineer
- **MRM Contact:** Lawrence Kim, VP Model Risk Management
- **Primary Validator:** Stephanie Morales, Senior Model Validator

## Why This Model Is Interesting for MDD Purposes

This example demonstrates MDD documentation for a high-materiality, real-time ML model with significant business impact. It covers: high-volume real-time scoring architecture, XGBoost-specific documentation (feature importance, SHAP values, hyperparameter tuning), fair lending considerations for a fraud model, and the complexities of documenting a model that replaces a vendor rules engine. The model's speed requirement (< 85ms p99) and high exposure (9.4M accounts, $210B total assets bank) make it a canonical example of a Tier 1 high-materiality model.
