---
name: hyperresearch-16-readability-audit
description: >
  Step 16 (final) of the hyperresearch V8 pipeline. Spawns the
  hyperresearch-readability-recommender subagent (Read+Write tool-locked,
  Opus) to audit the polished final report and write JSON
  recommendations for paragraph merges, breaks, list/table conversions,
  bold injection, sentence splits, and HR removal. The orchestrator
  reads the recommendations and SELECTIVELY applies them via direct
  Edit calls (the recommender does NOT modify the report itself).
  Logs orchestrator decisions to a separate file. Runs for ALL tiers.
  Invoked via Skill tool from the entry skill.
---

# Step 16 — Readability audit & selective apply (FINAL STEP)

**Tier gate:** Runs for ALL tiers. Every report gets a readability audit, regardless of tier — readability is the dimension where small structural changes (paragraph rhythm, list/table conversions, bold injection) yield outsized scoring gains.

**Goal:** improve the report's visual structure, paragraph rhythm, and scannability without changing substantive content. The recommender writes a JSON list of suggested changes; YOU (the orchestrator) decide which to apply.

**Why split recommender + orchestrator-applied:** an Edit-based reformatter (V7-style) sometimes makes changes that hurt the argument — converting a flowing paragraph to a bullet list when the prose was load-bearing, or merging paragraphs that addressed distinct sub-topics. By having the recommender produce JSON suggestions and the orchestrator decide what to apply, we get the recommender's pattern-matching speed plus your judgment about which changes serve the research_query.

---

## Recover state

Read these inputs:
- `research/scaffold.md` — vault_tag
- `research/notes/final_report_<vault_tag>.md` — the polished final report from step 15

---

## Step 16.1 — Spawn the readability recommender

Spawn ONE `hyperresearch-readability-recommender` subagent. Single spawn, runs once.

**Spawn template:**
```
subagent_type: hyperresearch-readability-recommender
prompt: |
  RESEARCH QUERY (verbatim, gospel):
  > {{paste research/query-<vault_tag>.md body}}

  QUERY FILE: research/query-<vault_tag>.md

  PIPELINE POSITION: You are step 16 of the hyperresearch V8 pipeline —
  the final analytical pass. The final report at
  research/notes/final_report_<vault_tag>.md has been drafted (step 10),
  synthesized (step 11), critiqued (step 12), gap-filled (step 13),
  patched (step 14), and polish-audited (step 15). Your job: write
  JSON recommendations for paragraph rhythm, list/table conversions,
  and other structural readability improvements. You are tool-locked
  to [Read, Write] — you cannot Edit the report. The orchestrator
  reads your recommendations and decides which to apply.

  YOUR INPUTS:
  - draft_path: research/notes/final_report_<vault_tag>.md
  - appendix_path: research/notes/appendix_<vault_tag>.md
  - recommendations_path: research/readability-recommendations.json
  - length_budget_words: <from the decomposition>
  - style_guide_path: STYLE-GUIDE.md

  Run the length-budget check first (body words against the budget;
  recommend moves to the appendix, never deletions). Then write
  recommendations as a JSON array per the schema in your agent prompt.
  Cap at 50 recommendations, prioritized by impact.
```

---

## Step 16.2 — Read the recommendations

When the recommender returns:

1. **Read `research/readability-recommendations.json`.**

2. **Read the recommender's report-back.** It tells you:
   - Body word count against `length_budget_words`
   - Total count of recommendations
   - Breakdown by category (move-to-appendix, add-tldr, prose-run, bold-lead, remove-tic, heading-style, merge-paragraphs, break-paragraph, make-list, make-table, bold-keyterms, split-sentence, remove-hr, add-whitespace)
   - Highest-severity issue
   - Expected net char delta if all applied

---

## Step 16.3 — Decide which to apply

You are not obligated to apply every recommendation. Use these heuristics:

**Apply confidently:**
- All `move-to-appendix` recommendations when the body is over `length_budget_words`: cut the block from the body, append it to `research/notes/appendix_<vault_tag>.md` under the matching heading, and leave the pointer sentence. Make sure Further reading links the appendix.
- All `remove-tic` recommendations ("Confidence is" labels, settle-it lines outside the callout, paragraph-ending markers moved inside the sentence)
- `add-tldr` when the report has no `## TL;DR`: insert the recommended block directly under the italic line after checking that its first bullet is the controlling finding from the lead and every bullet has a bold lead and a marker
- `prose-run` recommendations where the `recommended` text keeps every sentence and marker of the `current` text and only rearranges them into bullets or shorter paragraphs
- `bold-lead` recommendations (zero risk when the sentence after the lead is unchanged)
- All `merge-paragraphs` recommendations where adjacent paragraphs are clearly the same sub-topic (rationale field confirms this)
- All `break-paragraph` recommendations on paragraphs over 600 CJK / 560 EN chars (about 90 words)
- All `remove-hr` recommendations (horizontal rules don't belong in research reports)
- All `add-whitespace` recommendations (zero risk)
- `make-table` recommendations when the prose-comparison passage cited 3+ entities × 2+ dimensions, the suggested table preserves all comparison points, and it fits eight rows by five columns

**Apply with judgment:**
- `heading-style` recommendations: apply when a heading is a sentence or an "X, not Y" pair; keep the section numbers. Folding sections is structural: do it only if the body has more than six H2s.
- `make-list` recommendations: confirm the prose was actually enumerative (3+ facts in sequence) and not reasoning. Arguments stay in prose ("one reading is... a second..."); facts a reader scans become bullets. If the rationale says "items appear sequentially in flowing prose," that's NOT a list candidate; skip it.
- `bold-keyterms` recommendations: bold only run-in leads of list items. Never bold a term or a statistic inside a paragraph.

**Apply skeptically (often skip):**
- `split-sentence` recommendations where the long sentence is followed by a short judgment sentence; that alternation is the house rhythm. Apply when the sentence carries two sources or two figures.
- Recommendations that touch the lead's first paragraph (load-bearing; keep as written)
- Recommendations that change tables that already exist (unless they exceed eight by five, which is a move)

**Always skip:**
- Any recommendation whose `current` field doesn't match the actual draft (the recommender mis-anchored — log and ignore)
- Recommendations that would change H2 heading text
- Recommendations that delete substantive content (the recommender shouldn't, but verify)

---

## Step 16.4 — Apply chosen recommendations via Edit

For each recommendation you decide to apply:

1. Use the Edit tool on `research/notes/final_report_<vault_tag>.md`
2. `old_string` = the recommendation's `current` field (exactly as the recommender wrote it)
3. `new_string` = the recommendation's `recommended` field

**For non-ASCII text (CJK / Arabic / Cyrillic):** the recommender copied `current` verbatim from Read output. Trust that. Don't retype.

**Order of application:**
0. `move-to-appendix` first (largest changes; the rest of the anchors are then stable)
1. `remove-hr`, `remove-tic` and `bold-lead` (small changes, clean baseline)
2. `merge-paragraphs` and `break-paragraph` (paragraph-level changes)
3. `make-list` and `make-table` (structural conversions)
4. `bold-keyterms` (within paragraphs that survived the merges/breaks)
5. `split-sentence` (within finalized paragraphs)
6. `add-whitespace` (final cleanup)

If an Edit fails because `old_string` doesn't match (recommender mis-anchored), skip that recommendation and continue with the rest.

---

## Step 16.5 — Log decisions

Write `research/readability-decisions.json` with the orchestrator's decisions:

```json
{
  "total_recommendations": <int>,
  "applied": [<list of recommendation IDs you applied>],
  "skipped": [
    {"id": "rec-N", "reason": "<one sentence>"}
  ],
  "edit_failures": [
    {"id": "rec-N", "reason": "old_string did not match the draft"}
  ],
  "net_char_delta_actual": <int — measure the actual delta>
}
```

This is the audit trail. If a future review finds a readability problem we should have fixed, this log shows whether we considered it and skipped, or never saw the recommendation.

---

## Exit criterion

- `research/readability-recommendations.json` exists
- `research/readability-decisions.json` exists with at least one entry in `applied` or all `skipped`
- `research/notes/final_report_<vault_tag>.md` reflects the applied recommendations
- The final report's structure (H2 list, lead, callout, options, closing, Notes, Further reading) is unchanged from step 15's output apart from blocks moved to the appendix (this step does not otherwise restructure)
- Body word count is within `length_budget_words` plus ten percent, and `research/notes/appendix_<vault_tag>.md` holds whatever moved

---

## Pipeline complete

Return to the entry skill (`hyperresearch`). Mark all todos complete. Tell the user the final report path, the appendix path, and the body word count against the budget.

You're done.
