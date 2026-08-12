# Workflow 01 — Enquiry Intake & Response

**Business function:** Corporate ticketing sales
**Trigger:** Inbound email to the corporate ticketing inbox
**Prompts:** P01 → P05

---

## Section Purpose

This workflow covers everything from an email arriving to a formal quotation being issued. It replaces a manual sequence in which the ticketing head reads, sorts, transcribes, researches and composes a reply to every corporate enquiry personally.

The five prompts run as a chain: each consumes the structured output of the one before it. P02 is the pivot — its JSON record is consumed by P03, P04 and, downstream, by P06 and P08 in Workflow 02.

## Chain Diagram

```
Corporate email arrives
          │
          ▼
   P01 · Intent classification        → routes: auto / human review / escalate
          │
          │ (NEW_ENQUIRY or BOOKING_CONFIRMATION only)
          ▼
   P02 · Entity extraction            → structured JSON record
          │
          ├── extraction_complete: false ──▶ P03 · Chase missing fields
          │                                        │
          │                                        ▼
          │                                  Client replies → re-enters at P01
          │
          ▼ extraction_complete: true
   P04 · Availability & pricing reply  ⚠ human approval required before send
          │
          ▼ client indicates intent to proceed
   P05 · Formal bulk quotation         ⚠ human approval required before send
          │
          ▼
   Accepted → Workflow 02 (record capture, validation, invoicing)
```

## Human-in-the-Loop Points

| Step | Human action | Why it cannot be removed |
|---|---|---|
| P01 | Reviews the exception queue | Confidence scores are not calibrated probabilities |
| P02 | Reviews records flagged `extraction_complete: false` | Missing data must not be inferred |
| P03 | Approves before send | Customer-facing text on a live opportunity |
| P04 | **Verifies every price against the live system** | Quoted prices may be treated as binding |
| P05 | Verifies arithmetic and discount band | Commercial document with a binding total |

P04 and P05 are reviewed on 100% of outputs, not sampled. Both state commercial terms to a customer.

## Prompts in This Section

| File | Prompt | Automation | Risk |
|---|---|---|---|
| [P01](P01-email-intent-classification.md) | Email intent classification & routing | High | Medium |
| [P02](P02-entity-extraction.md) | Structured entity extraction | Very High | High |
| [P03](P03-incomplete-enquiry-handler.md) | Incomplete enquiry handler | High | Medium |
| [P04](P04-availability-response.md) | Availability & pricing response | Medium–High | High |
| [P05](P05-bulk-quotation.md) | Formal bulk quotation | High | High |

## Section-Level Risk Note

The dominant risk across this workflow is **fabrication of commercial information** — prices, availability, or client details the model was not given. P02 and P04 both address this the same way: by giving the model an explicit, legitimate way to report absence (`null`, `ESCALATE_TO_HUMAN`) rather than forcing it to produce an answer. This was the single most important change made during iteration, and it is documented in the v1.1 → v1.2 logs of both prompts.
