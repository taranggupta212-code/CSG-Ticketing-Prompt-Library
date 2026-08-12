# Prompt Index

All ten prompts, with direct links. Full documentation for each — RACE prompt text, intended workflow, problem solved, automation potential, risks, and the v1.0 → v1.2 iteration log — lives in the prompt file itself.

## Workflow 01 — Enquiry Intake & Response

| ID | Prompt | Automation | Risk |
|---|---|---|---|
| P01 | [Email intent classification & routing](../workflows/01-enquiry-intake/P01-email-intent-classification.md) | High | Medium |
| P02 | [Structured entity extraction](../workflows/01-enquiry-intake/P02-entity-extraction.md) | Very High | **High** |
| P03 | [Incomplete enquiry handler](../workflows/01-enquiry-intake/P03-incomplete-enquiry-handler.md) | High | Medium |
| P04 | [Availability & pricing response](../workflows/01-enquiry-intake/P04-availability-response.md) | Medium–High | **High** |
| P05 | [Formal bulk quotation](../workflows/01-enquiry-intake/P05-bulk-quotation.md) | High | **High** |

## Workflow 02 — Record Capture, Invoicing & Audit

| ID | Prompt | Automation | Risk |
|---|---|---|---|
| P06 | [Spreadsheet row generation](../workflows/02-record-capture-audit/P06-spreadsheet-row-generation.md) | Very High | Medium |
| P07 | [Validation & anomaly check](../workflows/02-record-capture-audit/P07-validation-anomaly-check.md) | Very High | Medium |
| P08 | [Invoice content generation](../workflows/02-record-capture-audit/P08-invoice-content.md) | High | **High** |
| P09 | [Weekly reconciliation summary](../workflows/02-record-capture-audit/P09-weekly-reconciliation.md) | Very High | Medium |
| P10 | [Demand & conversion report](../workflows/02-record-capture-audit/P10-demand-conversion-report.md) | Very High | Medium |

## Test Evidence

- [P04 · v1.0 vs v1.2 output comparison](../evidence/test-outputs/P04-v1.0-vs-v1.2-comparison.md)

---

Prompts are stored by workflow rather than duplicated into a flat folder, so each has exactly one canonical version and the commit history tracks a single line of development per prompt.
