# P10 · Demand & Conversion Report

**Workflow:** 02 — Record Capture & Audit | **Step:** 5 of 5 | **Version:** v1.2 

## Prompt Text (v1.2 — RACE)

```
[ROLE]
You are a ticketing operations analyst at Capital Sports Group (CSG). You report
corporate sales performance to the ticketing head. You distinguish clearly between
what the data shows and what it might mean, and you never present an inference as a
finding.

[ACTION]
Produce the weekly corporate demand and conversion report.
1. Calculate each metric in the METRICS list using the stated formula.
2. State the figure for the current week and the change from the previous week.
3. Where the previous week's figure is unavailable, state "no comparison available"
   rather than omitting the metric.
4. Identify the two fixtures with the highest enquiry volume.
5. Provide at most three observations. Each must reference a specific figure from
   the report. Label any causal statement as a hypothesis.
6. Do not recommend actions. The ticketing head decides those.

[CONTEXT]
METRICS:
- Enquiry volume: count of records with an enquiry logged this week
- Conversion rate: (bookings confirmed ÷ enquiries received) × 100, to 1 decimal place
- Average order value: total revenue ÷ number of confirmed bookings, in AUD
- Tickets sold: sum of the Tickets column across confirmed bookings
- Average response time: mean hours from enquiry received to first reply sent

Conversion is measured against enquiries received in the same week, which understates
performance where an enquiry converts in a later week. State this limitation whenever
conversion rate is reported.

CSG's corporate desk handles bulk bookings of 20+ tickets for home fixtures.

[EXPECTATION]
Return a plain-text report, maximum 250 words, in this structure:

CORPORATE TICKETING — WEEK ENDING {{WEEK_ENDING}}

METRICS
| Metric | This week | Last week | Change |
(one row per metric; write "n/a" where no comparison exists)

TOP FIXTURES BY ENQUIRY VOLUME
1. <fixture> — <count> enquiries
2. <fixture> — <count> enquiries

OBSERVATIONS
- <observation referencing a specific figure>

DATA LIMITATIONS
- Conversion rate measured within-week; enquiries converting later are not captured.
- <any metric affected by missing data this week>

No recommendations. No commentary beyond the observations section.

[RECORDS]
{{WEEKLY_RECORDS}}

[PRIOR_WEEK]
{{PREVIOUS_WEEK_METRICS}}
```

## Intended Workflow or Task
Final step of Workflow 02, running weekly across the register. Where P09 reports what is broken, P10 reports how the desk is performing. Output goes to the ticketing head and informs pricing and inventory-release decisions for upcoming fixtures.

## Problem Being Solved
CSG currently has no regular visibility of corporate demand. Enquiry volume, conversion and response time exist in the register but are never aggregated, so decisions about pricing and how many seats to release to corporate blocks are made on intuition. Because the register now captures this data consistently through P06, reporting it costs almost nothing — the information is already there and simply unused.

## Automation Potential
**Very High.** Calculation and formatting are fully automatable. Interpretation and any resulting decision remain with the ticketing head, which is why the prompt is explicitly prohibited from recommending actions.

*Assumption-based estimate:* this reporting does not currently happen, so the honest framing is capability created rather than hours saved. Producing it manually would cost ~2 hours weekly ≈ 100 hours annually — labour CSG would not spend, meaning the realistic benefit is the decision quality the report enables, not a payroll saving. **This distinction should be stated plainly in the business case rather than counted as savings.**

## Risks and Limitations

| Risk | Business impact | Mitigation |
|---|---|---|
| **Calculation error presented in an authoritative format** — a report that looks precise is trusted without checking | Pricing or inventory decisions made on wrong figures | Metrics should be computed by spreadsheet formula at production scale, with the model formatting rather than calculating; this limitation stated openly to management |
| Spurious causal inference from small weekly samples | A one-week fluctuation drives a pricing change | Causal statements must be labelled hypotheses; observations capped at three and must cite a figure |
| Within-week conversion measurement understates true performance | Corporate desk performance judged unfairly | Limitation stated in every report by mandatory structure |
| Metrics become targets and distort behaviour | Response time optimised at the expense of response quality | Report presented alongside P09 exceptions so quality issues remain visible |
| Model volunteers recommendations beyond the data | Analysis mistaken for advice; accountability blurred | Explicit prohibition; the ticketing head owns decisions |

**Overall: MEDIUM** — no customer contact and no financial document, but the output influences commercial decisions, and a well-formatted report carries more authority than its underlying accuracy warrants. That gap is the real risk.

## Iteration Log

**v1.0** — `Make a report on ticket demand.`
*Effect:* invented metrics that varied between runs, covered an unspecified period, and included confident recommendations unsupported by the data. **Lesson:** an undefined analytical task produces confident-sounding output with no consistent basis.

**v1.1** — Added Role, named audience and four defined metrics.
*Effect:* metrics calculated correctly, but the output read as an undifferentiated data dump with no comparison period and no length limit, and still drifted into causal claims stated as fact. **Lesson:** defining metrics fixes the numbers but not the reasoning around them.

**v1.2** — Added formulas, week-on-week comparison, the mandatory data-limitations section, the three-observation cap with figure citation, hypothesis labelling, the 250-word ceiling, and the no-recommendations prohibition.
*Effect:* report is readable in a minute; the limitations section prevents the within-week conversion measure being over-read; separating observation from recommendation keeps the decision with the human. **Lesson:** forcing a model to state what its data cannot show is the most effective guard against over-interpretation.
