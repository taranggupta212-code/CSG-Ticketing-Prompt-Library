# P06 · Spreadsheet Row Generation

**Workflow:** 02 — Record Capture & Audit | **Step:** 1 of 5 | **Version:** v1.2 

## Prompt Text (v1.2 — RACE)

```
[ROLE]
You are a ticketing data administrator at Capital Sports Group (CSG). You convert
confirmed bookings into rows in the corporate sales register. This register is the
organisation's audit record, so you never alter, complete or improve a value — you
transcribe it exactly as supplied.

[ACTION]
Convert the booking record below into a single row for the corporate sales register.
1. Map each source field to its register column using the mapping in CONTEXT.
2. Apply the formatting rule for each column exactly as specified.
3. Where a source value is null, write NULL in that column. Do not infer a replacement.
4. Do not calculate the Total — copy the value supplied in the booking record.
5. Set requires_review to true if any column contains NULL.

[CONTEXT]
Register columns, in this exact order:
| Column | Source field | Format rule |
|---|---|---|
| Client Name | organisation | Registered organisation name, title case |
| Business Number | business_number | 11 digits, no spaces or punctuation |
| Fixture | fixture | Opponent name and date as DD/MM/YYYY |
| Tickets | ticket_quantity | Whole number, digits only, no separators |
| Unit Price | unit_price | Digits and decimal point only, 2 decimal places, no currency symbol |
| Total | total_payable | Digits and decimal point only, 2 decimal places, no currency symbol |

The register is stored in a shared spreadsheet retained for audit purposes.
Currency symbols and thousands separators break downstream spreadsheet formulas and
must never appear in numeric columns.

[EXPECTATION]
Return only valid JSON. No preamble, no commentary.

{
  "row": {
    "client_name": <string or "NULL">,
    "business_number": <string or "NULL">,
    "fixture": <string or "NULL">,
    "tickets": <string or "NULL">,
    "unit_price": <string or "NULL">,
    "total": <string or "NULL">
  },
  "requires_review": <true or false>,
  "review_reason": ["<column names containing NULL>"]
}

[BOOKING]
{{BOOKING_JSON}}
```

## Intended Workflow or Task
First step of Workflow 02, running once a quotation from P05 is accepted. Consumes the structured record produced by P02 and enriched through the enquiry workflow, and outputs the row appended to CSG's corporate sales register — the spreadsheet relied on during audit.

## Problem Being Solved
Booking details are currently re-typed into the register by hand after the deal closes, a second transcription of data already typed once during the enquiry. Beyond the duplicated effort, inconsistent manual formatting — currency symbols in some rows, spaces inside business numbers, mixed date formats — breaks spreadsheet formulas and makes the register difficult to search or reconcile during audit.

## Automation Potential
**Very High.** This is deterministic field mapping with no judgement involved. Human role reduces to reviewing rows flagged `requires_review`.

*Assumption-based estimate:* ~5 minutes per booking to enter and format manually, ~4 confirmed bookings/day ≈ 20 min/day ≈ 85 hours annually. Expected reduction ~85%. The larger benefit is register consistency, which reduces audit-preparation effort separately.

## Risks and Limitations

| Risk | Business impact | Mitigation |
|---|---|---|
| Silent field mis-mapping — correct-looking data in the wrong column | Corrupted audit record; error may go unnoticed for months | Explicit column-to-source mapping table; P07 validates every row before it is committed |
| Model reformats a value and changes its meaning (e.g. truncating a business number) | Invalid ABN on the audit record; invoicing and compliance failure | Transcribe-exactly instruction; format rules specify presentation only; P07 checks digit count |
| Model recalculates Total instead of copying it | Register total disagrees with the total quoted to the client | Explicit do-not-calculate instruction; P07 independently re-checks the arithmetic |
| NULL values accepted into the register without follow-up | Incomplete audit trail; gaps discovered only during audit | `requires_review` flag; incomplete rows queued for administrator completion before month-end close |
| Data privacy — the register concentrates client and commercial data in one file | Breach severity is high because the file aggregates every client | Access-controlled storage; enterprise AI deployment only; retention limited to the audit requirement |

**Overall: MEDIUM** — individually low-risk transcription, but this prompt writes to the organisation's permanent audit record, so errors are persistent rather than transient.

## Iteration Log

**v1.0** — `Turn this booking into a row for our spreadsheet.`
*Effect:* column order changed between runs; some fields merged, others omitted. Output could not be appended to a real sheet. **Lesson:** the model has no knowledge of the target schema unless it is given one.

**v1.1** — Added Role and a fixed column order.
*Effect:* column order correct, but date and currency formatting remained inconsistent — dollar signs and thousands separators appeared in numeric columns — and empty fields were silently filled with plausible values. **Lesson:** column order is only half a schema; each column needs a format rule.

**v1.2** — Added the per-column format rules, mandatory NULL handling, do-not-calculate instruction, JSON output, and the `requires_review` flag.
*Effect:* rows now paste directly into the register without cleanup; formulas no longer break; incomplete records surface themselves instead of being quietly invented. **Lesson:** for data destined for a spreadsheet, formatting constraints are not cosmetic — they determine whether the output is usable at all.
