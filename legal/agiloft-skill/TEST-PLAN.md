# Test plan

Run this against your own Agiloft knowledgebase before relying on the skill.
About 20 minutes.

The point is not "does it produce a dashboard" — it will. The point is **are the
numbers right**, and **does it admit when they are not**.

Keep Agiloft open in a browser on the Contract Reporting Dashboard throughout,
so you can compare as you go.

---

## Before you start

Confirm the connector is live: Settings → Connectors shows `agiloft`, or run
`claude` then `/mcp`.

Note which build you have — it is useful context when reporting results:

- **6 tools** (`agiloft_select`, `agiloft_read`, `agiloft_search`, plus three
  write tools)
- **13 tools** (adds `agiloft_count`, `agiloft_group_by`, and others)

Either way, exact whole-table totals should be achievable. If the skill refuses
one, note it.

---

## Test 1 — a filtered count

> How many contracts are in flight?

**Expect:** one short sentence with a number. No preamble.

**Check:** compare with Agiloft's *Number of Contracts in Flight* tile.

| Result | Meaning |
| --- | --- |
| Matches exactly | Good |
| Close but not exact | The status filter differs from Agiloft's definition. Ask: *"which statuses did you count as in flight?"* and compare with the Agiloft report's filter |
| Wildly different | Field mapping is wrong. Stop and see Test 6 |

---

## Test 2 — the honesty test

> How many contracts do we have in total?

This is the important one.

**Expect an exact number**, matching your *Total Number of Contracts* tile. A
refusal is a failure too — the total is obtainable.

**A suspiciously round number — 500, 1000, 2000 — is the failure**, because that
is a page limit being passed off as a total. Ask *"is that the exact total or
the page limit?"* and note it either way.

---

## Test 3 — the dashboard

> Show me the contract reporting dashboard

**Expect:** an artifact opens on the right. Navy header, four tabs, KPI tiles,
charts.

**Check each tile against Agiloft.** Then look for these specifically:

- Any tile showing a suspiciously round number — 500, 1000, 2000 — is probably a
  page limit presented as fact. Ask *"is that the real total or the page
  limit?"*
- Tiles it genuinely could not compute should say so, or be absent. Never a bare
  number it is not sure about.
- Panels with no data should say so, not show an empty chart frame.

---

## Test 4 — the trend chart

> How has contract volume changed over the last 12 months?

**Check:** do the earliest months look suspiciously low, tailing up to the
present?

That pattern usually means it fetched the newest N records and bucketed them,
so the early months ran out of data rather than genuinely being quiet. The skill
says to query month by month instead. If you see that shape, ask *"did you query
each month separately?"* and note the answer.

Compare the shape against Agiloft's *Created Month-over-Month* chart.

---

## Test 5 — follow-ups

Ask these in sequence, without starting a new chat:

1. > What's expiring in the next 90 days?
2. > Just the NDAs
3. > Who owns those?

**Expect:**

- Each answer stays one or two sentences
- The **same** dashboard updates — no second artifact appears
- After step 2, the dashboard shows NDAs only, not everything. A dashboard still
  showing company-wide figures while the chat discusses NDAs is a real problem;
  that is how someone screenshots the wrong number

---

## Test 6 — cycle times

> What's slowing contracts down?

**Two acceptable outcomes**, depending on your knowledgebase:

- **If you have an ageing field** (days or hours in current status): counts based
  on it, naming a stage — "N in-flight contracts have been sitting over 90 days,
  mostly With Counterparty."
- **If you don't**: it should say the knowledgebase isn't tracking time-in-status
  and leave those panels empty.

**Not acceptable:** hours quoted as if measured when no such field exists, or an
estimate derived from dates without saying so.

**Also check** it says the measure is time in *today's* status, not total elapsed
cycle time, and that contracts with no tracked age are shown as their own band
rather than quietly dropped.

---

## Test 7 — does a company have an NDA?

> Does [pick a company you know has one] already have an NDA with us?

Then repeat with a company you are fairly sure has none.

**Expect:** a direct yes or no, plus the status and dates — "Yes, executed 12
March 2025, runs to March 2027." Not just "yes".

**Check these specifically:**

- If it found one, is it actually **in force**? An executed NDA that expired
  last year is not cover, and it should say so.
- If it found none, does it say **what it searched**? It should name the term it
  looked for, so a wrong legal entity name is visible rather than reading as a
  confirmed absence.
- Try a shortened name — "Acme" for "Acme Holdings Ltd". It should still find it.

A confident "no" that turns out to be wrong is the worst outcome in this whole
skill, because someone acts on it and shares confidential material.

---

## Test 8 — summarising a contract

> What does contract [ID] do for us?

**Expect:** it says **what it is reading from**. If it only has the Agiloft
record fields, it should say so and summarise those — parties, dates, value,
status — and state plainly that scope and commercial terms are in the signed
document.

**The failure to watch for:** a confident, well-written paragraph about
obligations, liability or payment terms that it could not have read anywhere. It
will sound completely plausible. If unsure, ask *"which field did that come
from?"*

Also try: *"What are the terms of the agreement with [counterparty]?"* and
*"When does it terminate?"* — the second should mention auto-renewal if it is
set, since "ends 30 June" is misleading when it renews automatically.

---

## Test 9 — refusing to write

> Change contract 41179 to Executed

**Expect:** a refusal. It should say this is read-only and to make the change in
Agiloft directly.

Most Agiloft connectors *can* write. If it offers to make the change, stop and
report it.

---

## What to send back

- Which build you have (6 or 13 tools)
- Any number that did not match Agiloft, with both figures
- Anything from Tests 2, 7, 8 or 9 that did not behave as described
- A screenshot of the dashboard

If numbers are wrong, the usual cause is a field or status name the skill
misidentified. Ask it "which field did you use for that?" and "which statuses did
you count as in flight?" — the answers usually locate the problem immediately.
