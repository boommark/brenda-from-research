---
name: brenda
description: Brenda from Research. Alias for the /hyperresearch deep-research pipeline. Use for any open research question that should end in a six-to-seven-page decision brief with footnotes, further reading and a linked appendix.
---

# Brenda from Research

`/brenda <question>` is the same as `/hyperresearch <question>`. Brenda is the name of the engine; hyperresearch is the CLI and vault it runs on.

Do exactly this:

1. Take the user's arguments verbatim as the research query. Do not rephrase them.
2. Invoke the `hyperresearch` skill with those arguments, using the Skill tool.
3. Follow that skill's chain end to end. Every step, artifact path, tier rule and subagent contract is owned by the `hyperresearch` entry skill and the `hyperresearch-N-*` step skills.
4. The final brief follows `STYLE-GUIDE.md` at the vault root: a `## TL;DR` of six to eight bold-led bullets with the controlling finding first, a two-paragraph lead, at most six numbered noun-phrase sections each carrying its evidence in bullets with bold leads or a table (never three prose paragraphs in a row), one "what would change this reading" callout, options described not ranked with a "Precedent" and an "Against it" bullet each, calibration in words, one footnote per claim, 2,500 to 3,500 words of body unless the user asks for a long document, then `## Notes`, `## Further reading`, and the linked appendix file.

If the user says "ask Brenda", "get Brenda on this" or similar, treat it as this skill.
