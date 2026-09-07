*Legal Ops & Automation Blog | 7 MIN READ*

Mihir Sahu, 2 Sep 2026

Contract questions arrive from everywhere. A salesperson wants to know whether they can send a deck to a prospect, or whether an NDA needs signing first. A finance lead wants to know what renews this quarter. A business requester wants to know where their own request has got to. A GC wants to know what is holding the pipeline up.

All of that information is already in Agiloft. The problem is who can get it out. Agiloft's reporting is built for people who use Agiloft — the paralegals and Legal Ops team who know which dashboard to open, which filter to apply, and what the status names actually mean. Most of the people asking these questions are not those people. They ask a paralegal, who stops what they are doing, logs in, applies a filter, reads a number off a widget, and types it back into Slack.

The answer usually takes under a minute to find. The interruption costs far more than that, and it happens many times a day.

There is a second problem underneath the first, and it is the more serious one. Some of these questions are easy to answer *wrongly*. "Does this company already have an NDA with us?" looks like a lookup. It is not. Counterparty names are filed inconsistently — the same company sits under "ABC Systems", "abc systems " and "ABC SYSTEMS" — so a careless search finds a fraction of the records and reports a confident "no". Someone then shares confidential material believing no NDA is needed. Search too loosely instead and you sweep in "ABC Holdings", a different legal entity, and report cover that does not exist.

> ***Legal teams needed a way for non-Agiloft users to get contract answers themselves — in plain language, with the number traceable back to the query that produced it, and without the failure modes that make a wrong contract answer worse than no answer at all.***

# The Solution

We built a Claude skill that sits on top of an existing Agiloft MCP connector. Someone asks a question in plain English; they get an answer in chat and a live dashboard beside it, in four tabs the team already recognises — Contract Reporting, Contract Requester, Cycle Times Reporting, Vendor Management.

Three design decisions shaped everything else.

**The answer is complete, and then it stops.** Not a fixed length — a good answer to "how many contracts are in flight" is one sentence, and a good answer to "what is slowing us down" is four, because the reasoning *is* the answer. What it never does is pad, and it never hands the reader arithmetic to finish. "Intake holds 18% of the backlog against 18% of the in-flight population" is not an answer. "Intake has the most stuck contracts, but only because it is the busiest stage — it is not actually slower than the others" is.

**Every figure names where it came from.** Not "from Agiloft" — that is the name of the system, not a source. The table, the fields filtered on, and the values used, in a clause at the end of the answer: *"…Contract table, `expiry_on` between today and 24 November with `wf_state` = Live."* Someone should be able to reproduce the number in Agiloft from what they were told. When two people get different numbers for the same question, the filter is always why, and naming it up front is what stops that argument.

**Nothing about any particular Agiloft is built in.** Field names, status values and agreement types are discovered at the start of each session, from the knowledgebase itself. There is no configuration file to fill in and no mapping to maintain, because there is nothing to map.

Most of the skill is not about fetching data. It is a set of rules about the specific ways a contract number goes wrong.

# Deployment

To run the skill you will need:

- An **Agiloft MCP connector**, installed and connected, with read permissions on the contract table. An Agiloft login on its own is not enough — the connector is a separate component, usually maintained by whoever administers Agiloft for your team.
- **Claude Cowork** on a Pro, Team or Enterprise plan.
- The **allNeurons Agiloft dashboards skill**.

Install the skill from the GitHub repository *(repo link — pending publication)* and invoke it in Cowork. There is no setup form and no configuration file. If the connector is live, the skill is ready.

**One thing to decide before rolling it out.** The connector authenticates as a single Agiloft account, and Agiloft applies that account's permissions to everything the skill reads. On individual seats, two people can ask the same question and get different numbers — and both be right. On a shared service account, everyone sees the same view, which is usually wider than the person reading the screen would have in Agiloft. Neither is wrong; it is a decision worth making deliberately rather than discovering later. The skill puts the account it is reading as into the dashboard header, so anyone looking at a screenshot knows whose view it is.

# Architecture

The `SKILL.md` file is the operating procedure. It turns the paralegal's manual lookup into the following automated steps:

1. **Step 1 — Detect the connector build.** Agiloft MCP connectors vary. Some expose three read tools; others add aggregation tools on top. The skill checks which are present and uses what is there. It never calls the create, update or delete tools, even when they exist and even if asked to.

2. **Step 2 — Learn the knowledgebase.** Reading one contract record gives the real field names, which differ from the labels shown in the UI. The status values and agreement types actually in use are read either through the aggregation tools or, on a read-only build, by paging the contract table once and counting. Every step after this works from what was found.

3. **Step 3 — Classify the statuses by meaning.** "In flight" has no universal definition. Once the status list is known, the skill sorts it into in force, in flight and closed by what each value means rather than by string match — `Lapsed` is Expired, `Void` is Cancelled, `Live` is Active. Where a status is genuinely ambiguous it says which way it counted rather than producing a silently different number.

4. **Step 4 — Read the question, not just the topic.** "How many" returns a number. "What are" and "which" return records as a table. A count is an addition to a list, never a substitute for one — answering "what are the agreements terminating this quarter" with a total is technically responsive and practically useless.

5. **Step 5 — Fetch, then check the fetch.** Every request is compared against the page limit it asked for. If a query asking for 500 records returns exactly 500, records were missed, and the skill refetches higher before that number reaches a tile. This is the single most likely way a page size gets reported as a company total.

6. **Step 6 — Bound time ranges properly.** Anything "over the last N months" is queried one month at a time. Pulling the most recent records and bucketing them by month makes the early months look empty and invents an upward trend that is not in the data.

7. **Step 7 — Search companies loosely, then split the results by entity.** Loose search is what prevents a false "no" on the NDA question. Splitting the results back out by counterparty before counting is what prevents a false "yes" from a similarly-named company. Both halves are necessary; either one alone is dangerous.

8. **Step 8 — Check the unit before reporting a duration.** Some knowledgebases store days in current status; others store hours. Ninety days is `90` in one and `2160` in the other, and a median of `3,869` reported as days is off by a factor of twenty-four. The skill reads the unit off the field, converts for display, and says which it used.

9. **Step 9 — Report missing fields as facts.** The Agiloft contract record holds metadata — parties, dates, type, status, owner. Payment terms, liability caps and governing law live in the signed document, unless the knowledgebase runs Agiloft's Extract Key Terms action, which writes model-extracted terms back onto the record. The skill checks for those fields, uses them where they exist and labels them as extracted rather than lawyer-entered — and where they do not exist, says what the record does not contain. A missing field is a fact to report, not a gap to fill with plausible prose.

10. **Step 10 — Build the dashboard and attach the source lines.** All four tabs are built every time; the one that answers the question asked is the one that opens. Every panel carries its filter, its record count, and its reconciliation against the corresponding Agiloft widget. On a follow-up — "just the NDAs", "what about last quarter" — the same dashboard re-scopes in place. A dashboard still showing company-wide figures while the chat discusses one vendor is exactly how a wrong number gets screenshotted and forwarded.

The person asking still owns the call. The skill produces a traceable answer; the reader decides whether to act on it, and the source line is there so they can check.

# Where This Has Helped

These are measured against synthetic knowledgebases where every correct answer was known in advance. Each one is a mistake the obvious implementation makes.

**A false "no" caught before it shipped.** One knowledgebase holds 173 confidentiality agreements for a single counterparty, spread across three spellings of its name — `Brightline Systems`, `BRIGHTLINE SYSTEMS`, and `brightline systems ` with a trailing space. An exact-string search returns 57 of them. Had none of those 57 been in force, the honest-looking answer would have been "no NDA on file" — the highest-consequence error this skill can make. Loose search finds all 173.

**A false "yes" caught the same way.** The same dataset contains a second company with a similar name — Brightline Holdings — and 39 confidentiality agreements of its own, 5 of them in force. Loose search alone folds both into one answer: 30 in force, when the true figure for the company actually asked about is 25. Splitting the results by entity before counting fixes it, and the answer names the other company rather than absorbing it.

**A 187% overstatement, explained rather than hidden.** Asked what is expiring in the next 90 days, that knowledgebase returns 250 records carrying an end date in the window — but only 87 are contracts in force. The other 163 are unsigned or already cancelled, so nobody has to renew them. Reporting 250 as "expiring" nearly triples the real figure. The skill reports the in-force number, states the scope, and gives the wider figure alongside rather than picking one and staying quiet.

**An ageing field read as the wrong unit.** One knowledgebase stores time-in-status in hours rather than days. A median of 3,869 is 161 days, not five and a half years. Reading the unit off the field and converting for display is the difference between a usable cycle-time chart and a nonsensical one.

# Impact

Previously, answering a contract question meant:

**Ask a paralegal → they stop work → log into Agiloft → find the right dashboard → apply the right filter → read the widget → type the number back into Slack**

With the skill:

**Ask in plain English → Claude → Agiloft MCP → fetch and check → an answer with its source + a live dashboard → the reader verifies against Agiloft if they want to**

| Metric | Before | With the Skill | Improvement |
| --- | --- | --- | --- |
| Time to answer a routine contract question | *TBD* | *TBD* | *TBD* |
| Paralegal interruptions per week | *TBD* | *TBD* | *TBD* |

*[To be measured during the live pilot. We have deliberately not estimated these — the accuracy numbers below are measured, and mixing them with guesses would undermine both.]*

### Accuracy

Testing ran against synthetic Agiloft knowledgebases where every correct answer was known in advance, through a stand-in connector matching a real one's behaviour: the same tools, the same 500-record page limit, and the same absence of a total count.

| What was tested | Checks | Result |
| --- | --- | --- |
| Answers, knowledgebase A | 30 | all correct |
| Dashboard figures, knowledgebase A | 28 | all correct |
| Answers, knowledgebase B — a different schema | 13 | all correct |
| Every value rendered on the dashboard, knowledgebase B | 114 | all correct |
| **Total** | **185** | **100%** |

Every figure is computed twice by independent code paths — once through the mock connector, once directly from the raw records — and compared. The paths share no code, so a common bug cannot hide behind a matching answer.

Knowledgebase B deliberately shares **no field name, status value or agreement type** with knowledgebase A. A skill carrying hardcoded names scores zero on it. Both datasets carry deliberate traps: closed contracts holding end dates, a month with no records at all, the same vendor filed under several spellings, a different company with a similar name, blank counterparties, an ageing field measured in hours, and a company with no NDA whatsoever.

Three further knowledgebases test what happens when the shape of the data changes rather than its labels:

- **Fields removed entirely** — no ageing, no owner, no contract value. The affected panels come back empty with a stated reason. Nothing is estimated to fill the space.
- **AI-extracted term fields** present on some records and not others, as Agiloft's Extract Key Terms action produces. Answers drawn from them are labelled as extracted by a model rather than entered by a lawyer.
- **Business-specific choice fields** with names appearing in no Agiloft documentation, alongside a free-text field holding 1,719 distinct values. The skill finds the choice fields, builds panels from them, and leaves the free-text field alone — grouping by it would produce a chart with 1,719 bars.

**What testing has not yet covered:** this ran against a mock connector, not a live Agiloft instance. It proves the logic and the arithmetic; it does not prove the tool call shapes against a real deployment, and it does not measure how consistently a model follows the rules in a live session as opposed to whether the rules are right. Both are on the list for the pilot.

# What's Next

1. **Live validation.** Run the full question set against a real Agiloft knowledgebase with the Agiloft dashboards open alongside, and reconcile every figure against the corresponding widget.

2. **Reading the contract document.** Where a knowledgebase does not run Extract Key Terms, questions about payment terms, liability caps and governing law cannot be answered from the record, and the skill says so. Reading the attached signed document would close that gap — and it is the most-requested capability by some distance.

3. **Behavioural evaluation.** Accuracy so far measures whether the rules produce the right number. The next layer measures how reliably a model follows those rules under live conditions — particularly the limit check and the entity split, which are the two that fail quietly.

4. **Learning from corrections.** When a paralegal catches a figure that does not match their own report, feed the definition back in — most such gaps are a difference in which statuses count as "in flight", and that is a knowable, fixable thing.

# Get Started

Install the skill from this GitHub repo — *(repo link — pending publication)*. It works against any Agiloft CLM knowledgebase; status names, agreement types and field names are discovered at runtime rather than configured. Ask it a question you already know the answer to, and check the source line against your own Agiloft report before relying on it for one you don't.

To get this customised and deployed for your specific use case, contact us here *(contact link — pending)*.
