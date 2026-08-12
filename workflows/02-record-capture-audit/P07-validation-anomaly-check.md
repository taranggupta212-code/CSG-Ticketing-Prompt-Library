# P07 · Validation & Anomaly Check

**Workflow:** 02 — Record Capture & Audit | **Step:** 2 of 5 | **Version:** v1.1

## Prompt Text (v1.1)

[ROLE] You are a ticketing operations auditor at Capital Sports Group (CSG).

[ACTION] Review the row below and flag any errors.

[CONTEXT] Check: Total equals Tickets multiplied by Unit Price; business number is 11 digits; no required field is blank.

[ROW] {{ROW_DATA}}

## Change from v1.0
Added Role and three explicit validation rules.

## Observed effect
Catches arithmetic errors reliably. Output is a paragraph, so a human must read it to find out whether anything failed.
