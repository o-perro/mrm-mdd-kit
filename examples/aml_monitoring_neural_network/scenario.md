# Scenario: AML Transaction Monitoring Model — Harborview Financial

> **Fictional scenario.** All institution names, personnel, performance metrics, and business details are completely made up. This scenario exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## The Institution

**Harborview Financial** is a fictional mid-size bank headquartered in Seattle, Washington with approximately $62 billion in total assets. It operates across the Pacific Northwest and serves a significant international remittance customer base — a mix that elevates its BSA/AML risk profile relative to a typical community bank.

## The Business Problem

Harborview's BSA/AML compliance team processes approximately 2.1 million transactions per day. The existing rule-based transaction monitoring system (TMS) generates approximately 4,800 alerts per month that must be reviewed by AML analysts for potential Suspicious Activity Report (SAR) filing. The investigation-to-SAR conversion rate is approximately 4.1% — meaning analysts spend approximately 96% of their time investigating alerts that do not result in a SAR filing. This high false positive burden consumes significant analyst capacity and risks desensitizing analysts to genuine suspicious activity patterns.

## The Model

Harborview's Compliance Analytics team developed a neural network model that scores each transaction for suspicious activity risk, supplementing (not replacing) the existing rule-based TMS. The model serves as a prioritization layer: it re-ranks the alert queue so that high-risk alerts are reviewed first and low-risk alerts can be batch-reviewed or de-prioritized within the review window. Additionally, the model surfaces net-new alerts — transactions that did not trigger a rule but that the model identifies as high-risk based on behavioral pattern recognition.

## The Team

- **Model Owner:** Frank Nakamura, Chief BSA/AML Officer
- **Lead Developer:** Aisha Oduya, Senior Data Scientist, Compliance Analytics
- **MRM Contact:** Sandra Lee, VP Model Risk Management
- **Primary Validator:** Michael Torres, Senior Model Validator

## Why This Model Is Interesting for MDD Purposes

This example demonstrates MDD documentation for a neural network-based AML model — one of the most sensitive model types in banking. Key documentation challenges: (1) high inherent risk from neural network black-box complexity; (2) BSA/AML regulatory applicability including the SR 26-2 supersession of SR 21-8; (3) the human-in-the-loop requirement is absolute in AML (no automated SAR filing); (4) the "dual-use" architecture where the model both prioritizes existing alerts and generates net-new ones creates distinct governance challenges; (5) SAR filing decisions must be defensible to FinCEN examiners.
