# Skill: /validate-mdd

Runs the Step 5 completion checklist against an MDD.md file and reports which items pass, need attention, or are missing. Use this during kit development to validate example MDDs, or to spot-check a draft before marking it ready for MRM review.

---

## Step 1: Identify the target file

If the user invoked `/validate-mdd` without specifying a file, ask:
> "Which MDD.md should I validate? (e.g., `examples/fraud_detection_xgboost/MDD.md`, or `MDD.md` in the project root)"

If a path was provided, use it directly.

---

## Step 2: Read the MDD

Read the full file. Do not begin the checklist until you have read the complete document.

---

## Step 3: Run the completion checklist

Evaluate each item below. For each one, report:
- ✅ **Pass** — clearly satisfied
- ⚠️ **Needs attention** — present but incomplete, vague, or inconsistent
- ❌ **Missing** — not present or contains an unfilled `[FILL IN: ...]` placeholder

### Checklist

**Document Control**
- [ ] Model Name is present and specific (not a placeholder)
- [ ] Model ID is present (format `MDL-YYYY-NNN` or equivalent)
- [ ] Model Owner name and title are real (not `[FILL IN]`)
- [ ] MRM Contact name and title are real
- [ ] Effective Date is present
- [ ] Approval Sign-offs block is present (signatures may be `*(pending)*` but the block must exist)
- [ ] Appendix C (Change Log) has at least one entry

**Section 1: Executive Summary**
- [ ] Section 1.1 explains what the model does in plain English — a non-technical stakeholder should understand it
- [ ] Section 1.2 risk rating is present (Inherent Risk, Model Exposure, Model Purpose dimensions)
- [ ] Section 1.3 key risks are present — no omissions or softening

**Internal consistency check**
- [ ] Risk rating in Section 1.2 is consistent with the detailed assessment in Sections 3.1–3.4
- [ ] Validation rigor and monitoring frequency stated in Section 3.4 match what is written in Sections 10.1 and 10.3

**Section 6: Model Performance**
- [ ] Primary performance results are from the holdout/test set — NOT training set
- [ ] If no holdout results are present, this is flagged as a blocker (❌ Missing)

**Section 7: Assumptions and Limitations**
- [ ] Assumptions registry has at least 3 entries
- [ ] Every assumption rated High sensitivity has a compensating control documented
- [ ] No limitation is listed without an explicit "Accepted" decision or compensating control

**Section 10: Monitoring**
- [ ] Monitoring thresholds in Section 10.2 are numeric (not qualitative language like "significant degradation")
- [ ] PSI thresholds are present: Warning > 0.10, Action > 0.25 (or documented rationale for deviation)
- [ ] Escalation path is specified: who is notified, in what timeframe, what actions are available

**Conditional sections**
- [ ] Section 9 (GenAI and LLM Governance): either fully populated OR marked `N/A — [reason]`
- [ ] All other conditional sections are either populated or explicitly marked N/A with a reason

**No unfilled placeholders**
- [ ] No `[FILL IN: ...]` patterns remain anywhere in the document

---

## Step 4: Report findings

Present results in this format:

```
## MDD Validation Report
File: [path]
Date: [today]

### Summary
- ✅ Pass: [N]
- ⚠️ Needs attention: [N]
- ❌ Missing: [N]

### Findings

[List each ⚠️ and ❌ item with a specific note — what exactly is wrong or missing, and what section/line it's in. Skip ✅ items unless the user asks for the full list.]

### Verdict
[READY FOR REVIEW / NEEDS WORK BEFORE REVIEW]
```

If all required items pass with no ❌ or ⚠️, say:
> "This MDD passes the completion checklist. It is ready for MRM review submission."

If any ❌ items exist, say:
> "This MDD has [N] missing required items. These must be resolved before submission to MRM."
