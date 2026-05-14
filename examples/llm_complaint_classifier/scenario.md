# Scenario: LLM Complaint Classifier — Pinnacle Community Bank

> **Fictional scenario.** All institution names, personnel, performance metrics, and business details are completely made up. This scenario exists solely to demonstrate the expected depth, tone, and structure of a completed MDD. No real company data was used.

---

## The Institution

**Pinnacle Community Bank** is a fictional mid-size regional bank headquartered in Columbus, Ohio with approximately $45 billion in total assets. It operates across seven Midwestern states with a customer base of roughly 2.1 million retail and small business accounts.

## The Business Problem

Pinnacle's Customer Experience Operations team handles approximately 85,000 inbound customer contacts per month across three channels: email, live chat, and phone (post-call transcripts generated via ASR). Under CFPB guidelines, any contact that constitutes a "complaint" — an expression of dissatisfaction with a product, service, or the bank's handling of a prior interaction — must be formally logged, acknowledged, and resolved within defined timeframes.

Before this model, Pinnacle's complaint identification process was entirely manual. A team of 14 contact center quality analysts reviewed a random 12% sample of all contacts for complaint identification, flagging those that met the regulatory definition. The rest were reviewed only if escalated by a frontline agent. This created two problems: (1) a meaningful percentage of unsampled contacts containing complaints went unidentified, exposing the bank to CFPB regulatory risk; and (2) the manual review process was slow — identified complaints often took 3–5 business days to enter the formal CFPB workflow.

## The Model

Pinnacle's Data Science team built a binary LLM-based classifier using a hosted large language model API. The model reads the full text of each inbound customer contact and classifies it as either **complaint** or **non-complaint** according to the CFPB regulatory definition. Contacts classified as complaints are automatically queued in Pinnacle's complaint management system, Resolve360, where a complaint specialist completes formal intake within one business day.

The model processes all inbound contact text — not a sample — giving Pinnacle 100% coverage for the first time. A human complaint specialist reviews every AI-identified complaint before it enters the formal CFPB workflow, satisfying human-in-the-loop governance requirements.

## The Team

- **Model Owner:** Sarah Kowalski, SVP Customer Experience Operations
- **Lead Developer:** Marcus Chen, Senior Data Scientist, Model Risk & Analytics
- **MRM Contact:** Diana Flores, Director of Model Risk Management
- **Primary Validator:** James Whitfield, Senior Model Validator (to be assigned at validation kick-off)

## Why This Model Is Interesting for MDD Purposes

This example demonstrates how to document an LLM-based model under SR 26-2 — a guidance that explicitly excludes generative AI from its formal scope while acknowledging institutions must still govern these tools. Section 9 of this MDD shows how to apply SR 26-2 principles in the spirit of sound governance for an LLM-based model, including prompt versioning, output schema validation, hallucination risk assessment, and human-in-the-loop controls. This is the approach leading banks are taking in practice.
