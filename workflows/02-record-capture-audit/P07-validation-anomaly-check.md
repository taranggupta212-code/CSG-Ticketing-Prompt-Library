# P07 · Validation & Anomaly Check

**Workflow:** 02 — Record Capture & Audit | **Step:** 2 of 5 | **Version:** v1.2 
## Prompt Text (v1.2 — RACE)

```
[ROLE]
You are a ticketing operations auditor at Capital Sports Group (CSG). You review
register rows before they are committed. You report what you find without correcting
it — your role is detection, not repair, because silent corrections would defeat the
purpose of the audit control.

[ACTION]
Run every check in the CHECK LIST against the row below.
1. Perform each check independently and record a PASS or FAIL for each.
2. For the arithmetic check, state your own calculation of Tickets × Unit Price and
   compare it to the Total supplied. Report both figures.
3. Assign each failure a severity using the SEVERITY SCALE.
4. Do not modify any value in the row. Report only.
5. Set overall_status to FAIL if any check of CRITICAL or HIGH severity fails.

[CONTEXT]
CHECK LIST:
- C1 Arithmetic: Total must equal Tickets × Unit Price (tolerance: $0.01 rounding)
- C2 Business number: must be exactly 11 digits, no letters or punctuation
- C3 Completeness: no column may contain NULL or be empty
- C4 Quantity plausibility: Tickets must be between 20 and 5000 inclusive
- C5 Price plausibility: Unit Price must be between 15.00 and 2000.00
- C6 Duplicate risk: flag if the same Client Name and Fixture appear in PRIOR_ROWS
- C7 Format: numeric columns must contain no currency symbols or thousands separators

SEVERITY SCALE:
- CRITICAL — the row is factually wrong and must not be committed (C1, C2)
- HIGH — the row is incomplete or implausible and needs human confirmation (C3, C4, C5)
- MEDIUM — the row may be a duplicate and needs checking (C6)
- LOW — presentation issue only, correctable without re-contacting the client (C7)

CSG sells corporate blocks of 20+ tickets. Bands outside the plausibility ranges are
not necessarily wrong, but always warrant human confirmation.

[EXPECTATION]
Return only valid JSON. No preamble, no corrected version of the row.

{
  "overall_status": "PASS" | "FAIL",
  "checks": [
    {"id": "C1", "result": "PASS" | "FAIL", "severity": <string or null>,
     "detail": "<what was found, including both figures for C1>"}
  ],
  "blocking_failures": ["<check ids of CRITICAL or HIGH severity>"],
  "recommended_action": "<one sentence for the reviewing administrator>"
}

[ROW]
{{ROW_DATA}}

[PRIOR_ROWS]
{{RECENT_REGISTER_ROWS}}
```

## Intended Workflow or Task
Second step of Workflow 02, running immediately after P06 and before any row is committed to the register. Functions as the automated control gate on the audit record: rows returning FAIL are held in an exception queue for administrator resolution rather than being written.

## Problem Being Solved
There is currently no systematic check between data entry and the audit record — errors are discovered during audit preparation or when a client disputes an invoice, both of which are the most expensive points at which to find them. A mistyped business number or a total that disagrees with the quotation requires re-contacting the client and reissuing documents, damaging credibility on a closed deal.

## Automation Potential
**Very High for detection; the human decides what to do about it.** Every check is rule-based and runs in seconds against every row, which is not feasible manually at volume. The administrator resolves flagged exceptions.

*Assumption-based estimate:* manual checking is currently ad hoc, so the honest framing is error-cost avoidance rather than hours saved. If ~5% of rows contain an error and each error costs ~45 minutes to trace and correct after the fact, catching them at entry avoids ~40 hours annually — and removes the audit exposure entirely, which management should weight more heavily than the hours.

## Risks and Limitations

| Risk | Business impact | Mitigation |
|---|---|---|
| **The model is not a reliable calculator** — it may confirm an incorrect total as correct | A validation control that itself fails is worse than no control, because it creates false assurance | Model must show its own calculation, making the check auditable; at production scale C1 should be a spreadsheet formula, not a prompt. This limitation should be stated openly to management |
| Plausibility bands reject legitimate unusual bookings | A large genuine order is delayed by an unnecessary review | Bands set deliberately wide; flags request confirmation rather than rejection |
| Duplicate detection depends on the quality of PRIOR_ROWS supplied | Genuine duplicate missed, or repeat business wrongly flagged | Flagged as MEDIUM only; never blocks a row; monthly manual duplicate sweep retained |
| False confidence in an automated gate | Administrators stop reading flagged detail and simply clear the queue | Blocking failures require explicit administrator sign-off with a recorded reason |
| Checks cannot detect a value that is wrong but well-formed (e.g. correct-format ABN belonging to a different company) | Plausible bad data passes into the audit record | Acknowledged limitation — format validation is not identity verification; periodic sample verification against source emails |

**Overall: MEDIUM** — the control adds real value, but its most important check depends on a capability language models are weak at. Presenting it as a first-pass filter rather than a guarantee is the honest position.

## Iteration Log

**v1.0** — `Check this data for errors.`
*Effect:* returned generic reassurance ("this looks correct") without performing any specific check, including on rows containing deliberate arithmetic errors. **Lesson:** without defined criteria, the model reports an impression rather than a result.

**v1.1** — Added Role and three explicit validation rules.
*Effect:* caught arithmetic errors reliably, but returned a prose paragraph, so a human had to read it to determine whether anything had failed — defeating the purpose of an automated gate. **Lesson:** a control must produce a machine-readable verdict, not a description.

**v1.2** — Added the seven-check list, severity scale, show-your-calculation requirement, JSON output, and the report-don't-repair instruction.
*Effect:* produces a binary gate decision with an auditable trail of what was checked and why it failed; severity lets the administrator triage the exception queue by risk. **Lesson:** requiring the model to show its arithmetic converts an unverifiable claim into evidence a human can check in seconds.
