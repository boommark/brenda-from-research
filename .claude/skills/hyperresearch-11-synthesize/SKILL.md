---
name: hyperresearch-11-synthesize
description: >
  Step 11 of the hyperresearch V8 pipeline. Reads the 3 angle-specific drafts
  from step 10, spot-checks factual conflicts, writes a synthesis plan +
  outline, then spawns ONE hyperresearch-synthesizer subagent (Read+Write
  tool-locked) that writes the final report in TWO passes — pass 1 rough
  integrated draft, pass 2 voice/redundancy/length cleanup. Skipped for
  light tier (which writes a single draft directly in step 10). Invoked
  via Skill tool from the entry skill (full tier).
---

# Step 11 — Synthesize the final report

**Tier gate:** SKIP entirely for `light` tier — light tier wrote `research/notes/final_report_<vault_tag>.md` directly in step 10 and proceeds straight to step 15 (polish). For `full`: run as documented below.

**Goal:** turn the 3 angle-specific drafts from step 10 into ONE integrated final report at `research/notes/final_report_<vault_tag>.md`. The orchestrator preps the strategic brief; the synthesizer subagent writes the report in two passes (rough integrated draft, then voice/redundancy/length cleanup).

**Why split orchestrator + synthesizer:** the orchestrator has been running for 30+ minutes and 200K+ tokens of context. Writing a coherent six-to-seven-page report in one human voice, plus its appendix, is the highest cognitive load step in the pipeline, and orchestrator context is full of stale subagent dispatch logic. The synthesizer is a fresh Opus session with `[Read, Write]` tool-lock, focused exclusively on producing the final report. This is the same architectural move that made the patcher and polish-auditor reliable.

---

## Recover state

Read these inputs:
- `research/scaffold.md` — vault_tag
- `research/prompt-decomposition.json` — atomic items, required_section_headings, response_format, citation_style
- `research/temp/draft-a.md`, `research/temp/draft-b.md`, `research/temp/draft-c.md` — the 3 angle-specific drafts from step 10
- `research/comparisons.md` (full tier) — cross-locus tensions
- `research/temp/source-tensions.json` (full tier) — expert disagreements
- `research/temp/evidence-digest.md` — load-bearing claims with verbatim quotes
- `research/query-<vault_tag>.md` — canonical research query (GOSPEL)

---

## Step 11.1 — Read all 3 drafts in full

1. **Read each draft in full** from `research/temp/draft-{a,b,c}.md`. Don't skim — actually read. Hold them in context.

2. **Re-read each sub-orchestrator's report-back** (from your own task results in step 10). Note each draft's:
   - Core findings
   - How many notes from `must_read_note_ids` it actually read
   - Strongest finding and the most contested point
   - Word/character count

---

## Step 11.2 — Spot-check factual conflicts (orchestrator only)

The synthesizer is tool-locked to `[Read, Write]` — it cannot run Bash to query the vault. So YOU resolve factual conflicts here, before spawning it.

For each substantive contradiction between drafts:
1. Identify the cited source IDs on both sides
2. `$HPR note show <id1> <id2> -j` to read the actual source bodies
3. Decide which side is correct. Write the verdict to `research/temp/synthesis-conflicts.md`:
   ```markdown
   ## Conflict 1: <one-line description>
   - Draft A says: <claim with citation>
   - Draft B says: <opposing claim with citation>
   - Source check: <what the source actually says, verbatim where possible>
   - **Verdict:** <which side, with reason>
   ```

If there are no substantive conflicts, write a one-line file: "No factual conflicts found across drafts."

---

## Step 11.3 — Write the synthesis plan

Write `research/temp/synthesis-plan.md`. This is your strategic brief for the synthesizer:

```markdown
# Synthesis plan

## Core findings (2-4 sentences)
<the direct answer to the research_query as the evidence supports it, with confidence>

## The 3-7 most important findings
1. **<short name>** — sourced from Draft <A/B/C>. <one sentence on the finding, its evidence base and confidence>
2. ...

## Section structure
<list required_section_headings if present, OR the inferred H2 structure>

## Per-section commitments
### Section 1: <heading>
- Evidence to pull from: Draft A's <topic>, Draft C's <topic>
- Key finding: <which assessment this section presents, with its confidence>
- Cross-locus dynamic to present (if any): <name from comparisons.md>

### Section 2: ...

## Where drafts disagreed
- **<Disagreement 1>:** Draft A says X; Draft B says Y. **Resolution:** <if the source check settles it, state which is correct and why; if the evidence genuinely leaves it open, the report presents both with their evidence and says it is unsettled; if a claim is dubious, cut it>.
- ...

## Length target
- response_format: <short|structured|analytical>
- length_budget_words: <from the decomposition, default 3000>; long_form: <true|false>
- Pass 1 target: <budget plus up to 20 percent>
- Pass 2 final target: <within budget; body only, excluding Notes, Further reading and the appendix>

## Appendix plan
<which tables, per-entity profiles and matrices from the three drafts go to research/notes/appendix_<vault_tag>.md, and the one-sentence pointer each leaves in the body>

## Controlling finding
<one sentence a reader could repeat at lunch; this is the first sentence of the report's lead and the thing the closing returns to>
```

---

## Step 11.4 — Write the synthesis outline

Write `research/temp/synthesis-outline.md`. This is the per-section contract — 1-2 sentences per H2 naming what evidence and findings live there:

```markdown
# Synthesis outline

## Title and italic line
<a descriptive title naming subject and scope; an italic line with audience and evidence cut-off date>

## TL;DR
<six to eight bullets, each a bold lead of two to eight words then the evidence with its marker; first bullet the controlling finding; one bullet the opening or decision the evidence points at; last bullet any correction to the brief's premises>

## Lead (no heading in the report)
<two paragraphs: the controlling finding restated as sentence one with the one or two facts that most shape the decision; then what the body covers and what the appendix holds>

## Key numbers
<four to six figures, each with a five-to-eight-word meaning>

## 1. <First body H2 as a short noun-phrase topic>
<1-2 sentences: the claim this section opens on, which evidence anchors it, the named instances, what contested point lives here, and which structural device carries the evidence (bullets with bold leads, a table within eight by five, bold sub-headings); H3s only if two genuinely separate parts>

## 2. <Second body H2>
<1-2 sentences>

... (at most six body H2s; if the brief named more topics, say which are grouped together and which go to the appendix)

## What would change this reading (callout)
<three to five bullets: the event, document or number, and what it would shift>

## Options (only when the query asks about choices)
<each option as a bold lead on its own line with two indented bullets, "Precedent" and "Against it"; described, never ranked; a summary table if more than three; a dates-to-watch table if the subject has a calendar>

## Closing
<one or two short paragraphs: return to the controlling finding, add the one new or unresolved thing; bullets where the evidence concentrates in a few places; may end on a sourced quote because it carries evidence; never a stinger or a summary sentence>

## Notes
<footnotes, one per claim, carrying the source and the [[note-id]] (wikilink) or URL (inline)>

## Further reading
<eight to fifteen links under two or three plain headings with a one-line note each; the appendix file first>
```

The outline is short (80-250 words total). It's the structural anchor that prevents pass-1 sections from rambling and keeps the body inside the budget.

---

## Step 11.5 — VERIFICATION GATE

Before spawning the synthesizer, verify these files exist with non-trivial content:
- `research/temp/synthesis-plan.md` — must include the core thesis and at least one per-section commitment
- `research/temp/synthesis-outline.md` — must include one outline entry per H2 in the planned structure
- `research/temp/synthesis-conflicts.md` — exists (may say "no conflicts found")
- `research/temp/draft-{a,b,c}.md` — all three exist

If any are missing or trivial, fix them before proceeding. The synthesizer cannot do strategic planning — it can only execute the plan. Skipping plan/outline produces a thin synthesizer output that doesn't beat the original drafts.

---

## Step 11.6 — Spawn the synthesizer

Spawn ONE `hyperresearch-synthesizer` subagent. Single spawn, runs once.

**Spawn template:**
```
subagent_type: hyperresearch-synthesizer
prompt: |
  RESEARCH QUERY (verbatim, gospel):
  > {{paste research/query-<vault_tag>.md body}}

  QUERY FILE: research/query-<vault_tag>.md

  PIPELINE POSITION: You are step 11 of the hyperresearch V8 pipeline.
  Step 10 produced 3 angle-specific drafts. The orchestrator wrote a
  synthesis plan and outline. You read everything and write the final
  report in TWO passes (pass 1 = rough integrated draft, pass 2 = voice/
  redundancy/length cleanup). You are tool-locked to [Read, Write] — you
  cannot Bash, cannot spawn subagents. After you return, step 12 (4
  critics) reads your final report.

  YOUR INPUTS:
  - query_file_path: research/query-<vault_tag>.md
  - draft_paths: [research/temp/draft-a.md, research/temp/draft-b.md, research/temp/draft-c.md]
  - synthesis_plan_path: research/temp/synthesis-plan.md
  - synthesis_outline_path: research/temp/synthesis-outline.md
  - synthesis_conflicts_path: research/temp/synthesis-conflicts.md
  - decomposition_path: research/prompt-decomposition.json
  - comparisons_path: research/comparisons.md
  - source_tensions_path: research/temp/source-tensions.json
  - evidence_digest_path: research/temp/evidence-digest.md
  - pass1_output_path: research/temp/synthesis-pass1.md
  - final_output_path: research/notes/final_report_<vault_tag>.md
  - appendix_output_path: research/notes/appendix_<vault_tag>.md
  - style_guide_path: STYLE-GUIDE.md
  - response_format: "<short|structured|analytical>"
  - citation_style: "<wikilink|inline|none>"
  - length_budget_words: <from the decomposition, default 3000>
  - long_form: <true|false>

  Read STYLE-GUIDE.md, then everything else. Write pass 1 to
  pass1_output_path, drafted warm (first person). Then audit pass 1 for
  redundancy, voice, stance, weak sections and body length, strip the
  first person, and write the cleaned pass 2 to final_output_path and
  the reference material to appendix_output_path. Do not paste
  paragraphs from the input drafts; synthesize them in your own voice.

  **Citation rendering:**
  - If citation_style == "wikilink" (default): body prose carries
    `[^N]` footnote markers, one per claim, after the verb or noun it
    supports, never two in a sentence. The `## Notes` block after the
    closing lists each as `[^N]: Author or organisation, "Title", date,
    [[note-id]]`. The wiki-link inside the footnote IS the vault
    citation; readers click through to the source note's frontmatter
    for title + URL. No wiki-links in body sentences.
  - If citation_style == "inline": the same `[^N]` markers, and the
    `## Notes` entries carry `Author. "Title." Publication, Year. URL`
    (read each cited note's YAML frontmatter for title + URL).
  - If citation_style == "none": no citation markers anywhere, no
    Notes block.
```

**CRITICAL: never emit bare text while the synthesizer is running.** It will take 5-15 minutes (two passes). Use the wait time to think — append notes to `research/temp/orchestrator-notes.md` about what you'll watch for in step 12 (the critics) based on the synthesis plan you just wrote.

---

## Step 11.7 — Validate the synthesizer output

When the synthesizer returns:

1. **Confirm the files exist:**
   - `research/temp/synthesis-pass1.md` (pass 1, rough integrated)
   - `research/notes/final_report_<vault_tag>.md` (pass 2, final)
   - `research/notes/appendix_<vault_tag>.md` (reference material; may be short if little was moved, but it should exist for full tier)

2. **Read the synthesizer's report-back.** It tells you:
   - Body word count against `length_budget_words`
   - Pass 1 vs pass 2 length delta (should be NEGATIVE; pass 2 cuts)
   - What moved to the appendix
   - Top redundancies cut
   - Top voice fixes
   - Sections flagged as still weak

3. **Sanity checks on the final report:**
   - Body (title through closing, excluding Notes, Further reading and the appendix) is within `length_budget_words` plus ten percent; `short` format within its own range
   - `## TL;DR` directly under the italic line, six to eight bold-led bullets, first bullet the controlling finding; lead opens on the controlling finding (not "This report examines"); key-numbers table present; at most six numbered noun-phrase body H2s covering every `required_section_headings` topic; one `### What would change this reading` callout; `## Options` only if the query asked about choices; a closing; `## Notes`; `## Further reading` with the appendix first
   - Citations match `citation_style`: `[^N]` markers in the body and a `## Notes` block carrying `[[note-id]]` for `"wikilink"` or URLs for `"inline"`; no markers for `"none"`; no `[[...]]` in body sentences; no chains of markers; every marker resolves
   - No "Confidence is" labels, no per-section settle-it lines, no glossary paragraph, no first person
   - No body table over eight rows or five columns
   - No body section with three or more consecutive prose paragraphs; no paragraph over 90 words; every bullet opens with a bold lead
   - No YAML frontmatter, no scaffold leaks, no pipeline vocabulary

If pass 2 is longer than pass 1 (positive delta), something went wrong — pass 2 is supposed to cut. Investigate before proceeding.

If a sanity check fails, hand-craft an Edit on `research/notes/final_report_<vault_tag>.md` yourself to fix it. Do NOT re-spawn the synthesizer — that's regeneration, which violates the patch-not-regenerate invariant once we have a final draft.

---

## Write-once after synthesis

After this step, the final report is only modified by Edit hunks from the patcher (step 14) and polish auditor (step 15). Do NOT re-write or re-synthesize.

---

## Exit criterion

- `research/notes/final_report_<vault_tag>.md` exists, body within `length_budget_words`
- `research/notes/appendix_<vault_tag>.md` exists and is linked from Further reading
- `research/temp/synthesis-pass1.md` exists (debugging artifact)
- Every `required_section_headings` topic covered by the body H2s (at most six)
- Citations match `citation_style` (`[^N]` markers plus a `## Notes` block; wikilink entries carry `[[note-id]]`, inline entries carry URLs; none means no markers)
- No YAML frontmatter, no pipeline vocabulary, no scaffold leaks, no calibration labels

---

## Next step

Return to the entry skill (`hyperresearch`). Invoke step 12:

```
Skill(skill: "hyperresearch-12-critics")
```
