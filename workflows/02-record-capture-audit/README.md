# Workflow 02 — Record Capture, Invoicing & Audit

**Business function:** Sales administration and finance
**Trigger:** Quotation accepted by the client
**Prompts:** P06 → P10

---

## Section Purpose

This workflow converts a closed sale into a permanent, auditable record and an issued invoice, then reports on the register weekly. It replaces a manual process in which booking details are re-typed into a spreadsheet — a second transcription of data already entered once during enquiry — with no systematic check before that data becomes CSG's audit record.

Two of the five prompts (P07, P09) are **controls rather than production steps**. They exist to catch errors made by the other prompts in the library. This is deliberate: an automated pipeline needs automated checking, and the checks must be independent of the step they are checking.

## Chain Diagram

```
Quotation accepted (from P05)
          │
          ▼
   P06 · Spreadsheet row generation     → register row + requires_review flag
          │
          ▼
   P07 · Validation & anomaly check     ── FAIL ──▶ Exception queue
          │                                          (administrator resolves)
          │ PASS
          ▼
   Row committed to corporate sales register
          │
          ▼
   P08 · Invoice content generation      ⚠ finance review before release
          │
          ▼
   Invoice issued


   ── WEEKLY, across the whole register ──

   P09 · Reconciliation summary   → exceptions, ranked P1/P2/P3 → ticketing head
   P10 · Demand & conversion report → performance metrics       → ticketing head
```

## Human-in-the-Loop Points

| Step | Human action | Why it cannot be removed |
|---|---|---|
| P06 | Completes rows flagged `requires_review` | NULL values must not be invented |
| P07 | Signs off blocking failures with a recorded reason | An unread control queue is not a control |
| P08 | **Finance reviews before release** | Tax invoices are regulated documents |
| P09 | Actions the exception list before month-end close | Detection without action delivers nothing |
| P10 | Interprets the report and makes decisions | The prompt is prohibited from recommending actions |

## Prompts in This Section

| File | Prompt | Automation | Risk |
|---|---|---|---|
| [P06](P06-spreadsheet-row-generation.md) | Spreadsheet row generation | Very High | Medium |
| [P07](P07-validation-anomaly-check.md) | Validation & anomaly check | Very High | Medium |
| [P08](P08-invoice-content.md) | Invoice content generation | High | High |
| [P09](P09-weekly-reconciliation.md) | Weekly reconciliation summary | Very High | Medium |
| [P10](P10-demand-conversion-report.md) | Demand & conversion report | Very High | Medium |

## The Register Schema

| Column | Format |
|---|---|
| Client Name | Registered organisation name, title case |
| Business Number | 11 digits, no spaces or punctuation |
| Fixture | Opponent and date, DD/MM/YYYY |
| Tickets | Whole number, digits only |
| Unit Price | 2 decimal places, no currency symbol |
| Total | 2 decimal places, no currency symbol |

Currency symbols and thousands separators are excluded deliberately — they break downstream spreadsheet formulas and are the most common defect in the current manual register.

## Section-Level Risk Note

The most significant limitation in this workflow is stated openly rather than mitigated away: **language models are unreliable calculators.** P05 computes a quotation total, P07 checks it, and P08 derives a GST component from it. All three rely on arithmetic the model performs by pattern rather than by computation.

Three controls address this: each prompt must show its own working so a human can verify at a glance; P07 checks P06 and P05 independently; and the recommended production architecture moves arithmetic into spreadsheet formulas, leaving the model to format and explain rather than calculate. Prompting reduces this risk. It does not eliminate it, and the business case should not claim otherwise.
