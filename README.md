# back-office-agents

Claude skills for back-office workflows.

| Skill | What it does |
| --- | --- |
| [`legal/agiloft-skill`](./legal/agiloft-skill) | Answers Agiloft contract questions in plain language and keeps a live four-tab dashboard open beside the chat. Works against any Agiloft CLM knowledgebase — field names, status values and agreement types are discovered at runtime. |

## Status

**Pre-release.** The skill has been validated against synthetic knowledgebases
with known ground truth (185 automated checks, 100%). It has **not yet been run
against a live Agiloft instance**, so the tool call shapes are unproven against a
real deployment. See the Testing section in the skill's README for what is and
is not covered.
