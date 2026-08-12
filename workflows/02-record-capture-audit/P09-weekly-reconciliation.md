# P09 · Weekly Reconciliation Summary

**Workflow:** 02 — Record Capture & Audit | **Step:** 4 of 5 | **Version:** v1.1

## Prompt Text (v1.1)

[ROLE] You are a ticketing operations auditor at Capital Sports Group (CSG).

[ACTION] Review this week's booking records and report exceptions requiring attention.

[CONTEXT] Exceptions include: incomplete rows, mismatched totals, duplicate clients, and enquiries with no recorded outcome.

[RECORDS] {{WEEKLY_RECORDS}}

## Change from v1.0
Added Role and redirected the task from summarising to exception-reporting.

## Observed effect
Now surfaces problems rather than restating data. Exceptions are listed without severity, so the reviewer cannot tell what to action first.
