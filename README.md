# Corporate Ticketing Prompt Library

> **BUS4005 Assessment 1 — AI Prompt Library and Business Proposal Pitch**
> La Trobe University
> Business field: **Sports & entertainment ticketing operations (B2B corporate sales)**

---

## Organisation Note

**Capital Sports Group (CSG)** is a fictional composite organisation used for this assessment: a mid-tier professional T20 cricket franchise operating a 40,000-seat home venue, with a corporate ticketing desk handling bulk enquiries of 20+ seats from corporate hospitality buyers, travel agencies and community groups.

The organisation is illustrative. The workflow, the operational pain points, and the industry context are representative of genuine practice in franchise ticketing operations. All figures in the business case below are **stated assumptions with the arithmetic shown**, not measured data.

---

## The Problem

CSG's corporate ticketing desk is a single-person function. Every inbound enquiry is handled end-to-end by hand:

1. The ticketing head reads and sorts every email personally.
2. Client details are transcribed manually into a spreadsheet.
3. Availability and pricing are looked up and a reply is composed from scratch.
4. Quotations are built by copying and overwriting a previous quotation.
5. Closed bookings are re-typed a second time into the corporate sales register.
6. Invoices are prepared by overwriting a previous invoice.
7. The register is reconciled only at month-end, or when a client disputes something.

Three consequences follow. **Response time** — corporate buyers approach several venues simultaneously, and slow replies lose bookings to faster competitors. **Data integrity** — the same information is typed twice, with no systematic check before it enters what is legally the audit record. **No visibility** — enquiry volume, conversion and response time all sit in the register and are never aggregated, so pricing and inventory-release decisions are made on intuition.

---

## The Solution

Ten prompts across two chained workflows, automating the enquiry-to-audit pipeline end to end.

```
workflows/
├── 01-enquiry-intake/          Email arrives → formal quotation issued
│   ├── P01 · Email intent classification & routing
│   ├── P02 · Structured entity extraction
│   ├── P03 · Incomplete enquiry handler
│   ├── P04 · Availability & pricing response
│   └── P05 · Formal bulk quotation
│
└── 02-record-capture-audit/    Sale closed → record captured, invoiced, reported
    ├── P06 · Spreadsheet row generation
    ├── P07 · Validation & anomaly check
    ├── P08 · Invoice content generation
    ├── P09 · Weekly reconciliation summary
    └── P10 · Demand & conversion report
```

These are not ten standalone prompts. **P02 is the pivot** — the structured record it produces is consumed by P03, P04, P06 and P08. A single extraction feeds the reply, the register row and the invoice, which is what removes the duplicate data entry entirely.

**P07 and P09 are controls, not production steps.** They exist to catch errors made by the other prompts. An automated pipeline requires automated checking, independent of the step being checked.

---

## Library Summary

| ID | Prompt | Workflow | Automation | Risk | Human role |
|---|---|---|---|---|---|
| P01 | Email intent classification | Intake | High | Medium | Exception queue |
| P02 | Entity extraction | Intake | Very High | **High** | Review flagged records |
| P03 | Incomplete enquiry handler | Intake | High | Medium | Approve before send |
| P04 | Availability & pricing reply | Intake | Medium–High | **High** | Verify every price |
| P05 | Formal bulk quotation | Intake | High | **High** | Verify arithmetic |
| P06 | Spreadsheet row generation | Capture | Very High | Medium | Complete flagged rows |
| P07 | Validation & anomaly check | Capture | Very High | Medium | Sign off failures |
| P08 | Invoice content generation | Capture | High | **High** | Finance review |
| P09 | Weekly reconciliation | Capture | Very High | Medium | Action exceptions |
| P10 | Demand & conversion report | Capture | Very High | Medium | Interpret and decide |

Four prompts are rated HIGH risk. All four either state commercial terms to a customer or write to a regulated or audited record, and all four are reviewed on 100% of outputs.

---

## Prompting Strategy

Every prompt uses the **RACE framework** — Role, Action, Context, Expectations — applied as follows.

| Element | How it is used | Why it matters here |
|---|---|---|
| **Role** | A specific job function at CSG (operations analyst, data administrator, accounts receivable officer, auditor) | Narrow professional roles produce more consistent domain vocabulary than a generic "assistant" |
| **Action** | Numbered sequential steps, not a single instruction | Multi-step tasks degrade when compressed; explicit ordering is a form of chain-of-thought scaffolding |
| **Context** | Business rules, fixed vocabularies, tie-break rules, format tables | Removes the ambiguity the model would otherwise resolve by guessing |
| **Expectations** | Exact output schema, length ceilings, prohibitions, and a defined refusal path | Machine-readable output is what makes chaining possible |

Three techniques carry disproportionate weight across the library:

**Closed vocabularies.** P01's six categories, P07's seven checks, P09's six exception types. A fixed list converts an open-ended judgement into a classification, which is stable across runs.

**Explicit refusal paths.** Every prompt that could fabricate has a legitimate way to report absence — `null`, `ESCALATE_TO_HUMAN`, `INCOMPLETE_QUOTATION`, `requires_review`. This single pattern eliminated the most dangerous failure mode found in testing.

**Show-your-working requirements.** P01 cites trigger phrases, P07 states its own calculation, P08 shows the GST formula. Requiring the model to expose its reasoning converts errors from silent to visible, which is what makes a human review step take seconds instead of minutes.

---

## Business Case

### Assumptions

All figures below rest on these stated assumptions. **They are estimates for the purpose of this proposal, not measured results.**

| Assumption | Value | Basis |
|---|---|---|
| Working days per year | 250 | Standard Australian business year |
| Corporate emails received | 10 per working day | Illustrative for a single-desk operation |
| Enquiries reaching pricing stage | 7 per day | Excludes out-of-scope and payment queries |
| Confirmed bookings | 4 per day | Implies ~57% enquiry-to-booking conversion |
| Enquiries arriving incomplete | 3 per day (~30%) | Illustrative |
| Coordinator salary | $75,000 | Mid-range Australian ticketing coordinator |
| On-costs multiplier | 1.25× | Superannuation, leave loading, payroll tax |
| Productive hours per year | 1,720 | 1,950 gross less leave and non-productive time |
| **Fully-loaded hourly rate** | **$55/hour** | ($75,000 × 1.25) ÷ 1,720 = $54.51, rounded |

### Derivation

| Prompt | Task time | Frequency | Baseline hrs/yr | Reduction | Hours saved |
|---|---|---|---|---|---|
| P01 | 2 min | 10/day | 83 | 75% | 62 |
| P02 | 4 min | 10/day | 167 | 80% | 133 |
| P03 | 6 min | 3/day | 75 | 75% | 56 |
| P04 | 12 min | 7/day | 350 | 60% | 210 |
| P05 | 15 min | 4/day | 250 | 70% | 175 |
| P06 | 5 min | 4/day | 83 | 85% | 71 |
| P07 | rework avoided | — | 40 | — | 40 |
| P08 | 10 min | 4/day | 167 | 70% | 117 |
| P09 | month-end remediation | — | 50 | 60% | 30 |
| P10 | **excluded — see below** | — | — | — | **0** |
| | | | **1,265** | | **894** |

**Worked example (P04):** 12 minutes × 7 enquiries × 250 days = 21,000 minutes = 350 hours. At 60% reduction: 210 hours saved. The reduction is lowest of any prompt because the human review step is substantive — every price must be verified against the live system — not cursory.

**P10 is excluded from savings deliberately.** CSG does not currently produce demand reporting at all, so automating it saves no existing labour. Counting hypothetical hours for work nobody does would inflate the case. P10's value is decision quality, and it belongs in the qualitative benefits below.

### Headline figure — and what it actually means

**~894 hours per year of capacity released.** At $55/hour fully loaded, that represents **~$49,000 in annual labour capacity.**

That figure requires an important qualification. **It is not a cash saving.** CSG has no plan to reduce headcount, so the released capacity does not leave the payroll — it is redeployed. The realistic financial framing is:

- **Year 1 cash saving: approximately $0.**
- **Capacity value: ~$49,000**, realised as the corporate desk absorbing higher enquiry volume without an additional hire.
- **Avoided cost:** if enquiry volume grows as projected, the alternative to this library is a second coordinator at ~$94,000 fully loaded.

The avoided incremental hire is the honest business case. Presenting released hours as banked cash would misrepresent it.

### Sensitivity

The assumptions above are uncertain, so the case should be read as a range:

| Scenario | Emails/day | Hours saved | Capacity value |
|---|---|---|---|
| Conservative | 5 | ~450 | ~$24,500 |
| **Base** | **10** | **~894** | **~$49,000** |
| High volume (finals) | 15 | ~1,340 | ~$73,500 |

Even at half the assumed volume the case holds, because the library's cost is near-zero once built.

### Benefits that are not hours

- **Response time** to corporate buyers — the direct driver of competitive win rate in bulk ticketing.
- **Audit integrity** — every register row validated before commit, where currently none are.
- **Elimination of duplicate data entry** — one extraction serves the reply, the register and the invoice.
- **Demand visibility** — pricing and inventory decisions supported by data that already exists but is never used.
- **Scalability at fixture-release peaks**, when enquiry volume spikes and the manual process degrades most.

---

## Risk Framework

| Risk category | Rating | Where it bites | Mitigation |
|---|---|---|---|
| **Fabrication of commercial information** | HIGH | P02, P04, P05 | Explicit refusal paths; grounding constraints; 100% human review on customer-facing outputs |
| **Arithmetic unreliability** | HIGH | P05, P07, P08 | Show-your-working requirements; independent re-check by P07; production architecture moves calculation to spreadsheet formulas |
| **Regulatory compliance** | HIGH | P08 (tax invoices), P04 (Australian Consumer Law) | Required elements specified in prompt; finance review before release; finance retains document responsibility |
| **Data privacy** | MEDIUM–HIGH | P02, P06, P09 | Enterprise deployment with data residency and no-training guarantees; no consumer-tier tools; access-controlled register; retention limited to audit requirement (Privacy Act 1988, APP 6 and 8) |
| **Prompt injection** | MEDIUM | P01, P02 | Email content bounded as data, never instructions; unrecognised domains default to human review |
| **Over-reliance** | MEDIUM | All | Mandatory sampling; blocking failures require recorded sign-off; review steps defined as permanent controls, not pilot measures |
| **Model drift** | MEDIUM | All | Fixed regression set of 20 labelled emails re-run monthly against P01 and P02 |

### What this library does not do

An honest scope statement, in the spirit of distinguishing genuine AI capability from overpromise:

**These prompts do not read email and do not write to the spreadsheet.** A language model has no such access. Integration is performed by mail rules, a script, or an automation platform. The prompts are the *reasoning layer* inside that pipeline — they classify, extract, draft and check. Everything else is conventional software.

This distinction matters commercially: it means the implementation cost is integration work, not AI work, and it means the failure modes to plan for are the ones documented in each prompt entry rather than the ones implied by vendor marketing.

---

## Implementation

| Phase | Duration | Activity | Gate to proceed |
|---|---|---|---|
| **1 — Shadow** | Weeks 1–2 | Prompts run alongside the manual process. Outputs compared, not used. | Agreement on accuracy baseline |
| **2 — Assisted** | Weeks 3–6 | Outputs used as drafts. 100% human review on all ten prompts. | Error rate acceptable to ticketing head |
| **3 — Selective automation** | Week 7+ | P01, P02, P06, P09, P10 move to exception-only review. P03, P04, P05, P07, P08 retain full review permanently. | Governance sign-off |

Phase 3 is where the savings are realised. Phases 1 and 2 deliberately deliver none — they buy the evidence that the controls work.

### Governance

- **Named owner:** the ticketing head owns the library and all outputs produced through it.
- **Weekly:** 10% sample audit of automated classifications against source emails.
- **Monthly:** regression set re-run to detect model drift; results logged.
- **Month-end:** close is gated on zero P2 (audit integrity) exceptions from P09.
- **Quarterly:** prompt review against logged failures; versions updated in this repository.
- **AI use register:** all prompts, their data inputs and their deployment environment recorded, consistent with ISO/IEC 42001 and the NIST AI Risk Management Framework's govern and measure functions.

---

## Evidence of Iteration

Every prompt was developed across three versions, and the commit history of this repository is the version log.

| Version | Change made | Typical observed effect |
|---|---|---|
| **v1.0** | Single-line instruction, no structure | Inconsistent output; fabricated values; unusable downstream |
| **v1.1** | RACE Role and Context added; vocabularies fixed | Content correct; format and edge cases still failing |
| **v1.2** | Expectations schema, constraints, refusal paths, show-your-working | Machine-readable, chainable, auditable |

The most consequential lesson across all ten prompts: **a model needs a legitimate way to fail, or it will invent its way to an answer.** Fabrication was not fixed by better instructions about accuracy — it was fixed by giving the model a correct output for "the information is not present."

Full per-prompt iteration logs, including the exact v1.0 text and what went wrong with it, are in each prompt file.

---

## Repository Contents

| Path | Contents |
|---|---|
| `workflows/01-enquiry-intake/` | P01–P05, section README with chain diagram |
| `workflows/02-record-capture-audit/` | P06–P10, section README with register schema |
| `evidence/test-outputs/` | Captured prompt outputs from testing |

Each prompt file contains: the full RACE prompt text, intended workflow, problem being solved, automation potential with stated assumptions, a risk table with business impact and mitigation, and the complete v1.0 → v1.2 iteration log.

---

**Author:** Tarang Gupta  
**Submission:** BUS4005 Assessment 1
