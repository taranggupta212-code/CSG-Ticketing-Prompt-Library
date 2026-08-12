# P08 · Invoice Content Generation

**Workflow:** 02 — Record Capture & Audit | **Step:** 3 of 5 | **Version:** v1.2 

## Prompt Text (v1.2 — RACE)

```
[ROLE]
You are an accounts receivable officer at Capital Sports Group (CSG). You prepare the
content of tax invoices for corporate ticket sales. Tax invoices are regulated
documents, so you populate only supplied values and never originate a figure or a
reference number yourself.

[ACTION]
Generate the content fields for a tax invoice from the booking record below.
1. Copy the client, fixture, quantity, unit price and total exactly from BOOKING.
2. Use the invoice number supplied in INVOICE_REF. Do not generate one.
3. Show GST as the component already included in the total: GST = Total ÷ 11,
   rounded to 2 decimal places. State the calculation used.
4. Calculate the due date as the issue date plus 14 calendar days.
5. If BOOKING is missing any required field, return the refusal output.

[CONTEXT]
CSG is registered for GST. All ticket prices are GST-inclusive, so GST is shown as a
component of the total, never added to it. Australian tax invoices for amounts over
$1,000 must show the supplier's ABN, the recipient's identity, the date of issue, a
description of what is supplied, and the GST amount.

Supplier details:
- Capital Sports Group Pty Ltd
- ABN: 51 824 753 556
- Payment terms: 14 days from issue

Invoice numbers are issued sequentially by the finance system and supplied as input.
A model-generated reference would break sequence and create audit gaps.

[EXPECTATION]
Return only valid JSON. No preamble, no formatted invoice layout, no commentary.

{
  "invoice_number": <string, copied from INVOICE_REF>,
  "issue_date": <YYYY-MM-DD>,
  "due_date": <YYYY-MM-DD, issue date + 14 days>,
  "supplier_name": "Capital Sports Group Pty Ltd",
  "supplier_abn": "51 824 753 556",
  "client_name": <string>,
  "client_abn": <11-digit string>,
  "line_item_description": "<quantity> x corporate tickets — <fixture>",
  "quantity": <integer>,
  "unit_price_aud": <number, 2 decimal places>,
  "total_inc_gst_aud": <number, 2 decimal places>,
  "gst_component_aud": <number, 2 decimal places>,
  "gst_calculation_shown": "<total> / 11 = <gst>",
  "payment_terms": "Payment due within 14 days of issue date."
}

If any required field is missing from BOOKING, return:
{"status": "INCOMPLETE", "missing_fields": ["<field names>"]}

[BOOKING]
{{VALIDATED_ROW}}

[INVOICE_REF]
{{INVOICE_NUMBER}}

[ISSUE_DATE]
{{TODAY}}
```

## Intended Workflow or Task
Third step of Workflow 02, running only on rows that passed P07 validation. Produces the content fields that populate CSG's invoice template. The finance system supplies the sequential invoice number; the model never originates it.

## Problem Being Solved
Invoices are currently prepared by copying a previous invoice and overwriting fields, which propagates whatever error was in the source document and occasionally leaves a previous client's details in place. Invoice errors on corporate accounts delay payment, require reissue, and are visible to the client's finance team — a poor impression at the point where the relationship should be closing cleanly.

## Automation Potential
**High for content population; the document itself remains a finance-system function.** Human role is verification against the validated row before release.

*Assumption-based estimate:* ~10 minutes per invoice to prepare and check manually, ~4 invoices/day ≈ 40 min/day ≈ 170 hours annually. Expected reduction ~70%, retaining verification on every invoice because these are regulated documents.

## Risks and Limitations

| Risk | Business impact | Mitigation |
|---|---|---|
| **Incorrect GST treatment** — adding 10% rather than showing the included component | Client over-charged; non-compliant tax invoice; correction and reissue required | GST formula stated explicitly as ÷ 11; calculation shown in output for verification; finance review before release |
| Model-generated invoice numbers | Broken sequence, duplicate references, audit gaps in the receivables ledger | Invoice number supplied as input with an explicit do-not-generate instruction |
| Arithmetic error in the GST component | Understated or overstated tax liability | `gst_calculation_shown` field makes the arithmetic checkable at a glance; finance verifies before release |
| Stale client details carried across | Invoice issued to the wrong entity | Every field sourced from the current validated row; no template inheritance |
| Regulatory exposure — tax invoices are governed documents | Compliance risk if requirements are not met | Required elements listed in Context; the prompt produces content only, and finance retains responsibility for the issued document |

**Overall: HIGH** — a regulated financial document. Automation is appropriate for populating fields, but review before release should be presented to management as a permanent control rather than a pilot-phase measure.

## Iteration Log

**v1.0** — `Write an invoice for this booking.`
*Effect:* produced a plausible-looking invoice with an invented invoice number, no GST treatment and no payment terms. **Lesson:** the model will produce something invoice-shaped, and its confidence disguises the compliance gaps.

**v1.1** — Added Role and the full field list including GST.
*Effect:* fields correct, but invoice numbering was still invented rather than sequential, and GST was sometimes added to the total instead of shown as an included component — a direct over-charge. **Lesson:** naming a tax field does not communicate the tax treatment.

**v1.2** — Added the ÷ 11 formula with calculation shown, the supplied-invoice-number constraint, supplier details, due-date derivation, JSON schema, and the INCOMPLETE refusal path.
*Effect:* GST treatment correct in testing; sequence integrity preserved; the shown calculation lets finance verify in seconds. **Lesson:** where a rule is domain-specific, stating the rule is not enough — the prompt must state the exact operation.
