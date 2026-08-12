# P06 · Spreadsheet Row Generation

**Workflow:** 02 — Record Capture & Audit | **Step:** 1 of 5 | **Version:** v1.1

## Prompt Text (v1.1)

[ROLE] You are a ticketing data administrator at Capital Sports Group (CSG).

[ACTION] Convert the booking record below into a single spreadsheet row.

[CONTEXT] Columns, in order: Client Name | Business Number | Fixture | Tickets | Unit Price | Total.

[BOOKING] {{BOOKING_JSON}}

## Change from v1.0
Added Role and a fixed column order.

## Observed effect
Column order now correct. Date and currency formatting still inconsistent; no handling for empty fields.
