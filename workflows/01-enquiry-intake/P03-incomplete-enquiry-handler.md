# P03 · Incomplete Enquiry Handler

**Workflow:** 01 — Enquiry Intake | **Step:** 3 of 5 | **Version:** v1.2 

## Prompt Text (v1.2 — RACE)

```
[ROLE]
You are a corporate ticketing coordinator at Capital Sports Group (CSG). You write
brief, courteous business emails to corporate clients. Your priority is getting a
complete enquiry back in one reply rather than a long exchange.

[ACTION]
Draft a reply requesting only the information listed in MISSING_FIELDS.
1. Open by acknowledging the client's enquiry and their organisation by name.
2. Request each missing field using the customer-facing wording in the mapping table.
3. Group all requests into a single short bulleted list.
4. Close by stating that a quotation follows within one business day of receipt.
5. Do not request any field that is not in MISSING_FIELDS.

[CONTEXT]
Senders are corporate buyers, travel agencies and community coordinators. They are
purchasing on behalf of an organisation, not personally.

Field mapping — internal name to customer-facing wording:
- fixture → the fixture or date you would like to attend
- ticket_quantity → the number of tickets required
- business_number → your ABN for invoicing
- contact_email → the best billing contact address
- organisation → your organisation's registered name

[EXPECTATION]
Return the email body only — no subject line, no signature block, no commentary.
Maximum 120 words. Tone: professional, warm, direct. Australian English spelling.
Do not quote prices, confirm availability, or promise seats. Do not apologise for
requesting information.

[MISSING_FIELDS]
{{MISSING_FIELDS}}

[CLIENT_DETAILS]
{{EXTRACTED_JSON}}
```

## Intended Workflow or Task
Step 3, triggered when P02 returns `extraction_complete: false`. Sends the enquiry back to the client to be completed before it can proceed to pricing. Its output re-enters the pipeline at P01 when the client replies.

## Problem Being Solved
Incomplete enquiries currently stall. Either the coordinator writes a one-off chase email, or the enquiry sits until someone notices it. Multiple partial chases across several emails extend the sales cycle by days on deals worth thousands of dollars, and slow response is a known driver of corporate buyers going elsewhere.

## Automation Potential
**High.** Drafting is fully automatable and the field list is supplied programmatically by P02. Human role is a single read-and-send approval.

*Assumption-based estimate:* if ~30% of enquiries arrive incomplete (~3/day) and each chase takes ~6 minutes to compose, that is ~18 min/day ≈ 75 hours annually, reducible by ~75%. The larger benefit is cycle-time, not labour: one complete chase replaces an average of two partial ones.

## Risks and Limitations

| Risk | Business impact | Mitigation |
|---|---|---|
| Requesting a field the client already supplied | Client perceives CSG as inattentive; damages credibility on a live deal | Prompt is strictly bounded to MISSING_FIELDS; accuracy depends wholly on P02 |
| Tone misjudged — chase reads as demanding or officious | Relationship damage with a high-value buyer | Explicit tone constraints; 120-word ceiling; coordinator approves before send |
| Model volunteers pricing or availability it has not been given | Unauthorised commercial commitment | Explicit prohibition in Expectation block; no inventory data supplied to this prompt |
| Cascading error from upstream extraction failure | Wrong information requested; enquiry stalls further | Human approval gate before every send |

**Overall: MEDIUM** — low technical risk, but this is customer-facing text on a live commercial opportunity, so no send is fully unattended.

## Iteration Log

**v1.0** — `Write a reply asking the customer for the information they left out.`
*Effect:* did not know which fields were required; invented some, omitted others. **Lesson:** the prompt needs the required-field list as an input, not as an assumption.

**v1.1** — Added Role and defined mandatory fields.
*Effect:* asked for the right things, but tone swung between curt and over-apologetic, and length varied from two lines to four paragraphs. **Lesson:** correct content does not guarantee usable output.

**v1.2** — Added the field-mapping table, 120-word limit, tone constraints, Australian English, and prohibitions on quoting price or availability.
*Effect:* consistent, send-ready drafts requiring minimal editing; internal field names no longer leak into customer-facing text. **Lesson:** mapping internal vocabulary to customer language is what makes an automated email sound human.
