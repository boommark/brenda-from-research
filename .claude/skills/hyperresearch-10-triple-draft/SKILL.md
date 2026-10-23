---
name: hyperresearch-10-triple-draft
description: >
  Step 10 of the hyperresearch V8 pipeline. Orchestrator pre-curates 20-50
  angle-specific source IDs per draft, then spawns 3 hyperresearch-draft-
  orchestrator subagents in parallel — each reads its curated list (no
  vault surveys, no source-fetching) and writes one angle-specific draft.
  This step ENDS when all 3 drafts are written and validated. Step 11
  (synthesizer) handles the synthesis-write that produces the final report.
  For light tier: writes a single draft directly to final_report.md and
  skips ahead to step 15 (polish). Invoked via Skill tool.
---

# Step 10 — Triple-draft ensemble (curated lists, parallel writers)

**⚠ CRITICAL ANTI-PATTERN: Writing a single draft for `full` tier is a PIPELINE VIOLATION.** In V7 runs, context compaction caused the orchestrator to forget this step's procedure and write a single draft instead of spawning 3 sub-orchestrators. V8 fixes this by loading this skill fresh at the moment it's needed. **If you find yourself about to write `research/notes/final_report_<vault_tag>.md` directly without spawning 3 `hyperresearch-draft-orchestrator` subagents, STOP. Re-read this skill. Spawn the sub-orchestrators.** (Light tier is the ONE exception — see "Light tier" section below.)

**Tier gate:** Runs for ALL tiers. For `light` tier: write a single draft directly to `research/notes/final_report_<vault_tag>.md` and skip ahead to step 15 (polish). For `full`: run the triple-draft ensemble below — step 11 (synthesizer) will turn the 3 drafts into the final report.

**Goal:** produce THREE independent angle-specific drafts (`draft-{a,b,c}.md`). Step 11 (synthesizer subagent) consumes all three and writes the final report.

---

## Recover state

Read these inputs:
- `research/scaffold.md` — vault_tag, modality, wrapper requirements
- `research/prompt-decomposition.json` — atomic items, required_section_headings, response_format, citation_style, pipeline_tier
- `research/temp/evidence-digest.md` — top claims + verbatim quotes — PRIMARY EVIDENCE LAYER (full only; absent for light)
- `research/comparisons.md` (full tier) — cross-locus tensions
- `research/temp/source-tensions.json` (full tier) — expert disagreements
- `research/temp/coverage-gaps.md` (if exists) — items with weak source coverage
- Survey vault: `$HPR search "" --tag <vault_tag> -j` for the evidence landscape
- Modality calibration (from the scaffold's `modality` field):
  - **collect** — enumerative coverage, per-entity sections with named fields
  - **synthesize** — defended thesis with evidence chains, interpretive density
  - **compare** — proportionate per-entity depth + a committed recommendation
  - **forecast** — predictive claims grounded in past + present, explicit time horizon

---

## Step 10.0 — Read response_format and citation_style

Read `response_format`, `citation_style`, `length_budget_words` and `long_form` from `research/prompt-decomposition.json`. These control the draft shape:

| Format | Body length | Character |
|--------|-------------|-----------|
| `"short"` | 500–2000 words / 1500–6000 chars (CJK) | Direct answer, compact |
| `"structured"` | Final body within `length_budget_words` (default 3,000); drafts may run to 1.5x before their `## Appendix material` section | Scannable, breadth-first, tables within eight rows by five columns |
| `"analytical"` | Final body within `length_budget_words` (default 3,000); drafts may run to 1.5x before their `## Appendix material` section | Decision support written as a human expert would (STYLE-GUIDE.md) |

**Length discipline:** the shipped body is six to seven pages. Reference material (entity tables, matrices, per-account profiles) is unbudgeted but lives under `## Appendix material` in each draft and in `research/notes/appendix_<vault_tag>.md` in the final output, never in the body.

---

## Light tier ONLY: single-draft path

If `pipeline_tier == "light"`: SKIP step 10.1 — 10.3 below and follow this section instead.

**Light tier writes a single draft directly to `research/notes/final_report_<vault_tag>.md`.** No subagents, no triple-draft ensemble, no synthesizer.

1. **Read the vault directly.** Light tier has no `evidence-digest.md` (step 9 was skipped). Survey the vault: `$HPR search "" --tag <vault_tag> -j` and pick the 8–15 most relevant non-deprecated notes. Read each one (`$HPR note show <id1> <id2> ... -j`) before writing.

2. **Honor the structural contract.** Read `STYLE-GUIDE.md` at the vault root first.
   - Use the H2 topics from `required_section_headings` in `research/prompt-decomposition.json`, in order, as the numbered body sections (at most six; group if there are more and let the appendix keep the full order).
   - Wrap them in the report frame: a descriptive title naming subject and scope, an italic line with audience and evidence cut-off date, a `## TL;DR` of six to eight bold-led bullets (first bullet the controlling finding, one bullet the opening the evidence points at, last bullet any premise correction), a lead of two paragraphs whose first sentence restates the controlling finding (no heading, never "This report examines"), a key-numbers table of four to six figures, the numbered noun-phrase body sections with the finding in each section's first paragraph and the evidence below it in bullets with bold leads or a table, one `### What would change this reading` callout (three to five bullets), `## Options` only when the query asks about choices (bold run-in option on its own line, then two indented bullets "Precedent" and "Against it"; described, never ranked), a short `## Closing`, then `## Notes` (footnotes) and `## Further reading` (eight to fifteen links with a one-line note each; the first links the appendix file if one exists). For `short` format the frame collapses to a lead and a closing.
   - House prose: open each section on a claim worth testing; each paragraph is an assertion, two or three named and dated instances, and a closing sentence with the strongest word last; alternate long evidence sentences with short judgment sentences; anchor every abstraction with a named instance; one figure per sentence paired with its meaning; calibration in words inside the sentence (never "Confidence is medium", never a per-section settle-it line); define terms at the point of need; quotes as whole sentences from named people; even stance (observation, not verdict); bold only on run-in list leads, option names and sentence-case sub-headings on their own line; no em-dashes; no first or second person in the shipped text; no metaphors, hinges or stingers; "operator" rather than "telco" outside quotes. Omit dubious claims rather than defend them.
   - Structure for the skim: never three prose paragraphs in a row; no paragraph over 90 words; three or more parallel instances become bullets with bold leads; a quotation gets its own bullet with the speaker's name and role in the lead; a reader of only the TL;DR, the bold leads and the tables gets the finding and its evidence.
   - Body within `length_budget_words` (default 3,000). Tables over eight rows or five columns, per-entity profiles and matrices go to `research/notes/appendix_<vault_tag>.md` with a pointer sentence in the body.
   - Apply the modality calibration from the recover-state list above.

3. **Citations.** Three styles:
   - `"wikilink"` (default for non-wrapped runs): `[^N]` footnote markers in the body, one per claim, after the verb or noun it supports, never two in one sentence. A `## Notes` block after the closing lists each as `[^N]: Author or organisation, "Title", date, [[<source-note-id>]]`. The wiki-link inside the footnote resolves to the source note's frontmatter (title + URL). This is the navigable-vault format.
   - `"inline"` (benchmark + public deliverables): the same `[^N]` markers; `## Notes` entries carry `Author. "Title." Publication, Year. URL` (read each cited note's YAML frontmatter for title + URL).
   - `"none"`: no citation markers anywhere, no Notes block.

4. **Hygiene.** No YAML frontmatter on the final report. No pipeline vocabulary in prose ("hyperresearch", "evidence digest", "comparisons.md", "committed reading", etc.). When `citation_style == "wikilink"`, the `[[<source-note-id>]]` links inside `## Notes` ARE the citation system and must be preserved; only strip wikilinks that point at workspace artifacts (interim-*, scaffold, comparisons). Step 15 (polish) is a backstop, not a license to leak.

5. **Exit and route.** Once `research/notes/final_report_<vault_tag>.md` is written, return to the entry skill and invoke `Skill(skill: "hyperresearch-15-polish")`. Light tier skips steps 11–14 entirely.

---

## Step 10.1 — Define 3 analytical angles (full tier)

Based on the evidence, tensions, and query, assign each sub-orchestrator a distinct angle. The angles should produce genuinely different drafts — not three versions of the same report. The angles are complementary lenses on the same evidence, NOT opposing sides of a debate. No draft is asked to defend a thesis.

- **Draft A — Breadth-optimized:** widest possible coverage of all atomic items. Every finding the corpus supports, with its confidence.
- **Draft B, depth-optimized:** deeper treatment of the 3-4 most important or most contested atomic items. Where credible sources disagree, lay out the evidence on each side in prose, say which has the better evidence and why, and send the open points to the callout. Read the minority-view and methodological-critique sources so their evidence is represented fairly.
- **Draft C, practitioner-optimized:** organized around what a decision-maker needs: comparison tables within the eight-by-five cap, boundary conditions, applied cases, the forward view. Options described with their precedent, requirement and obstacle, not recommendations.

All three drafts read `STYLE-GUIDE.md` and write in its register: six-to-seven-page body, footnotes, appendix material separated. Tell each sub-orchestrator the `length_budget_words` and `long_form` values.

Write the 3 angle assignments to `research/temp/draft-angles.md` (for the run log). Each angle: 2-3 sentences describing the analytical direction.

---

## Step 10.2 — Curate per-angle source lists

**Critical step.** Each draft sub-orchestrator does NOT decide what to read. YOU pick the 20-50 most relevant vault notes for each angle and pass them as `must_read_note_ids`. This eliminates wasted vault-survey loops in the sub-orchestrators and forces real differentiation by giving each draft a different evidence base.

1. **List all substantive vault notes:**
   ```bash
   $HPR search "" --tag <vault_tag> --json
   ```
   Filter to non-deprecated notes. You should have 50-100 candidates.

2. **For each draft (A, B, C), pick 20-50 angle-specific notes.** Use these signals:
   - **Source-analysis notes** (`type: source-analysis`): high-value, full digests of long sources. Include relevant ones in EVERY draft's list — these are gold.
   - **Interim notes** (`type: interim`, full tier only): include all of them in EVERY draft's list — these have the calibrated assessments.
   - **For Draft A (breadth):** the widest spread of sources across atomic items. Include any source the evidence digest cites for high-confidence claims.
   - **For Draft B (depth):** the strongest sources on the most important or most contested items, on every side of each contested point. Pull from `source-tensions.json` and `contradiction-graph.json` so both sides' best evidence is in the list.
   - **For Draft C (practitioner):** prefer sources with boundary conditions, comparative analyses, or applied case studies. Pull from sources the evidence digest groups under multiple atomic items (cross-cutting sources).

3. **Source overlap is fine.** Drafts can share source IDs — interim notes and key source-analyses should appear in all three lists. Differentiation comes from the angle-specific extras (the 5-15 sources unique to each draft's list).

4. **Cap each list at 50, minimum 20.** For `analytical` format, lean toward 35-50. For `structured`, lean toward 25-40. For `short`, lean toward 20-30.

5. **Write each list to disk** so the spawn template can reference it:
   - `research/temp/draft-a-source-list.md`
   - `research/temp/draft-b-source-list.md`
   - `research/temp/draft-c-source-list.md`

   Format:
   ```markdown
   # Draft A — must_read_note_ids (n=37)
   Angle: <2-3 sentence angle assignment>

   - <note-id-1>: <one-line summary or title>
   - <note-id-2>: <one-line summary or title>
   ...
   ```

---

## Step 10.3 — Spawn 3 draft sub-orchestrators in parallel

**Spawn 3 `hyperresearch-draft-orchestrator` subagents in ONE message.** This is true parallel execution. Each gets a different `draft_id`, `analytical_angle`, and (CRUCIALLY) a different `must_read_note_ids` array.

**Spawn template:**
```
subagent_type: hyperresearch-draft-orchestrator
prompt: |
  RESEARCH QUERY (verbatim, gospel):
  > {{paste research/query-<vault_tag>.md body}}

  QUERY FILE: research/query-<vault_tag>.md

  PIPELINE POSITION: You are one of 3 parallel step 10 sub-orchestrators
  in the hyperresearch V8 pipeline. After you and the other two return, the
  main orchestrator runs step 11 (synthesizer subagent) which reads all
  3 drafts and writes the final report. Your draft is an INPUT to that
  synthesis, not the final output.

  YOUR INPUTS:
  - query_file_path: research/query-<vault_tag>.md
  - vault_tag: <vault_tag>
  - draft_id: "a" (or "b" or "c")
  - output_path: research/temp/draft-a.md (or draft-b.md or draft-c.md)
  - analytical_angle: "<the 2-3 sentence angle assignment>"
  - must_read_note_ids: [<paste the IDs from research/temp/draft-<x>-source-list.md, e.g. 30-50 IDs>]
  - decomposition_path: research/prompt-decomposition.json
  - evidence_digest_path: research/temp/evidence-digest.md
  - comparisons_path: research/comparisons.md
  - source_tensions_path: research/temp/source-tensions.json
  - response_format: "<short|structured|analytical>"
  - citation_style: "<wikilink|inline|none>"
  - modality: "<collect|synthesize|compare|forecast>"
  - length_budget_words: <from the decomposition, default 3000>
  - long_form: <true|false>
  - style_guide_path: STYLE-GUIDE.md

  Read STYLE-GUIDE.md and every note on must_read_note_ids before
  writing. Do NOT survey the vault; your reading list is curated. Do
  NOT fetch new sources. Write your draft from your assigned angle in
  the house register, with footnote citations and reference material
  under a final "## Appendix material" heading.
```

**CRITICAL: never emit bare text while the 3 sub-orchestrators are running.** They will take 5-15 minutes each. Use this time to think — append notes to `research/temp/orchestrator-notes.md` about the synthesis you'll plan in step 11: what's the strongest thesis emerging across angles? Which atomic items will be contentious? Which findings must the final draft present, and with what confidence? One vault count check per minute max. Write your thoughts, don't just poll.

---

## Step 10.4 — Validate that all 3 drafts came back

When all 3 sub-orchestrators return:

1. **Confirm all 3 draft files exist:**
   - `research/temp/draft-a.md`
   - `research/temp/draft-b.md`
   - `research/temp/draft-c.md`

2. **Read each sub-orchestrator's report-back.** Each should report:
   - Path to the draft
   - Core findings
   - How many notes from `must_read_note_ids` it actually read
   - Strongest finding and the most contested point
   - Word/character count

3. **If a draft is missing or trivially short** (under 1000 chars for analytical, 500 for structured), re-spawn that single sub-orchestrator with the same inputs. Do not proceed to step 11 with fewer than 3 drafts.

4. **Do NOT synthesize the drafts in this step.** Step 11 (the synthesizer subagent) does that. Your only job here is to ensure 3 valid drafts exist.

---

## Exit criterion

**Light tier:**
- `research/notes/final_report_<vault_tag>.md` exists, its body is within `length_budget_words` (or the `short` range), it follows the report frame and `required_section_headings`, and it respects `citation_style`. If reference material was moved out, `research/notes/appendix_<vault_tag>.md` exists and Further reading links it.

**Standard / full tier:**
- All three drafts exist at `research/temp/draft-{a,b,c}.md`
- Each draft has non-trivial length (1000+ chars analytical, 500+ structured)
- Sub-orchestrator report-backs are captured (you can paraphrase them in `research/temp/orchestrator-notes.md` for the synthesis plan you'll write in step 11)

---

## Next step

Return to the entry skill (`hyperresearch`). Tier-based routing:

- **light tier:** You already wrote `research/notes/final_report_<vault_tag>.md` directly. Skip steps 11-14 (no synthesis, no critics, no patcher) and invoke `Skill(skill: "hyperresearch-15-polish")`.
- **full tier:** Invoke `Skill(skill: "hyperresearch-11-synthesize")`.
