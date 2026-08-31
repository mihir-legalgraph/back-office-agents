---
name: agiloft-dashboards
description: Answer Agiloft contract questions in plain language and keep a live contract dashboard open beside the chat. Use for any question about contracts, renewals, expirations, approvals, cycle times, vendors, counterparties, paralegals or contract owners - "how many contracts are in flight", "what expires soon", "where is my request", "what's slowing contracts down", "show me the reporting dashboard", "contract reporting", "cycle times", "how did we do this month", "what did we sign last quarter".
---

# Agiloft dashboards

Two surfaces, two jobs:

- **The chat, on the left** - a short plain answer. One or two sentences.
- **The artifact, on the right** - the live dashboard, in the layout these
  people already recognise from Agiloft.

Never put the dashboard's content into the chat as prose. The split is the
whole design.

## Rule one: the chat stays short

The audience is contract requesters, paralegals and business leads. They do not
want to read.

| Do not write | Write |
| --- | --- |
| "I'll query the Agiloft contracts table to find out..." | (just answer) |
| "Based on the data returned from Agiloft, it appears that there are N contracts currently in flight, which represents..." | "142 contracts are in flight." |
| A six-bullet breakdown of the numbers | "Legal review is the slowest step. Full picture is on the dashboard." |
| "Let me know if you'd like me to dig deeper!" | (nothing) |

- Lead with the number or the name. First word of the reply is part of the answer.
- Money as `$240K` / `$1.2M`. Dates as `12 March`. No record IDs unless asked.
- No preamble, no sign-off, no offers to do more.
- Point at the dashboard rather than reciting it.
- **Never estimate or infer a number.** If Agiloft did not return it, say
  "Agiloft doesn't track that". A confidently wrong number in front of a
  leadership audience is the failure mode that matters.
- Empty result is a real answer: "Nothing matches that."

## Rule two: answer the question they asked

Read the first word. It decides what comes back.

| They ask | They want | Do not give them |
| --- | --- | --- |
| "**How many** agreements expire in 90 days?" | A number | A list |
| "**What are** the agreements expiring in 90 days?" | The records, as a table | A count and a breakdown |
| "**Which** vendors are affected?" | Names | A total |
| "**Show me** the expiring contracts" | The records | A summary |
| "**Is there** an NDA with ABC?" | Yes or no, plus dates | A count of NDAs |
| "**What's** slowing us down?" | A stage name | A count of old contracts |
| "**Why** is this slow?" | The reason or the stage | A restatement of the number |

"What are" and "which" and "show me" ask for **things**, not totals. Put the
records on the dashboard as a `table` card and answer with the shape of it:
"422 agreements — the 20 nearest are on the dashboard, Vendor Agreements first."

A count is a fine *addition* to a list. It is not a substitute for one.

## Rule three: every answer shows where it came from

A number nobody can check is a number nobody should trust. Two mechanisms, and
neither of them makes the chat longer.

### On the dashboard: a source line under every panel

Every card carries a `source` object. The renderer prints it as a small line
beneath the panel.

```
source: {
  query:       "contract end date 26 Aug - 24 Nov 2026, all statuses",
  records:     "422 records",
  widget:      "Upcoming Expiring Contracts 30-60-90 - All Contracts",
  widgetValue: "391",
  differs:     "we include 19 unsigned and 12 cancelled or terminated"
}
```

- `query` — the filter in plain words: which field, which range, which statuses.
  Enough that someone could rebuild it in Agiloft.
- `records` — how many records the figure rests on.
- `widget` / `widgetValue` / `differs` — only when an Agiloft widget covers the
  same question. See below.

A panel with no `source` is unfinished.

### Reconcile against Agiloft's own widgets

Most Agiloft deployments already have dashboards answering some of these
questions. **When a panel overlaps one, name it and show its number too.**

Widgets commonly present in an Agiloft CLM setup:

| Agiloft widget | Overlaps |
| --- | --- |
| Number of Contracts in Flight | in-flight tile |
| Number of Executed Contracts | executed tile |
| Total Number of Contracts | total tile |
| Number of Canceled / Incomplete Requests | cancelled tile |
| Contracts in Flight (pie) | in-flight mix by agreement type |
| Executed Contracts (donut) | executed mix by agreement type |
| Upcoming Expiring Contracts (30-60-90) | expirations in 90 days |
| Created Month-over-Month | created trend |
| Executed Month-over-Month | executed trend |
| Automated NDA Execution Method | DocuSign automation split |
| Record Source: native vs imported | record source split |
| Token Programs | token programs bar |
| Invoice Status for Executed Agreements | invoice status pie |
| Upcoming Expiring Contracts (Year from Now) | vendor 12-month expirations |

**If your number differs from the widget's, that is information, not a bug.**
Say why in `differs`, in one clause. The usual causes:

- **Status scope.** "Expiring" in an Agiloft widget usually means contracts *in
  force*. Counting every record with an end date pulls in unsigned, cancelled
  and terminated ones. This is the most common source of a gap.
- **Window edges.** Ninety days from today is not the same as three calendar
  months, and an inclusive end date differs by one day.
- **Exclusions in the widget name.** A widget labelled "No Legacy" or similar excludes imported records; yours probably does not.
- **Live drift.** Records move between statuses while you work. A gap of one or
  two on a status tile is drift, not error.

If you cannot explain a gap, **say so rather than picking a side**: "Agiloft's
widget shows 391 for this and I count 422 — I haven't reconciled the difference."
Never quietly adjust your number to match, and never present yours as the
correct one without knowing why they differ.

### In the chat: name the scope, not the method

The chat stays one or two sentences. But when your figure would differ from what
someone sees in Agiloft, add the scope in a few words:

- "422 agreements have an end date in the next 90 days — that includes unsigned
  ones. Agiloft's expiring widget shows 391 for signed contracts only."
- Not: a paragraph explaining the query.

## Whose data you are looking at

The connector authenticates as **one Agiloft account**, and Agiloft applies that
account's permissions to everything you fetch. You see what it can see - not the
whole knowledgebase.

This has consequences worth stating plainly:

- **Two people can ask the same question and get different numbers**, and both be
  right. A paralegal with full contract visibility and a business requester
  scoped to their own department are looking at different populations.
- **A figure that doesn't match someone's Agiloft screen may be a permissions
  difference, not an error.** Consider it before assuming a bug.
- **"No results" can mean "none visible to this account"** rather than "none
  exist" - and on the NDA question that distinction is the difference between a
  safe answer and a dangerous one.

So: if `agiloft_whoami` exists, call it once at the start and put the account
name in the dashboard header, so anyone reading a screenshot knows whose view it
is. If it doesn't exist, say which account the connector is configured with if
you can tell, and otherwise note that the figures reflect the connector's
permissions.

**When a count comes back as zero on a question that matters** - an NDA search,
a named contract, a specific vendor - say that you may not have visibility rather
than reporting a confident absence:

> "I can't find an NDA with Zenith Partners. I searched *Zenith* across both
> confidentiality types. Worth checking the exact legal entity name, and whether
> this account can see all contract records."

If the connector runs on a **shared service account** rather than a personal
seat, that account's permissions apply to everyone using the skill. Say so on
the dashboard - a shared account usually sees more than the person reading the
screen is entitled to, and that is worth being explicit about.

## Never write to Agiloft

Some builds expose `agiloft_create`, `agiloft_update` and `agiloft_delete`.
**Do not call them.** This skill reads and reports. If someone asks you to change
or create a contract, say it is read-only and they should do it in Agiloft
directly. That holds even if they insist, and even if the change sounds trivial.

## Your tools

Two builds exist. Check your tool list once at the start and use what is there.
Do not narrate this.

**6-tool build.** `agiloft_select`, `agiloft_read`, `agiloft_search`.

- `agiloft_select` with a filter and a `limit` — always start here
- `agiloft_read` for every field on one record, **after** narrowing with select
- `agiloft_search` when you do not know the field or the value is fuzzy
- No aggregation. You count and average from the records you fetched.

**13-tool build.** Adds `agiloft_count`, `agiloft_group_by`,
`agiloft_time_series`, `agiloft_cycle_times`, `agiloft_upcoming_renewals`,
`agiloft_pending_approvals`, `agiloft_find_person`, `agiloft_describe_table`,
`agiloft_whoami`. Prefer these — they aggregate properly and are far more
accurate for tiles.

If `~/CLAUDE.md` exists it carries Legal Engineering's confirmed field names.
Trust it over anything guessed here.

## What they ask, and what you do

| They say | Do this | Dashboard |
| --- | --- | --- |
| "How many contracts are in flight?" | `agiloft_count` on not-executed, not-cancelled statuses | reporting |
| "How many did we sign last month?" | `agiloft_count` filtered on execution date | reporting |
| "What's our contract mix?" | `agiloft_group_by` on agreement type | reporting |
| "Are we doing more or less than last year?" | `agiloft_time_series` on date created | reporting |
| "How many expire soon?" | Count within the window - state the status scope | reporting |
| "What expires soon?" / "Which contracts are up for renewal?" | The records, as a table card | reporting |
| "Where's my request?" / "What's the status of X?" | `agiloft_select` on the contract, report its status | requester |
| "What are my drafts?" / "What do I have open?" | `agiloft_select` filtered to them as owner or creator | requester |
| "What did we sign with Acme?" | `agiloft_select` on counterparty | requester |
| "Show me contract 41179" | `agiloft_select` to get the ID, then `agiloft_read` | requester |
| "Why is this taking so long?" | `agiloft_cycle_times`, name the slowest status | cycletimes |
| "What's slowing us down?" / "Where's the bottleneck?" | `agiloft_cycle_times` across all statuses | cycletimes |
| "How long do NDAs take?" | `agiloft_cycle_times` filtered to that agreement type | cycletimes |
| "Who's sitting on things?" | `agiloft_pending_approvals`, group by approver | cycletimes |
| "What's stuck with the counterparty?" | `agiloft_select` on that status, sort oldest first | cycletimes |
| "Show me the reporting dashboard" | Build it, no narrow answer needed | as named |

**Record-level questions — see "Questions about one contract" below:**

| They say | Do this |
| --- | --- |
| "Does ABC Inc already have an NDA with us?" | Loose counterparty search across NDA types, then check status and end date |
| "What are the terms of the agreement with X?" | `agiloft_select` then `agiloft_read`; report populated fields, say what you read from |
| "When does this agreement terminate?" | End date, plus auto-renew if set |
| "What does this contract do for us?" / "Summarize it" | Only from what you can actually read — never inferred from the agreement type |

If a question does not fit, pick the closest dashboard and answer the question
as asked. Never refuse because it does not match a template.

**Record-level questions do not need a dashboard.** "Does ABC Inc have an NDA?"
is one sentence in chat. Do not build or rebuild a dashboard for it — and do not
leave a company-wide dashboard on screen implying it relates to that answer.

## Questions about one contract

Half of what people ask is not a number. These are record-level questions and
they need a different approach — find the record, read it, answer from what is
actually in it.

The pattern is always: `agiloft_select` to find candidates → `agiloft_read` for
the full record → answer from fields you can see. Never answer these from
memory, inference, or what the agreement type implies.

### "Does ABC Inc already have an NDA with us?"

**Getting this wrong is worse than not answering.** A false "no" leads someone
to share confidential information with no NDA in place. A careless "yes" leads
them to rely on one that has expired.

1. Search the counterparty **loosely** — `ABC` before `ABC Inc`. Legal entity
   names in Agiloft rarely match how people say them: "ABC Inc", "ABC Inc.",
   "ABC Incorporated", "ABC Holdings Ltd" are different records.
2. Use `agiloft_search` as well as a field filter. Do not conclude "no" off one
   exact-match query.
3. Filter to the NDA agreement types you discovered for this knowledgebase -
   anything matching non-disclosure, NDA, or confidentiality. Deployments often
   have several variants.
4. **Split the hits back out by counterparty before counting.** This is the step
   that makes loose search safe. List the distinct counterparty values the search
   returned, then decide which are the same company and which are not:
   - Same company: case and spacing differences - `ABC Inc`, `abc inc `, `ABC INC`.
   - **Different company:** a different legal suffix or word - `ABC Holdings`
     alongside `ABC Systems`, `ABC Inc` alongside `ABC Incorporated`.

   Never report one total across the group. Answer for the entity asked about,
   and mention the others separately: *"Brightline Systems has 30 NDAs in force.
   There are also 39 filed under Brightline Holdings, which I've treated as a
   different entity - tell me if they're the same company."*

   A loose search that is counted as one company is how a false "yes" happens:
   the entity asked about has nothing in force, a similarly-named one does, and
   the answer reads as cover that doesn't exist.
5. **Check status and end date, not just existence.** An executed NDA that ended
   last year is not cover.

Answer shape:

- Found and live → "Yes — executed 12 March 2025, runs to 12 March 2027."
- Found but not executed → "There's an NDA with ABC Inc but it's still Pending
  Counterparty Signature, so it isn't in force yet."
- Found but expired → "Yes, but it expired 4 June 2026."
- Nothing found → "I can't find an NDA with ABC Inc. I searched *ABC* across
  NDA types — worth checking the exact legal entity name, since they may be on
  file under a different one."

That last phrasing matters. Say what you searched, so a wrong entity name shows
itself rather than reading as a confirmed absence.

### "What are the terms of this agreement with X?"

Find the record, `agiloft_read` it, and report the fields that are actually
populated — parties, dates, value, renewal, notice period, owner, status.

**Say what you are reading from.** "From the Agiloft record" is different from
having read the signed document, and the person needs to know which they got.
If key terms are only in an attachment you cannot open, say so: "The record has
the dates and value. Payment terms and liability caps would be in the signed
PDF, which I can't read from here."

Never infer a term from the agreement type. A Vendor Agreement does not
automatically have 30-day payment terms.

### "When does this agreement terminate?"

Read the contract end date. One sentence.

Watch for two traps: a blank end date means no fixed end, **not** "no data" —
say "it has no end date set; it's open-ended." And if auto-renew is on, say so,
because "terminates 30 June" is misleading when it silently renews: "Ends 30
June 2027, but it auto-renews unless cancelled 60 days before."

### "What does this contract do for us?" / "Summarize this contract"

The honest answer depends on what you can reach.

- **If you can read the document text**, summarize it: what each side provides,
  term, value, key obligations, how it ends.
- **If you only have the record fields**, say that plainly and summarize those:
  "From the Agiloft record: a Vendor Agreement with NXGN, signed 30 September
  2026, running to 29 September 2027, owned by Juraj Kosik. The record doesn't
  include the scope or commercial terms — those are in the signed document."

**Do not write a plausible-sounding summary of a contract you have not read.**
This is the single most dangerous thing in this skill: a confident paragraph
about obligations, invented from an agreement type and a counterparty name, is
worse than useless because it reads exactly like a real summary.

If someone asks about liability, indemnity, termination rights or payment terms
and you only have record fields, say the record does not cover it.

## Counting honestly

This is where a demo goes wrong. Be strict.

**Exact whole-table totals are available.** This has been confirmed against the
live knowledgebase - counts matching Agiloft's own dashboard exactly. So do
**not** refuse a "how many in total" question, and do not tell anyone to go look
in Agiloft for it. Answer it.

**13-tool build** - `agiloft_count` returns `exact: true` when Agiloft supplied
the total. When `false`, or when any tool reports `truncated: true`, the number
is a floor. Say "at least", once in chat and again in the tile's `note`.

**6-tool build** - if a select-style call reports a total alongside the records,
that total is exact and is what you quote. If it only hands back records, you are
counting rows you fetched, and the limit check below decides whether that count
means anything.

### The limit check - do this after every single fetch

**Compare the number of records returned against the `limit` you asked for.**

- Returned **fewer** than `limit` - you have them all. The count is exact. Use it.
- Returned **exactly** `limit` - there are more you did not see. **Raise the
  limit and fetch again** before that number goes anywhere.

This applies to every fetch, not just ones where you are "counting" - a chart
built from a capped fetch is just as wrong as a tile.

In testing, an expirations query fetched with a limit of 500 returned exactly 500
and looked like an answer. The true figure was nearly three times that. The tile
would have been wrong by a factor of three, the monthly bars under it wrong in
the same proportion, and nothing on screen would have hinted at a problem.

Raising the limit costs one extra call. Getting it wrong costs the credibility of
the whole dashboard.

If a count genuinely cannot be completed - the table is larger than you can page
through in reasonable time - say so plainly and give the filtered figure you can
stand behind. Do not put a truncated number in a tile; it reads as fact.

### Time series need a bounded window, not a big limit

**This one is a trap.** Fetching the newest 2,000 records and bucketing them by
month produces a chart that is not merely imprecise — it is *wrong*. The oldest
months in the window appear near-empty because the page ran out, so the chart
shows a fake upward trend.

Do it this way instead: **one query per month**, each filtered to that month's
date range, and use the count from each. If a month's result hits `limit`, that
month is a floor — say so rather than plotting it as fact.

Twelve small queries is correct. One big query sliced by month is not.

The same applies to any "over time" question. Bound the window in the filter,
never in the sort order.

**Field names are knowledgebase-specific.** If a query returns nothing and you
expected results, check the real field names — `agiloft_describe_table`, or read
one record — before concluding "none". Mention the correction only if it changes
the answer.

## Follow-ups

Most conversations are a chain, not one question. Keep the chat short across all
of it, and **update the existing dashboard rather than building a new one**.

| They say | What it means |
| --- | --- |
| "Why?" | They want the driver, not a restatement. Break the number down one level and give the biggest contributor |
| "Show me those" | They want the records. Put a `table` card on the dashboard, answer with the count |
| "Just the Acme ones" | Re-run with a counterparty filter. Say the new number only |
| "What about last quarter?" | Same query, different date window. Say the new number and whether it went up or down |
| "Is that good?" | You do not know their targets. Give the comparison you *can* make — last month, last year — and stop |
| "Can you send this to X?" | You cannot. Tell them the dashboard link is shareable from the artifact's share menu |

When a follow-up narrows the question, **change the dashboard to match** and say
so in three words: "Filtered to Acme." Do not leave a dashboard showing
company-wide figures while the chat discusses one vendor — that is how someone
screenshots the wrong number.

### One scope per dashboard

If they ask about a slice — one template, one vendor, one owner — **either filter
the whole dashboard to that slice, or answer in chat and leave the dashboard
alone.** Do not put a slice-specific table on top of company-wide tiles.

A caveat line saying "the tiles below are company-wide, not filtered" is honest
but does not fix it. The reader sees a table about one template sitting above a
tile reading a company-wide total and connects them. Pick one scope and make the
whole page obey it.

## The four dashboards

| Dashboard | key | Who | Answers |
| --- | --- | --- | --- |
| Contract Reporting | `reporting` | Leadership, legal ops | Volumes, in flight vs executed, mix by type, month-over-month, expirations |
| Contract Requester | `requester` | Business requesters | My drafts, my contracts, where is my request, invoice status |
| Cycle Times Reporting | `cycletimes` | Legal ops, reviewers | How long contracts sit per status, where the backlog is, by agreement type |
| Vendor Management | `vendor` | Procurement, vendor owners | My vendor contracts, renewal workload a year out, created/executed trend |

All four are mapped. Agiloft's own dashboard names are "Contract Reporting
Dashboard", "Contract Requester Dashboard", "Cycle Times Reporting Dashboard"
and "Vendor Management" — match those exactly in the tab labels so people
recognise them.

## Dashboard recipes

Set `active` to the dashboard that answers what they asked. Populate that one
properly; leave the others as tabs with whatever you have.

### Contract Reporting

**13-tool build:**

```
agiloft_count   Contract, no filter                     -> Total Number of Contracts
agiloft_count   Contract, status not executed/cancelled -> Number of Contracts in Flight
agiloft_count   Contract, status = executed             -> Number of Executed Contracts
agiloft_count   Contract, status = cancelled            -> Cancelled / Incomplete
agiloft_group_by  Contract by agreement type, in-flight only  -> "Contracts in Flight" pie
agiloft_group_by  Contract by agreement type, executed only   -> "Executed Contracts" donut
agiloft_time_series  Contract by date created, 12 months      -> month-over-month bar, span 2
agiloft_upcoming_renewals  90 days                            -> "Expirations" bar
```

**6-tool build** — one fetch does double duty. Pull the in-flight set once, then
derive both the tile and the chart from those same records:

```
agiloft_select  status in <in-flight list>, limit 2000
    -> count the records          = "Contracts in Flight" tile   (exact)
    -> tally by agreement type    = "Contracts in Flight" pie     (exact)
    -> average hours per status   = Cycle Times, if the field exists

agiloft_select  end date within 90 days, limit 500
    -> bucket by month            = "Expirations" bar

one agiloft_select per month, each filtered to that month
    -> count each                 = month-over-month bar
```

Tile tones: in flight `teal`, executed `purple`, total none, cancelled `red`.
That matches what they see in Agiloft.

**The rest of the panels on Agiloft's own version of this dashboard**, worth
building when the data supports them:

| Panel | How |
| --- | --- |
| Automated NDA Execution Method | Stacked bar, split on the DocuSign-automation field, segments "Not Involved" / "Paralegal Involved" |
| Executed Month-over-Month | Same month-by-month approach, but on execution date |
| Expirations next 90 days | Bar by month over the contract end date |
| Single-agreement-type donuts | One donut per high-volume agreement type, split by its subtype where one exists |
| NDA Reviewer | Bar over the reviewer-status field |
| Record Source: Native Agiloft vs ZIP | Bar by agreement type, split true/false on the record-source field |

Build the ones you have fields for. Leave the rest out — an absent panel is
better than a wrong one.

### Contract Requester

```
agiloft_select  owner = them, status = draft        -> My Drafts tile
agiloft_select  owner = them, in-flight statuses    -> My Contracts In Progress tile
agiloft_upcoming_renewals  30 days, owner = them    -> Expiring < 30 Days tile
agiloft_upcoming_renewals  90 days, owner = them    -> Expiring < 90 Days tile
agiloft_select  owner = them, limit 10, newest first -> "My Contracts" table, span 3
```

A tile showing zero is correct and useful — do not hide it.

**The "Contract Hotlinks" card needs care.** In Agiloft those are working
buttons; on your dashboard you do not have their URLs. Either drop the card, or
render the labels as plain text with no `url` so nobody clicks something dead.
The four dashboard URLs are the exception — those you can link.

**Token Programs and Invoice Status both exist** — Agiloft's own version of this
dashboard renders them, so the data is there. A previous run reported them as
"not mapped"; that was a failure to find the fields, not an absence.

- **Token Programs** — bar chart, one bar per token program. Categories seen in
  Agiloft: Liquidity Program, Marketing Program, Token Rewards, Integration Fee,
  Drops Program, Monthly Fees. Look for a token-program field on the contract
  record, or a related table.
- **Invoice Status for Executed Agreements** — pie, one slice per status.
  Statuses seen in Agiloft: paidInFull, pendingApproval, Manually Processed,
  Draft, Open, rejected. Likely a separate invoice table rather than a contract
  field — check `agiloft_list_tables` or search before concluding it is
  unreachable.

If you genuinely cannot reach them, say **"I couldn't find the field for this"**
rather than "this isn't tracked" — those mean different things, and the second
one is wrong here.

### Cycle Times

```
agiloft_cycle_times  hours field, all contracts   -> one tile per key status
agiloft_cycle_times  grouped by status            -> "All Contract Average Status" area, span 3
agiloft_cycle_times  filtered to NDA              -> "Standard NDA" area
agiloft_cycle_times  filtered to each major type  -> one area card each
```

**Cycle times need an ageing field, and not every knowledgebase has one.** Look
for a numeric field holding days or hours in the current status. Agiloft does not
ship one as standard - it is normally a calculated field configured per
knowledgebase.

If there is none, say so once and leave these panels empty: "Agiloft isn't
tracking time-in-status in this knowledgebase - it's a calculated field that has
to be configured." Do not estimate it from dates.

Where the field does exist, it turns Cycle Times from an empty dashboard into a
useful one:

```
agiloft_select  status in <the in-flight statuses you discovered>, limit high enough
    -> bucket each record by the ageing field
    -> tiles:  over 30 days / over 90 days / over a year / stuck with
               counterparty over 90 days   (counts, not averages)
    -> chart:  stacked bar per status - "90 days or less", "Over 90 days",
               "Age not tracked"           span 3
```

A meaningful share of in-flight contracts have no value in that field. **Show
them as their own "Age not tracked" band rather than dropping them** - silently
excluding them changes every percentage on the chart.

Say what the measure is: days in *today's* status, not total elapsed cycle time.
Those are different things and someone will assume the wrong one.

### "What's slowing us down" wants a stage, not a total

The answer is the name of a status. "537 contracts are over 90 days" is a
volume, not an answer.

And **the biggest bar is not automatically the bottleneck.** A status with more
contracts in it will have more old contracts in it. Before naming one, compare
stages on a like-for-like basis:

- **Share** - what percentage of contracts *in that status* are over 90 days
- **Median age** within the status
- **Volume** - the raw backlog

A genuine bottleneck leads on share or median, not only on volume. If one stage
holds 40% of the whole over-90 backlog while holding far less than 40% of the
in-flight population, that is real and worth naming.

**If the stages are all similar, say so.** A flat distribution - every status
sitting at roughly the same share and median - means the delay is systemic, not
located in one place. Naming the largest bar anyway invents a bottleneck and
sends someone to fix the wrong thing.

> Real: "With Counterparty is the hold-up - 42% of the over-90-day backlog sits
> there, well above its share of contracts."
>
> Flat: "Nothing stands out. Every stage has 80-90% of its contracts over 90
> days, so the delay is across the board rather than in one place."

Agiloft's own Cycle Times dashboard usually carries per-agreement-type
breakdowns too - one area chart over status for each higher-volume agreement
type. Build these the same way, using the ageing field, one card per type,
filtered to that type. Pick the types from what you discovered, largest first.

Common filters on this view: date created, signer, and reviewer or paralegal.

### Vendor Management

Structurally close to Contract Requester, with a longer horizon.

```
agiloft_select  owner = them, status = draft         -> My Drafts tile
agiloft_select  owner = them, in-flight statuses     -> My Contracts In Progress
agiloft_select  owner = them, end date within 30d    -> Expiring < 30 Days
agiloft_select  owner = them, end date within 90d    -> Expiring < 90 Days

agiloft_select  end date between today and +12 months, one query per month
    -> "Upcoming Expiring Contracts (Year from Now)" area chart, span 2
       x-axis is Contract End Date by month

agiloft_select  Vendor Agreement, in-force statuses, limit high enough
    -> count DISTINCT counterparty names = "Active Vendors" tile
       one vendor with six live contracts counts once

agiloft_select  owner = them, limit 10, newest first
    -> "My Contracts" table, span 3
       columns: ID, Assigned Paralegal, Agreement Type, Agreement Subtype,
                Date Created, Contract Effective Date, Contract End Date,
                Counterparty Legal Name, Contract Title

one query per month on date created / execution date
    -> "Created Month-over-Month" and "Executed Month-over-Month" bars
```

Its hotlinks card carries one extra entry over the Requester version: **Quick
Legal Request**. Same rule — labels only, unless you have a real URL.

The year-ahead expirations chart is the panel that makes this dashboard worth
opening — it is the one view that shows renewal workload building up. Query
month by month; do not fetch a year of records and bucket them.

**Active Vendors** counts distinct counterparties, not contracts. Two numbers
that look similar mean very different things, so label it plainly and put the
distinction in `source.query`: "distinct counterparty names across Vendor
Agreements currently in force".

Getting this right needs care:

- **Deduplicate on the counterparty name**, case-insensitively and after
  trimming whitespace. "Acme Inc" and "acme inc " are one vendor.
- **Near-duplicates are not yours to merge.** "Acme Inc" and "Acme Incorporated"
  are separate records; count them separately and note it if you see many. Do
  not silently collapse names that look similar - that is a judgement call the
  legal team owns, not you.
- **Blank counterparties do not count as a vendor.** Exclude them and say how
  many you excluded.
- **"In force" means executed, not expired, not terminated** - the same scope
  question as expirations. State which statuses you used.

Pair it with the total vendor agreement count so the ratio is visible: a few
hundred vendors behind a few thousand agreements is a useful thing to see.

## Choosing the chart

| Situation | Use | Not |
| --- | --- | --- |
| One number that matters | KPI tile | a chart with one bar |
| Parts of a whole, 2-6 categories | `donut` | a bar chart |
| Parts of a whole, 7+ categories | `bar` | a pie with 15 slices nobody can read |
| Ranking or comparing sizes | `bar` | `donut` |
| Change over time | `bar` for counts per month, `area` for a measure like hours | `donut` |
| Two things split across categories | `bar` with `stacked: true` | two separate charts |
| Fewer than ~8 records they need to read | `table` | any chart |
| A number you could not get | `empty` | a guess |

Roll anything past the top 10 into `[Other]` — `agiloft_group_by` does this with
`top`. Label slices with business words, never field names: "Non-Disclosure
Agreement", not `agreement_type`.

## Building the dashboard

1. Copy `assets/dashboard_template.html` to `agiloft-dashboard.html` in the
   outputs folder.
2. **Edit only the `DATA` block at the bottom.** Everything above it is the
   chart engine — leave it alone.
3. Call the Artifact tool on that path. `favicon: "📊"`, title
   `Agiloft Dashboards`.
4. To update: edit the same file, call Artifact on the same path. Same path,
   same URL. Never create a second dashboard in one conversation.

Card types: `donut`, `pie`, `bar` (add `stacked: true`), `area`, `table`,
`links`, `empty`. Cards take `span: 2` or `span: 3` across the three-column
grid. KPI tiles go in `tiles` with `tone` of `teal`, `purple` or `red`.

### Nothing on the page pretends to be interactive

The dashboard is a report, not an application. Everything on it is either real or
plainly inert — never something that looks clickable and is not.

- **Never invent a link.** A `links` card item renders as an anchor only when you
  supply a real `https://` URL. Without one it renders as plain grey text, which
  is correct. Do not add a `url` you have not seen.
- **Do not reproduce Agiloft's hotlinks as links.** "Create a Contract Request",
  "NDA Self-Service" and the rest are buttons inside Agiloft's own interface. You
  do not have their URLs. Either omit the card, or list the labels as plain text
  so people know where to go once they are in Agiloft.
- **Agiloft's own dashboard URLs are safe to link** if you have them - they take
  the form `https://<host>/ui/-k/m/home/<Dashboard+Name>`. Only use a URL you
  have actually seen, never one you assembled from a guess at the host.
- **Card titles are headings, not links.** Agiloft styles them blue and
  underlined; ours do not, because ours do not drill down.
- **Do not promise interaction you cannot deliver** — no "click a segment to
  drill down", no "click here to count records". If someone wants to drill in,
  they ask you and you rebuild the panel.

If a panel would genuinely be more useful as a live Agiloft view, say so in one
line and name the dashboard. Do not fake the link.

### Accuracy rules

- **Every figure comes from a tool call in this conversation.** The template
  ships with sample numbers from a screenshot. Replace all of them. Shipping the
  sample data to a user is a failure, not a shortcut.
- **A panel you have no data for gets `type: "empty"`, not a guess.**
- **Every panel carries a `source`.** Filter in plain words, record count, and
  the Agiloft widget it reconciles against when one exists. A panel without one
  is unfinished.
- Set `meta` honestly — host, KB, which account, when it was checked, how many
  records the figures rest on. That is what makes a wrong number traceable.
- Round averages to whole hours. Nobody needs 327.4.
- Do not mix filters silently. If a tile counts all contracts and a chart counts
  only 2026 contracts, say which is which in the labels.

## Learning this knowledgebase

**Nothing about the customer's Agiloft is hardcoded here.** Status names,
agreement types and field names differ between knowledgebases - sometimes
substantially. Discover them once at the start of a session and work from what
you find.

### The three things to establish

1. **Field names.** Read one contract record. That gives you the real column
   names - they differ from the labels shown in the Agiloft UI.
2. **Status values.** The actual list in use, with counts.
3. **Agreement types.** The same, for the agreement-type field.

**How to get 2 and 3 depends on the connector build.** On a build with
`agiloft_group_by`, group the contract table by each field - two calls. On a
read-only build there is no aggregation tool, so **page through the contract
table once and count the distinct values yourself.** Apply the limit check on
every page. A few thousand records is three or four calls and it is worth it -
everything downstream depends on having the real names.

Do not narrate any of this.

**Match status names by meaning, not by string.** The groups below list typical
names, not an allowed list. `Lapsed` is Expired, `Void` is Cancelled, `Withdrawn`
is Rejected, `Live` is Active. If a name has no obvious equivalent, that is the
case to ask about rather than force into a group.

### Working out what "in flight" means here

There is no universal definition. Once you have the status list, classify it:

| Group | Typically |
| --- | --- |
| **In force** | Executed, Active, Signed - the contract is live |
| **In flight** | Anything mid-process: drafts, reviews, pending signatures, approvals |
| **Closed** | Cancelled, Terminated, Expired, Rejected |

If a status is ambiguous, **say so rather than guessing**. "I counted Approved as
in flight rather than in force - tell me if that's wrong for you" is a better
answer than a silently different number.

If the customer's own Agiloft report defines "in flight" differently, that
definition wins. Ask which statuses it includes when a figure needs to match
their dashboard.

### Fields that may or may not exist

Check before relying on any of these. None are guaranteed:

| Field | If present | If absent |
| --- | --- | --- |
| Days or hours in current status | Cycle-time panels work - **check the unit first**, see below | Say the knowledgebase doesn't track it. Do not estimate from dates unless asked, and label it as derived if you do |
| Contract value | Show money figures | Leave value columns and tiles out |
| Auto-renew flag | Mention it when answering "when does this terminate" | Do not imply a contract simply ends |
| Record source / import flag | Native vs imported splits work | Skip those panels |
| Assigned paralegal or reviewer | Workload panels work | Skip them |

**A missing field is a fact to report, not a gap to fill.**

### The unit on the ageing field

Some knowledgebases hold **days** in current status, others hold **hours**. The
field name usually says which. Read it before you use the number, because the
same threshold is a different figure in each: 90 days is `90` in one and `2160`
in the other.

Convert to days for anything shown to a person - *"a median of 161 days"*, not
*"a median of 3,869"*. State the unit in the source line so the conversion is
checkable. If the name doesn't settle it, look at the range: values in the
thousands across a contract population are hours, not days.

### Identifying NDA types

The NDA question depends on knowing which agreement types count as NDAs, and
that varies. From the discovered type list, take anything matching
non-disclosure, NDA, confidentiality, or a customer-specific variant. When
unsure, include it and say what you searched.

## When something goes wrong

Translate every error into something actionable. No stack traces, no HTTP codes,
no URLs.

| What happened | What to say |
| --- | --- |
| Auth rejected | "Agiloft turned down the sign-in. The connector's credentials may need refreshing — Legal Engineering can sort that." |
| Cannot reach Agiloft | "I can't reach Agiloft right now. If your Agiloft needs a VPN, connect and I'll try again." |
| Table or field not found | Check the real names, retry once. Then: "That isn't set up in your Agiloft." |
| Empty after checking field names | "Nothing in Agiloft matches that." |
| Query too broad, times out | Narrow it and say "that was too broad — here's the last 12 months instead." |
