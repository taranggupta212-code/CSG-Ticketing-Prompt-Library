# P05 · Formal Bulk Quotation

**Workflow:** 01 — Enquiry Intake | **Step:** 5 of 5 | **Version:** v1.2 

## Prompt Text (v1.2 — RACE)

```
[ROLE]
You are a corporate sales executive at Capital Sports Group (CSG). You issue formal
quotations for bulk ticket purchases. Quotations are commercial documents that clients
may act on, so structure and arithmetic accuracy matter more than style.

[ACTION]
Produce a formal quotation from the booking details below.
1. Reproduce every figure from BOOKING exactly as supplied.
2. Calculate the line total as unit price × quantity, then state the calculation shown.
3. Verify that your stated total equals your own calculation before returning output.
4. Apply the bulk discount band from CONTEXT that matches the ticket quantity.
5. State the validity period as 5 business days from today's date.
6. Include the standard terms listed in CONTEXT verbatim.

[CONTEXT]
CSG bulk discount bands:
- 20–49 tickets: no discount
- 50–99 tickets: 5% discount
- 100+ tickets: 10% discount

Standard terms (include verbatim):
- Payment due within 14 days of invoice date.
- Seats are allocated on receipt of payment, not on issue of this quotation.
- Quotation superseded by any subsequent written quotation for the same fixture.

All prices in AUD, inclusive of GST.

[EXPECTATION]
Return the quotation in exactly this structure, with these headings:

QUOTATION REFERENCE: {{QUOTE_REF}}
DATE: {{TODAY}}
VALID UNTIL: <5 business days from date>

CLIENT: <organisation name>
ABN: <business number>

FIXTURE: <fixture>
SEATING CATEGORY: <category>
QUANTITY: <number> tickets
UNIT PRICE: $<amount> AUD
SUBTOTAL: $<amount> AUD
BULK DISCOUNT: <percentage>% (−$<amount>)
TOTAL PAYABLE: $<amount> AUD

TERMS:
<the three standard terms>

NEXT STEP: <one sentence>

If any required figure is missing from BOOKING, return:
INCOMPLETE_QUOTATION: <the missing fields>

[BOOKING]
{{BOOKING_DETAILS}}
```

## Intended Workflow or Task
Final step of the enquiry workflow, running once a client indicates intent to proceed after P04. Output is the formal document the client signs off internally to authorise purchase. On acceptance, the record passes to Workflow 02 for capture and invoicing.

## Problem Being Solved
Quotations are currently assembled by hand from a previous quotation, which propagates formatting inconsistency and, occasionally, arithmetic and discount-band errors. An incorrect discount is a direct revenue loss if under-charged, and a credibility problem with a corporate buyer if over-charged. Inconsistent quotation formats also complicate audit.

## Automation Potential
**High.** Structure, terms and discount logic are fully rule-based. Human role is verifying figures against the source enquiry and approving release.

*Assumption-based estimate:* ~15 minutes per quotation manually, ~4 quotations/day ≈ 60 min/day ≈ 250 hours annually. Expected reduction ~70%, retaining a verification step on every document.

## Risks and Limitations

| Risk | Business impact | Mitigation |
|---|---|---|
| **Arithmetic error** — language models generate plausible numbers rather than computing them reliably | Under-charging loses revenue; over-charging damages a live deal | Self-verification step in Action; P07 independently re-checks totals; deterministic calculation should replace this step at production scale |
| Wrong discount band applied at boundary quantities (49/50, 99/100) | Margin loss or client dispute | Bands stated explicitly with inclusive ranges; reviewer checks band against quantity |
| Terms silently reworded rather than reproduced | Contractual ambiguity; weakened legal position | "Include verbatim" instruction; reviewer compares against master terms |
| Quotation issued on incomplete data | Client receives a document CSG cannot honour | INCOMPLETE_QUOTATION refusal path |

**Overall: HIGH** — a commercial document with a binding total. The arithmetic limitation is worth stating openly to management: prompting mitigates it but does not solve it, and the correct long-term fix is calculating totals in the spreadsheet rather than in the model.

## Iteration Log

**v1.0** — `Write a quote for these tickets.`
*Effect:* prose paragraph with no structure, no terms and no validity period. Unusable as a commercial document. **Lesson:** "quote" carries an assumed format the model does not share.

**v1.1** — Added Role and the required quotation components.
*Effect:* all components present, but layout differed on every run and totals were occasionally miscalculated. **Lesson:** listing required content does not fix presentation or arithmetic.

**v1.2** — Added the fixed output template, explicit discount bands, verbatim terms, self-verification step, and INCOMPLETE_QUOTATION refusal path.
*Effect:* identical structure across runs; discount application correct in testing; arithmetic errors reduced but not eliminated — which is why P07 independently re-validates every total. **Lesson:** prompting can constrain a known weakness but should not be the only control over it.
