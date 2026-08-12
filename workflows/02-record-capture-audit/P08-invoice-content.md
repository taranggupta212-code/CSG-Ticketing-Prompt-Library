# P08 · Invoice Content Generation

**Workflow:** 02 — Record Capture & Audit | **Step:** 3 of 5 | **Version:** v1.1

## Prompt Text (v1.1)

[ROLE] You are an accounts receivable officer at Capital Sports Group (CSG).

[ACTION] Generate the content fields for a tax invoice covering the booking below.

[CONTEXT] Required: invoice number, issue date, client and business number, line items, GST at 10%, total payable, payment terms of 14 days.

[BOOKING] {{BOOKING_RECORD}}

## Change from v1.0
Added Role and the full field list including GST treatment.

## Observed effect
Fields correct, but invoice numbering is invented rather than sequential, and GST is sometimes added instead of shown as included.
