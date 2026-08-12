# P04 · Availability & Pricing Response

**Workflow:** 01 — Enquiry Intake | **Step:** 4 of 5 | **Version:** v1.2 

## Prompt Text (v1.2 — RACE)

```
[ROLE]
You are a corporate ticketing coordinator at Capital Sports Group (CSG). You respond to
bulk ticket enquiries. You are commercially careful: you state only what the supplied
inventory record confirms, because every figure you write may be relied upon by the
client.

[ACTION]
Draft a reply to the enquiry below.
1. Confirm the fixture and quantity the client requested.
2. State availability using ONLY the INVENTORY block.
3. State pricing using ONLY the INVENTORY block.
4. If the requested quantity exceeds available seats, state the maximum available and
   offer the next fixture listed in INVENTORY.
5. If INVENTORY does not contain the requested fixture, or contains no price, do NOT
   estimate. Return the refusal output defined in EXPECTATION.
6. Close with the next step and a validity period of 5 business days.

[CONTEXT]
CSG sells corporate blocks of 20+ seats at a 40,000-seat venue. Prices vary by seating
category and fixture demand. All prices in AUD and inclusive of GST.
Availability changes continuously; the INVENTORY block is a point-in-time snapshot.

[EXPECTATION]
Return the email body only. Maximum 180 words. Australian English. Professional and
concise. Every number you write must appear in the INVENTORY block — do not calculate,
round or estimate any figure that is not supplied.

If required inventory data is absent, return exactly:
ESCALATE_TO_HUMAN: <the specific data that is missing>

[INVENTORY]
{{INVENTORY_DATA}}

[ENQUIRY]
{{EXTRACTED_JSON}}
```

## Intended Workflow or Task
Step 4, running on complete enquiries classified NEW_ENQUIRY. Produces the substantive first reply to the client. Where the client accepts, the enquiry proceeds to P05 for formal quotation.

## Problem Being Solved
This is the response corporate buyers are waiting for, and it is currently the slowest step — the coordinator must check inventory, look up pricing, and compose a reply. Delay here directly costs revenue: bulk buyers approach several venues and events simultaneously, and slow responses lose bookings to faster competitors.

## Automation Potential
**Medium–High.** Drafting is automatable; the commercial content is not, because it depends on live inventory the model cannot see. Human approval is mandatory before send — this message contains prices the client may treat as binding.

*Assumption-based estimate:* ~12 minutes per response manually (lookup plus composition), ~7 enquiries/day ≈ 84 min/day ≈ 350 hours annually. Realistic reduction ~60% — lower than other prompts because the human review step is substantive, not cursory.

## Risks and Limitations

| Risk | Business impact | Mitigation |
|---|---|---|
| **Hallucinated pricing or availability** — the highest-severity risk in the library | A quoted price CSG cannot honour; potential misleading-conduct exposure under Australian Consumer Law | Hard grounding constraint; explicit ESCALATE_TO_HUMAN refusal path; mandatory human approval before every send |
| Stale inventory snapshot | Seats sold twice, or a genuine sale declined | Inventory refreshed immediately before prompt runs; validity period of 5 days stated in every reply |
| Over-reliance after a period of accurate output | Reviewer approves without checking figures | Coordinator verifies every price against the live system; 100% review, never sampled |
| Tone reads as automated | Weakens a relationship-driven B2B sale | Word limit and tone constraints; coordinator personalises before send |

**Overall: HIGH** — the only prompt in the library that states binding commercial terms to a customer. Full human review is non-negotiable and should be presented as a permanent control, not a pilot-phase safeguard.

## Iteration Log

**v1.0** — `Reply to this customer about ticket availability and prices.`
*Effect:* invented both availability and prices — fluent, confident and entirely fabricated. **Lesson:** fluency is not accuracy; this failure mode is invisible to a hurried reader.

**v1.1** — Added Role and an explicit inventory input.
*Effect:* major improvement when data was complete, but the model still filled gaps with plausible invented prices when inventory was partial. **Lesson:** supplying data does not stop fabrication; the model needs an instruction for what to do when data is absent.

**v1.2** — Added the every-number-must-appear-in-INVENTORY constraint, the ESCALATE_TO_HUMAN refusal path, over-capacity handling, and the validity period.
*Effect:* fabrication eliminated in testing; incomplete inventory now produces a visible escalation instead of a confident invention. **Lesson:** a model needs a legitimate way to fail, or it will invent its way to an answer.
