# back-office-agents

Claude skills for back-office workflows.

| Skill | What it does |
| --- | --- |
| [`legal/agiloft-skill`](./legal/agiloft-skill) | Answers Agiloft contract questions in plain language and keeps a live four-tab dashboard open beside the chat. Works against any Agiloft CLM knowledgebase — field names, status values and agreement types are discovered at runtime. |

| Also here | |
| --- | --- |
| [`blog/`](./blog) | The publication blog for the Agiloft skill. Read the `.docx`; [`blog/REVIEW-NOTES.md`](./blog/REVIEW-NOTES.md) says what is settled and what is still open. |
| [`dist/`](./dist) | The installable `.skill` package, in sync with `legal/agiloft-skill/`. |

## Status

**Pre-release.** Validated against synthetic knowledgebases with known ground
truth — **185 automated checks, 100%**. Every figure is computed twice by code
paths that share nothing: once through a mock connector, once from the raw
records.

It has **not been run against a live Agiloft instance.** That proves the logic
and the arithmetic; it does not prove the tool call shapes against a real
deployment, and nothing has yet observed a model following the rules in a live
session.

We do not hold an Agiloft licence, so no live connector can be stood up here.
The dashboard images in the skill README are real renders against a dummy
knowledgebase; the Claude Desktop frames around them are reconstructions, and
are labelled as such.
