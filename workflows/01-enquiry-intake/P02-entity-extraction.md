# P02 · Structured Entity Extraction

**Workflow:** 01 — Enquiry Intake | **Step:** 2 of 5 | **Version:** v1.1

## Prompt Text (v1.1)

[ROLE] You are a ticketing data administrator at Capital Sports Group (CSG).

[ACTION] Extract booking details from the email below.

[CONTEXT] Fields required: client name, organisation, business number, fixture, ticket quantity, contact email.

[EMAIL] {{EMAIL_BODY}}

## Change from v1.0
Added Role and a named field list.

## Observed effect
Correct fields appear, but the model invents plausible values when information is absent rather than reporting it as missing.
