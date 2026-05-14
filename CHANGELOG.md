# Changelog

All notable changes to mrm-mdd-kit are documented here. This file also serves as the regulatory change tracking log — every version that reflects a change in the regulatory basis is noted with the governing document and date.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/). Versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] — 2026-05-14

### Initial release

**Regulatory basis:** SR 26-2 — *Supervisory Guidance on Model Risk Management*, issued jointly by the Federal Reserve, OCC, and FDIC on April 17, 2026. SR 26-2 supersedes SR 11-7 (April 4, 2011) and SR 21-8 (April 9, 2021).

### Added

- `template/MDD.md` — SR 26-2 compliant blank MDD template; 12 sections, 4 appendices; dual human/LLM readable with `[FILL IN: ...]` placeholders throughout
- `governance/sr26-2.md` — full text of SR 26-2 in markdown
- `governance/how-to-fill-out.md` — section-by-section annotated guidance for MRM teams and Claude Code; includes LLM Collaboration Guide blocks for every section
- `governance/update-process.md` — regulatory monitoring sources, versioning convention, and downstream submodule update process
- `CLAUDE.md` — Claude Code instructions for MDD generation; human-LLM collaboration model; 5-step MDD workflow; section-specific guidance
- `examples/llm_complaint_classifier/` — fully completed fictional MDD for an LLM-based CFPB complaint classifier (Pinnacle Community Bank); demonstrates Section 9 GenAI/LLM governance
- `examples/fraud_detection_xgboost/` — fully completed fictional MDD for a real-time CNP fraud scoring model (Westbrook National Bank); demonstrates Critical materiality, SHAP adverse action, UDAAP
- `examples/credit_scoring_random_forest/` — fully completed fictional MDD for a personal loan PD scorecard (Cascade Consumer Lending); demonstrates ECOA/Reg B, fair lending, adverse action reason codes
- `examples/aml_monitoring_neural_network/` — fully completed fictional MDD for a BSA/AML suspicious activity scoring model (Harborview Financial); demonstrates SR 21-8 supersession, human-in-the-loop requirements
- `examples/cecl_logistic_regression/` — fully completed fictional MDD for a CECL expected credit loss model (Lakeshore Community Bank); demonstrates FASB ASC 326, external validation, macroeconomic scenario dependency
- `CHANGELOG.md`, `CONTRIBUTING.md`, `README.md`, `LICENSE` (MIT)

### Notes for downstream projects

This is the initial release. Projects adding this kit as a submodule should review `governance/update-process.md` for guidance on monitoring future releases and assessing impact on project MDDs.

---

*For the regulatory monitoring process and versioning convention, see [`governance/update-process.md`](governance/update-process.md).*
