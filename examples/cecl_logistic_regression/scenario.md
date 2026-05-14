# Scenario: CECL Expected Credit Loss Model — Lakeshore Community Bank

> **Fictional scenario.** All institution names, personnel, performance metrics, and business details are completely made up. This scenario exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## The Institution

**Lakeshore Community Bank** is a fictional community bank headquartered in Madison, Wisconsin with approximately $4.8 billion in total assets. It operates primarily in Wisconsin and northern Illinois with a consumer loan portfolio dominated by personal installment loans and home equity lines of credit.

## The Business Problem

FASB ASC Topic 326 — the Current Expected Credit Loss (CECL) standard — requires financial institutions to estimate and reserve for lifetime expected credit losses on their loan portfolios at each reporting period, rather than recognizing losses only when incurred. For Lakeshore, this affects its consumer loan portfolio of approximately $1.9 billion in outstanding balances.

Prior to adopting CECL, Lakeshore used a historical loss rate approach for its allowance for loan and lease losses (ALLL). CECL's lifetime expected loss requirement demands a more forward-looking model that incorporates macroeconomic forecasts and accounts for the full contractual life of each loan — a fundamentally different modeling challenge.

## The Model

Lakeshore's Finance and Credit Risk team developed a logistic regression-based CECL model that estimates 12-month and lifetime probability of default (PD) for each loan in the consumer portfolio at each quarter-end reporting date. The model combines account-level credit attributes with macroeconomic scenario forecasts (base case, downside, severe downside) to produce an expected loss estimate that feeds into the quarterly allowance for credit losses (ACL) disclosure.

## The Team

- **Model Owner:** Christine Hoang, CFO / Chief Financial Officer
- **Lead Developer:** Robert Adeyemi, Senior Credit Risk Analyst
- **MRM Contact:** Patricia Kim, VP Internal Audit & Model Risk
- **Primary Validator:** Alan Marsh, External Validator (Baker & Monroe LLP, risk advisory)

## Why This Model Is Interesting for MDD Purposes

This example demonstrates MDD documentation for a CECL model — one of the most highly scrutinized model types for community banks, with direct FASB/regulatory capital implications. Key documentation challenges: (1) CECL requires lifetime loss estimation which introduces long-horizon forecast uncertainty; (2) macroeconomic scenario dependency means the model output changes with each scenario forecast update; (3) logistic regression is chosen for interpretability over predictive power — a deliberate governance trade-off; (4) an external validator (third-party firm) is used rather than an internal team, which is common at community banks; (5) the model output feeds directly into regulatory financial reporting, making it the highest-stakes reporting model at the institution.
