# P09 · Weekly Reconciliation Summary

**Workflow:** 02 — Record Capture & Audit | **Step:** 4 of 5 | **Version:** v1.2 

## Prompt Text (v1.2 — RACE)

```
[ROLE]
You are a ticketing operations auditor at Capital Sports Group (CSG). You produce the
weekly exception report for the ticketing head. Your reader has limited time and needs
to know what to act on, in what order — not what happened.

[ACTION]
Review the week's register records and report exceptions only.
1. Identify every record matching an exception type in CONTEXT.
2. Assign each a priority using the PRIORITY SCALE.
3. Order the report by priority, highest first.
4. For each exception, name the specific record, state what is wrong, and state the
   single action required to resolve it.
5. Report the count of clean records as a single figure. Do not list them.
6. If there are no exceptions, say so in one line and return nothing further.

[CONTEXT]
EXCEPTION TYPES:
- E1 Incomplete record: any column containing NULL
- E2 Arithmetic mismatch: Total does not equal Tickets × Unit Price
- E3 Unresolved enquiry: enquiry logged with no recorded outcome after 5 business days
- E4 Possible duplicate: same client and fixture appearing more than once
- E5 Invoice not issued: booking confirmed with no corresponding invoice reference
- E6 Overdue payment: invoice issued more than 14 days ago, unpaid

PRIORITY SCALE:
- P1 Revenue at risk — E5, E6
- P2 Audit integrity — E1, E2
- P3 Follow-up required — E3, E4

CSG's register is the audit record. Month-end close requires zero P2 exceptions.

[EXPECTATION]
Return a plain-text report, maximum 300 words, in this structure:

WEEKLY RECONCILIATION — WEEK ENDING {{WEEK_ENDING}}

SUMMARY: <count> records reviewed. <count> clean. <count> exceptions requiring action.

P1 — REVENUE AT RISK
- <client / reference>: <what is wrong>. Action: <what to do>.

P2 — AUDIT INTEGRITY
- <client / reference>: <what is wrong>. Action: <what to do>.

P3 — FOLLOW-UP REQUIRED
- <client / reference>: <what is wrong>. Action: <what to do>.

Omit any priority heading with no exceptions. Do not include commentary,
interpretation, or congratulation. Do not suggest process improvements.

[RECORDS]
{{WEEKLY_RECORDS}}
```

## Intended Workflow or Task
Runs weekly across the full register rather than per-record. Produces the exception report the ticketing head reviews to clear issues before month-end close, and provides the standing evidence that the audit record is being actively maintained.

## Problem Being Solved
Register problems currently surface at month-end or during audit preparation, by which time an unissued invoice may be weeks old and an incomplete record may no longer be reconstructable from memory. Weekly detection converts a compounding backlog into a short, actionable list, and materially reduces audit-preparation effort.

## Automation Potential
**Very High.** Scanning every record against six rules is not realistically done manually at volume, so this prompt enables a control that does not currently exist rather than merely replacing one. Human role is acting on the report.

*Assumption-based estimate:* framed honestly, the saving is in avoided month-end remediation — if reconciliation currently consumes ~4 hours at each month-end plus ad hoc chasing, weekly detection reduces that by ~60%, roughly 30 hours annually. The stronger argument to management is earlier detection of revenue at risk, not the hours.

## Risks and Limitations

| Risk | Business impact | Mitigation |
|---|---|---|
| Missed exception — the model overlooks a record matching a rule | False assurance that the register is clean; the exception surfaces at audit instead | Rules are also enforced at row level by P07, so this is a second layer rather than a sole control; monthly full manual reconciliation retained |
| Model summarises rather than enumerates when record volume is high | Individual exceptions disappear into an aggregate statement | Explicit per-record output structure; report cross-checked against the count of P07 flags raised that week |
| Priority misassignment | Revenue-at-risk item actioned after a formatting issue | Priority mapping is fixed and rule-based, not left to model judgement |
| Report becomes routine and stops being read | Detection without action delivers nothing | Report addressed to a named role with month-end close gated on zero P2 exceptions |
| Aggregated client data in a single output | Higher breach severity than any individual record | Enterprise deployment; report distribution limited to the ticketing head and finance |

**Overall: MEDIUM** — a detective control operating on data already validated upstream. Its value depends entirely on someone acting on it, which is a governance question rather than a technical one.

## Iteration Log

**v1.0** — `Summarise this week's bookings.`
*Effect:* produced a readable narrative of the week's activity that restated the data and mentioned no problems at all, including on records with deliberate errors. **Lesson:** summarising and auditing are opposite tasks; asking for one does not deliver the other.

**v1.1** — Added Role and redirected the task from summarising to exception reporting.
*Effect:* began surfacing genuine problems, but listed them in the order encountered with no severity, so an unissued invoice appeared below a formatting issue. Length was unbounded. **Lesson:** an unranked exception list transfers the triage work back to the reader.

**v1.2** — Added the six defined exception types, the three-tier priority scale, the fixed report structure, the one-action-per-exception requirement, the 300-word ceiling, and the instruction to suppress commentary.
*Effect:* report is scannable in under a minute and each line states what to do; clean records collapse to a single count. **Lesson:** for a management-facing output, what you exclude matters as much as what you include.
