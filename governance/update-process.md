# Regulatory Monitoring and Kit Update Process

This document explains how to monitor for regulatory changes affecting this kit, determine when a kit update is required, and execute a versioned release so downstream project submodules can deliberately adopt changes.

---

## Why This Process Exists

The MDD template and examples in this kit are grounded in SR 26-2 (April 17, 2026). When the regulatory landscape changes — new guidance, revised standards, new agency interpretation — the kit must be updated to remain accurate. Downstream projects that have added this kit as a submodule should never inherit a regulatory change silently; they bump their submodule deliberately, with a clear understanding of what changed and why.

---

## Regulatory Sources to Monitor

These are the authoritative sources for changes that could affect this kit. Check each at the frequency noted.

| Source | What to Watch For | Check Frequency | URL |
|---|---|---|---|
| Federal Reserve SR Letters | New or revised model risk management guidance | Monthly | federalreserve.gov/supervisionreg/srletters |
| OCC Bulletins | OCC model risk guidance; interagency model risk statements | Monthly | occ.gov/news-issuances/bulletins |
| FDIC Financial Institution Letters | FDIC model risk guidance; AI/ML guidance | Monthly | fdic.gov/news/financial-institution-letters |
| FinCEN Advisories | BSA/AML model governance; SAR filing guidance | Monthly | fincen.gov/resources/advisories |
| CFPB Supervisory Guidance | Consumer model governance; adverse action requirements | Quarterly | consumerfinance.gov/supervisory-guidance |
| FASB ASC 326 Updates | CECL standard amendments; implementation guidance | Quarterly | fasb.org/standards |
| Interagency AI/ML Statements | Any joint agency statement on AI/ML model governance | Monthly | All three agency sites above |

---

## What Triggers a Kit Update

Not every regulatory development requires a kit update. Use this framework to decide.

### Major update required (bump minor or major version)

- A new or revised SR Letter, OCC Bulletin, or FDIC FIL that materially changes model risk management requirements
- A new interagency statement on AI/ML governance that affects how LLM-based models should be documented
- A FASB ASC 326 amendment that changes CECL model documentation requirements
- A FinCEN advisory that materially changes BSA/AML model governance expectations
- Any regulatory development that would make a section of `template/MDD.md` inaccurate or non-compliant

### Minor update required (bump patch version)

- Clarifying guidance that does not change requirements but adds interpretive nuance
- A correction to factual content in `governance/sr26-2.md` or other governance documents
- An improvement to the template or examples that does not change the regulatory basis
- A new example added to the `examples/` directory

### No update required

- An agency speech or testimony that does not constitute formal guidance
- Industry commentary or white papers (not authoritative)
- A regulatory development in an unrelated area (e.g., capital rules, liquidity) with no model governance implications
- Minor wording clarifications that do not change substantive requirements

---

## Versioning Convention

This kit uses semantic versioning: `vMAJOR.MINOR.PATCH`

| Version Component | When to Bump | Example |
|---|---|---|
| **MAJOR** | A new regulation supersedes the current basis (as SR 26-2 superseded SR 11-7) | v1.x.x → v2.0.0 |
| **MINOR** | New content added or material requirements change within the current regulatory basis | v1.0.x → v1.1.0 |
| **PATCH** | Bug fixes, corrections, non-substantive improvements | v1.0.0 → v1.0.1 |

**Current version:** v1.0.0 | **Regulatory basis:** SR 26-2 (April 17, 2026)

A MAJOR version bump (e.g., a new superseding regulation) is a significant event for downstream projects — it signals that the regulatory basis has changed and project MDDs may need revision. MINOR and PATCH bumps are lower urgency.

---

## Update Process: Step by Step

### Step 1: Identify the change

Monitor the sources listed above. When a potentially relevant regulatory development is identified:

1. Read the full guidance document (not just the press release or summary)
2. Assess whether it affects the kit's regulatory basis, template, examples, or governance documents
3. Use the "What Triggers a Kit Update" framework above to determine the update type

### Step 2: Draft the changes

Depending on the change type:

- **Regulatory basis change:** Update `governance/sr26-2.md` (or add a new governance document) with the full authoritative text; update all references throughout the template, how-to-fill-out guide, CLAUDE.md, and examples
- **Template change:** Update `template/MDD.md`; update `governance/how-to-fill-out.md` for any affected sections; assess whether examples need updating
- **Example update:** Update the relevant example MDD(s) and scenario(s)
- **CLAUDE.md change:** Update kit instructions if the workflow or guidance has changed

### Step 3: Update CHANGELOG.md

Add an entry to `CHANGELOG.md` describing:
- What changed
- Why it changed (cite the specific regulatory document and date)
- Which files were modified
- Impact on downstream projects (do existing MDDs need revision?)

### Step 4: Review

For MAJOR or MINOR version bumps, the changes should be reviewed by someone with MRM or regulatory expertise before release — either an internal review or community review via a GitHub pull request with a comment period.

For PATCH bumps, self-review by the maintainer is sufficient.

### Step 5: Tag and release

```bash
# Create the version tag
git tag -a v1.1.0 -m "v1.1.0: [brief description of change]"
git push origin main --tags

# Create a GitHub release
gh release create v1.1.0 \
  --title "v1.1.0: [brief description]" \
  --notes "$(cat <<'EOF'
## What changed
[Description of the regulatory or content change]

## Why it changed
[Citation of the specific guidance document and date]

## Impact on downstream projects
[Do existing project MDDs need revision? What sections? Urgency?]

## Files changed
[List of modified files]
EOF
)"
```

### Step 6: Notify downstream projects

GitHub Release Notes are the primary notification mechanism. Projects using this kit as a submodule will see the new release in the repository's Releases page.

For MAJOR version bumps, consider also:
- Adding a prominent notice to `README.md` noting the regulatory basis change
- Opening a GitHub Discussion explaining the impact for downstream projects

---

## How Downstream Projects Adopt Updates

Projects using this kit as a git submodule control when they adopt updates. They are never automatically updated — a submodule bump is a deliberate, traceable decision.

```bash
# Check the current kit version in use
cd _mrm-mdd-kit && git describe --tags && cd ..

# See what's new in the latest kit release
cd _mrm-mdd-kit && git fetch origin && git log HEAD..origin/main --oneline && cd ..

# Adopt the latest release
cd _mrm-mdd-kit && git pull origin main && cd ..
git add _mrm-mdd-kit
git commit -m "chore: bump mrm-mdd-kit to v1.1.0 — [brief reason]"
```

**When should a downstream project bump?**

| Kit Update Type | Urgency for Downstream |
|---|---|
| MAJOR — new regulatory basis | High — existing MDDs may be non-compliant with current guidance; plan revision |
| MINOR — material requirement change | Medium — review CHANGELOG to assess if affected sections need updating in project MDD |
| PATCH — corrections and improvements | Low — bump at next convenient opportunity |

---

## Assessing Impact on Existing Project MDDs

When adopting a MINOR or MAJOR kit update, the project team should assess whether their existing `MDD.md` needs revision. Use this checklist:

- [ ] Did the update change any regulatory requirements documented in Sections 1–12 of the template?
- [ ] Did the update add new required sections or fields?
- [ ] Did the update change the regulatory applicability table (Section 1.4)?
- [ ] Did the update affect the monitoring or validation requirements (Sections 10.1, 10.2)?
- [ ] Does the CHANGELOG entry indicate that existing MDDs need revision?

If any item is checked: open a task to update the project `MDD.md` before the next scheduled MRM review. Document the revision in Appendix C (Model Change Log) of the project MDD.

---

## Historical Regulatory Basis

| Kit Version | Regulatory Basis | Date Effective |
|---|---|---|
| v1.0.0 | SR 26-2 — Supervisory Guidance on Model Risk Management (Federal Reserve, OCC, FDIC) | April 17, 2026 |

*Note: SR 26-2 superseded SR 11-7 (April 4, 2011) and SR 21-8 (April 9, 2021). Both prior guidance documents are now retired.*
