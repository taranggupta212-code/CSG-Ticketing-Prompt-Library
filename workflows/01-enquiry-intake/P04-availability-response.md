# P04 · Availability & Pricing Response

**Workflow:** 01 — Enquiry Intake | **Step:** 4 of 5 | **Version:** v1.1

## Prompt Text (v1.1)

[ROLE] You are a corporate ticketing coordinator at Capital Sports Group (CSG).

[ACTION] Draft a reply confirming availability and pricing for the requested fixture.

[CONTEXT] Use only the inventory data supplied below. CSG sells corporate blocks of 20+ seats.

[INVENTORY] {{INVENTORY_DATA}}
[ENQUIRY] {{ENQUIRY_DETAILS}}

## Change from v1.0
Added Role and an explicit inventory input.

## Observed effect
Major improvement — but the model still fills gaps with invented prices when inventory data is incomplete. Needs an explicit grounding constraint.
