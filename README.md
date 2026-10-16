# mit-wired-hyperresearcher

The Claude Code skill and agent set behind the hyperresearch V8 deep-research pipeline, tuned to write final reports in an MIT Sloan / McKinsey / Wired house style: factual, credible, engaging, no arguments.

This repository holds the engine only. The research vault it runs against (notes, raw sources, the SQLite index) stays local and is excluded by `.gitignore`.

## Layout

- `.claude/skills/hyperresearch/` is the entry skill, a router over the 16 pipeline steps.
- `.claude/skills/hyperresearch-1-decompose/` through `hyperresearch-16-readability-audit/` hold one step each.
- `.claude/agents/hyperresearch-*.md` are the subagent prompts: fetcher, loci analyst, depth investigator, draft orchestrator, synthesizer, four critics, patcher, polish auditor, readability recommender, corpus critic, source analyst.
- `.claude/settings.json` registers the pre-tool hook the CLI uses.
- `CLAUDE.md` is the project instruction file for a vault that uses this engine.
- `STYLE-GUIDE.md` is the human-readable version of the report style the pipeline now enforces, with the patterns traced to their source articles.

## What changed from the stock V8 engine (October 2026)

- The argumentative voice is gone. "ARGUE, DON'T JUST REPORT" became "REVEAL, DON'T ARGUE". The `argumentative` response format is now `analytical`. Depth investigators write a calibrated `## Assessment` instead of a `## Committed position`. The three parallel drafts are breadth, depth and practitioner lenses, not thesis versus contrarian. Critics review for calibration and completeness; they do not attack. The polish auditor never strips an honest qualifier.
- Every final report follows a fixed frame: title with a verb and a stake, a one-sentence dek, `At a glance`, lede, `About the evidence`, finding-style body headings, `Open questions`, `Implications`, and a close on a quote or a factual stinger.
- Prose rules: paragraphs of 40 to 70 words, percentages with their base, events with their date, whole-sentence quotes from named people, bold only on run-in list leads, no em-dashes, no first or second person.

## Requirements

- Claude Code with the `hyperresearch` CLI installed (`pipx install hyperresearch`), and the CLI path in `CLAUDE.md` adjusted to the local machine.
- A vault directory containing `research/` and `.hyperresearch/`, created with `hyperresearch init`.

## Syncing

The vault directory is the git working tree. Only the engine files are tracked. After editing anything under `.claude/`, `CLAUDE.md` or `STYLE-GUIDE.md`:

```
git add -A && git commit -m "describe the change" && git push
```
