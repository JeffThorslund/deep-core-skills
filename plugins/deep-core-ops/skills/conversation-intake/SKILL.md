---
name: conversation-intake
description: Use right after a call — a pilot check-in, a customer discovery call, or a Vesper Alpha advisory call — to turn the transcript into Conversational Insights records. Trigger phrases: "log this call", "intake this call", "process the transcript", "I just finished a pilot/discovery/advisory call". Asks for type, date, and transcript; extracts requests (pilot calls only), customer insights, and advisory insights (advisory calls only), all deduplicated; writes to the Notion Conversations, Customer Requests, Customer Insights, and Advisory Insights tables only after I confirm.
---

# Conversation Intake

## What this does
Turns one finished call into clean records across the Conversational Insights tables: a Conversation log row, the People/Account it involved, any Requests (pilot calls), and any Insights — Customer Insights for customer calls, Advisory Insights for Vesper Alpha advisory calls — deduplicated against what's already there, and written only after you approve.

Three call families: **customer calls** (pilot check-in / ad hoc / discovery) produce Customer Insights and, for pilots, Requests. **Advisory calls** — only the Vesper Alpha partners (Adam, Dan, Salar) — produce Advisory Insights only; advisors don't file feature Requests, and other advisors (e.g. Marni) stay out of this system.

## Two ways to run
**Single call** — I just finished one call and want it logged. Start at Step 1.

**Catch-up sweep** — "catch up the insights", "sweep since last time", "log all the recent calls". Run Step 0 first, then loop Steps 2–6 over each in-scope call, and present **one combined plan** for all of them before writing.

## Step 0 — Find the watermark and sweep (catch-up runs only)
- **Watermark:** the most recent `Date` already in the **Conversations** table is the last time this ran. Read it — don't re-log anything on or before it that's already present.
- **Sweep Attio:** list every call recording with `starts_after` = the watermark (Attio call-recordings-by-metadata / search-meetings). Each Attio call already in Conversations (match by party + date) is skipped.
- **Sort every remaining call into one of three buckets — file it where it belongs:**
	- **Customer call** → Conversations + Customer Insights (+ Requests if it's a paying pilot). Pilot check-ins, ad hoc pilot calls, and prospect discovery calls.
	- **Advisory call** → Conversations + Advisory Insights only. *Only* the Vesper Alpha partners (Adam, Dan, Salar).
	- **Out of scope — do not log:** internal team calls (standups, planning), investor/fundraising calls, candidate/recruiting screens, grant calls (e.g. IRAP), and partner/advisor calls that aren't Vesper Alpha. Name these in the plan with a one-line reason so the sort is visible, but write nothing for them.
- Accounts are referred to by the same name everywhere (Notion Accounts, Attio, meeting titles) — match on it. When a call's bucket is genuinely ambiguous (an unlabelled 1:1, an unfamiliar name), surface it in the plan as *unclassified* and let me decide rather than guessing.

## Step 1 — Ask me about the call
Before reading anything, ask these in one message and wait for the answers:
1. **Type** — Pilot check-in, Ad hoc, Customer discovery call, or Advisory call (Vesper Alpha)?
2. **Attio link** — for the account or person, if handy (optional).

Do nothing else until I answer.

## Step 2 — Work out who and where
- Pull participant names from the transcript. Match each against **People**; flag anyone new to create.
- Determine the **Account**: pilot calls have one. For discovery calls, default to **person-only** — do **not** create an Account just because the person has an employer/company in Attio (e.g. a consultant's firm, or any org that isn't itself the customer). Only create an Account for an actual pilot/customer, or a prospect org being actively pursued as a pilot. When in doubt, stay person-only and flag it. Match against **Accounts**; flag only if genuinely new.
- Don't create anything yet — just show the matches and proposed new rows.

## Step 3 — Extract
- **Customer Insights** (customer calls): distinct learnings — pain, workflow, willingness to pay, buying process, segment/positioning, competitive. Phrase each as a short claim and tag a Type.
- **Advisory Insights** (advisory calls only): distinct pieces of advice/guidance. Phrase each as a short claim and tag a Type (Fundraising / Valuation / Round strategy / Cap table / Governance / advisory / GTM / commercial / Product / strategy / Market validation / Team / hiring / Other). No Requests on advisory calls — skip them entirely.
- **Requests** (pilot calls only — skip for discovery and advisory): explicit asks. Phrase imperatively, e.g. "Auto-detect UTM zone on import". Tag Category (Product / Bug / Data / Commercial / Docs / Other).

## Step 4 — Dedup (propose, never merge silently)
- **Requests:** for each ask, check existing Requests for this Account with Status ≠ Declined. If an equivalent ask exists, propose linking this call into its `Mentioned in` (bumps Mention count) instead of a new row. Otherwise a new row, Status = Captured.
- **Customer Insights:** for each learning, check existing Insights for this Person of the same Type. Match → propose linking into `Mentioned in`. Otherwise a new row.
- **Advisory Insights:** for each piece of advice, check existing Advisory Insights for this Advisor of the same Type. Match → propose linking into `Mentioned in` (recurring themes link across calls). Otherwise a new row.
- Show the proposed matches; I confirm before any merge.

## Step 5 — Show the plan and wait
List everything you're about to write: the Conversation row, People/Account to create or link, Requests (new vs linked), Insights (new vs linked). **Write nothing until I say go.**

## Step 6 — Write to Notion
On my go:
- Upsert the Account and People (set Attio links if I gave them).
- Create the Conversation: title `{Account or Person} — {Type} — {YYYY-MM-DD}` (use the Person when there's no Account; advisory calls use `Vesper Alpha — Advisory call — {YYYY-MM-DD}`); set Date, Type, Attio transcript, Account, People. Put the call summary in the page **body** — never paste the raw transcript.
- New Requests: set Request, Category, Status = Captured, Customer = Account, Raised by = Person, Mentioned in = this Conversation. Matched Requests: just add this Conversation to `Mentioned in`.
- New Customer Insights: set Insight, Type, Person, Mentioned in. Matched: add this Conversation to `Mentioned in`.
- New Advisory Insights: set Insight, Type, Advisor (the Vesper Alpha partner who said it), Mentioned in. Matched: add this Conversation to `Mentioned in`.

## Don'ts
- Never touch Linear or set delivery status — promoting a request to a ticket is a separate triage step, not this skill.
- Never copy the transcript into Notion; link to Attio.
- Never auto-merge a dedup match I haven't confirmed.

## Targets (data source IDs)
- Accounts `23d057d9-5005-4bd2-ade1-97b11c0f40bb`
- People `deae1bb3-800c-41e7-a42c-57d06128379f`
- Conversations `9254ee74-9648-48c2-aa14-ea37f8b59d52`
- Customer Requests `50813116-3a1c-4383-add5-af8bb0fb33a4`
- Customer Insights `49826b8b-ec88-47b3-8007-73451906281a`
- Advisory Insights `2ef68483-410c-4700-85cb-4939b8262b1a` (advisory calls only; Advisor relation → People, Mentioned in → Conversations)
