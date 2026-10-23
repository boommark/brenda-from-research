---
name: hyperresearch-6-cross-locus-reconcile
description: >
  Step 6 of the hyperresearch V8 pipeline. Reconciles the assessments
  from all depth investigators into research/comparisons.md — 3-5 named
  cross-locus dynamics (conflicts, convergences, complications) with
  guidance on how the draft should present each. This is the structural
  step that lets the report show where the evidence forks and where it
  converges.
  Invoked via Skill tool from the entry skill (full tier only).
---

# Step 6 — Cross-locus reconciliation

**Tier gate:** SKIP entirely for `light` tier (no loci = no comparisons). Only `full` tier runs this step.

**Goal:** before drafting, reconcile the assessments from all depth investigators. Produce `research/comparisons.md` — a short document naming 3–5 places where the loci conflict or complicate each other.

**Why this step exists:** the depth investigators each produced a calibrated assessment of their own locus. Some of those assessments disagree, some reinforce each other, some partially complicate each other. The draft must present those cross-locus dynamics explicitly and honestly — not summarize each locus in isolation, and not resolve a disagreement the evidence leaves open. Writing `comparisons.md` forces you to see the loci in cross-section before opening the draft.

**This step is always-on for full tier.** Even single-locus runs produce `comparisons.md` — with that locus's assessment as the lone anchor the draft must present. The discipline of writing it down BEFORE drafting is the same.

---

## Recover state

Read these inputs:
- `research/scaffold.md` — vault_tag
- `research/loci.json` — scored loci
- All interim notes: `$HPR search "" --tag <vault_tag> --type interim --json` then `$HPR note show <id1> <id2> ... -j`

You need the `## Assessment` section from every interim note in your context.

---

## Procedure

1. **Lay out all assessments.** For each interim note, read its `## Assessment` section. Write them down side-by-side in a scratch list (you can use `research/temp/orchestrator-notes.md` as scratch).

2. **Hunt for dynamics.** Ask of every pair of assessments:
   - Do they agree on the facts but disagree on what the facts mean?
   - Do they cite different evidence and reach opposite conclusions?
   - Does one locus's position assume something another locus's evidence complicates?
   - Is one locus's position a special case of another's general claim?
   - Do they converge on a conclusion but via different mechanisms (worth noting — convergence from independent paths is itself a finding)?

3. **Pick the 3–5 strongest cross-locus dynamics.** Reject weak ones (loci that are simply orthogonal, or that restate each other). You want cross-locus relationships that a good final draft should actually wrestle with.

4. **Write `research/comparisons.md`:**

   ```markdown
   # Cross-locus comparisons

   ## Tension 1: <short name for the dynamic>

   - **Locus A** ([[interim-A]]) finds: <one-line assessment, with confidence>
   - **Locus B** ([[interim-B]]) finds: <one-line assessment, with confidence>
   - **The cross-locus dynamic:** <2–3 sentences naming exactly how they relate — conflict? convergence? complication? special case? Name the load-bearing disagreement or agreement.>
   - **How the draft should present this:** <one sentence. Example: "Section on X should show that Y from Locus B narrows the reading of Locus A" or "Present both findings with their evidence; Locus B's evidence base is stronger (3 empirical studies vs 1 model) and the draft should say so plainly" or "The evidence does not settle this; the draft should say so and name what would settle it.">

   ## Tension 2: ...
   ```

5. **Calibration synthesis.** For each tension, note the investigators' confidence levels and "what would change this assessment" conditions from their calibration fields. When two investigators disagree but one is "high confidence" and the other is "low confidence," the draft should weight accordingly. When both name the same "what would change my mind" condition, that's a genuine open question to flag explicitly.

6. **This document is the evidence map of the draft.** Every dynamic you name here must be visibly presented in the final report: a paragraph inside the body section it bears on that lays out the evidence on each side in prose and says which side has the better evidence and why, not a one-line gesture and not a "Confidence is" label. The "what would settle it" conditions go to the report's single `### What would change this reading` callout. If you write `comparisons.md` with 4 dynamics and the draft only visibly presents 1, the report is hiding what the research found. The body is budgeted (six to seven pages), so a dynamic that does not bear on the decision is presented in the appendix file, with one sentence in the body.

---

## Exit criterion

- `research/comparisons.md` exists
- Contains 3–5 named tensions (or 1 distilled position for single-locus runs)
- Each tension includes: locus references, dynamic description, engagement guidance, calibration note

---

## Next step

Return to the entry skill (`hyperresearch`). Invoke step 7:

```
Skill(skill: "hyperresearch-7-source-tensions")
```
