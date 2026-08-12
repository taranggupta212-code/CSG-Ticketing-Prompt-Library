# P01 · Email Intent Classification & Routing

**Workflow:** 01 — Enquiry Intake | **Step:** 1 of 5 | **Version:** v1.1

## Prompt Text (v1.1)

[ROLE] You are a corporate ticketing operations analyst at Capital Sports Group (CSG), a professional T20 cricket franchise.

[ACTION] Classify the inbound email below into one category.

[CONTEXT] CSG's bulk desk handles corporate enquiries for home fixtures at a 40,000-seat venue. Categories: NEW_ENQUIRY, BOOKING_CONFIRMATION, AMENDMENT, PAYMENT_QUERY, COMPLAINT, OUT_OF_SCOPE.

[EMAIL] {{EMAIL_BODY}}

## Change from v1.0
Added Role and Context; replaced open-ended categorisation with a fixed six-item list.

## Observed effect
Categories now stable across runs. Still fails on emails containing two requests, and still returns conversational prose instead of usable output.
