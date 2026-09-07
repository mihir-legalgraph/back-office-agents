# Blog — review notes

**Read:** `Solving Contract Lookup Bottlenecks Using a Claude Skill.docx` — that is
the deliverable. `blog.md` is the source it is built from; edit the markdown, not
the Word file, or the next build overwrites your changes.

Rebuild after editing `blog.md`:

```
cd blog && python build_docx.py
```

Needs `python-docx`. The architecture diagram is pulled from
`../legal/agiloft-skill/assets/architecture.png` at build time, so it never
drifts from the one in the skill README.

## Settled

- **Impact table** — now one row: building the four-tab dashboard takes **15–20
  minutes** from a single question. The "before" column carries no number
  deliberately; Agiloft has no view that answers all four questions side by side,
  so there is nothing to time against.
- **Screenshots** — the Claude Desktop frames in the skill README are
  reconstructions and ship that way. A live connector needs an Agiloft licence we
  do not hold. Every figure inside the frames is computed from the dummy
  knowledgebase, so the content is accurate even where the chrome is drawn.

## Open items — deliberate, not oversights

| Item | Where | Why it is open |
| --- | --- | --- |
| Repo link | *Deployment*, *Get Started* | The repo is private. Fills in at publication. |
| Contact link | *Get Started* | Needs the allNeurons contact URL. |

## What a reviewer should check

1. **The accuracy table is defensible.** 185 checks, 100%. Every figure is computed
   twice by code paths that share nothing — once through a mock connector, once
   from the raw records. 127 of those were re-run on 4 Sep (13 answers, 114
   rendered dashboard values, zero failures).
2. **The claims are hedged where they should be.** There has been no live Agiloft
   run. The piece says so explicitly under *Accuracy*; that paragraph should not
   be softened.
3. **No customer is identifiable.** The piece was scrubbed of customer names and
   customer business vocabulary. Worked examples come from synthetic
   knowledgebases and every counterparty in them is invented.
4. **The three worked examples are real.** 173 confidentiality agreements across
   three spellings, 57 found by exact match; a second entity with 39 more;
   250 records in the expiry window against 87 actually in force. All verified
   against the test data, not illustrative.

## Format

allNeurons variance-analysis house format: kicker and byline, problem, pull-quote,
then Solution, Deployment, Architecture as numbered steps, Where This Has Helped,
Impact, What's Next, Get Started.
