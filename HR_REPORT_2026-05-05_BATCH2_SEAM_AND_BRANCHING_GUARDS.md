# HR Report - 2026-05-05 Batch 2: Seam Hardening And Doc 06 Anchors

Repo: `C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn`
Branch: `refactor/ontology-profile`
Initial reviewer: Claude Opus 4.7
Follow-up reviewer: Codex GPT-5
Runner: Pi CLI 0.72.0 with `opencode-go` models, `--thinking high`, `--mode json`, `PI_PERMISSION_LEVEL=high`, Git Bash launcher.

## Why This Batch Existed

After the previous 2026-05-05 ontology guard batch, three real coverage gaps remained that did not require any unsettled product decision (correction UI, correction persistence, overlay runtime source, relationship-semantics reconciliation):

1. The active-profile overlay seam tests covered no-cache and personal precedence, but did not test frozen overlay inputs, three-kind mixed overlays at the seam, no-arg/empty-array reference equality, or seam-level same-kind later-wins.
2. The stage10 durable-doc anchor block guarded docs 05/07/08/09/`NEXT_LLM_CONTEXT.md` but **not** `06_PROFILE_BRANCHING_AND_MERGE.md` - the only refactor doc still un-anchored after the previous batch.
3. The composition tests covered 2-overlay same-kind and personal-vs-project precedence, but no end-to-end three-kind mix (project + learning + personal in one composition) and no 3-overlay same-kind chain.

All three slices were edit-capable, test-only, single-file scopes, on HR-trusted models in their proven lanes. None entered any unsettled product decision.

## Slices

| # | Slice | Model | Result | HR score | Reviewer note |
|---|---|---|---:|---:|---|
| 1 | Active-profile seam runtime guards | DeepSeek V4 Pro | Accepted after cleanup | 9 | Self-corrected a diagnostic flaw mid-run; Codex fixed one inaccurate test title |
| 2 | Doc 06 branching/merge anchor guards (capped) | GLM 5.1 | Accepted after cleanup | 9 | Codex replaced one brittle implementation-sequence anchor with durable persistence-boundary anchors |
| 3 | Composition 3-kind / 3-overlay / no-op coverage | Kimi K2.6 | Accepted | 9 | Correctly flagged a misstated baseline count instead of padding |

No score `<= 7`, so the workflow's pause-and-review gate did not trigger.

## Verification (cumulative across the batch)

```text
node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
npm test -- --run src/features/ontology/__tests__/activeProfile.test.ts \
                 src/features/ontology/__tests__/profileComposition.test.ts \
                 src/__tests__/stage10-architecture-guards.test.ts
npm test -- --run
```

Result:

```text
TypeScript:                clean
activeProfile.test.ts:     12/12 passed (was 8/8;  +4 new)
profileComposition.test.ts: 20/20 passed (was 17/17; +3 new)
stage10-architecture-guards: 39/39 passed (was 38/38; +1 new)
Full suite:                52/52 files, 410/410 tests passed (was 402/402)
```

Additional checks per slice:

- `rg -n "as any"` in any new test file: 0 matches.
- `rg -n "@ts-(ignore|expect-error)"` in any new test file: 0 matches.
- ASCII scan of changed test files: clean.
- `git diff --stat`: 3 files, +209 / -0 lines, all under `src/`.
- Codex follow-up cleanup changed wording/anchors only; no production code was touched.

## Files Changed

```text
src/features/ontology/__tests__/activeProfile.test.ts        (+123 lines, 4 new it blocks)
src/__tests__/stage10-architecture-guards.test.ts            (+10  lines, 1 new it block, 5 new anchor assertions)
src/features/ontology/__tests__/profileComposition.test.ts   (+76  lines, 3 new it blocks)
```

No production source changed. No docs changed inside the main repo (the doc-anchor work pinned existing strings in 06; it did not edit the doc).

## Slice 1 - DeepSeek V4 Pro - Active-Profile Seam Runtime Guards (9/10)

**Prompt style:** `test-only guard` with explicit single-file scope, ordered required-test list, hard caps on test count and line growth, ASCII-only constraint, no-`as any` constraint.

**Goal closed:** four coverage gaps in `getActiveDomainProfile`:
- frozen overlay input safety (Object.freeze at every nested level: overlay, overrideLabels, overrideGraph, overrideGraph.nodeColors, addOntologyNodes array, inner OntologyNode);
- mixed three-kind overlay composition with reversed input orderings;
- no-arg vs empty-array reference equality (both must `===` `codingProfile`);
- seam-level same-kind later-wins precedence (mirrors the existing rule from `composeDomainProfile`).

**Strengths:**
- Added exactly the 4 required tests in the requested order with the requested titles. No padding, no extra tests.
- **Self-corrected a real diagnostic flaw mid-run.** The first iteration set `learningOverlay.overrideLabels.hubTitle = 'Learning Hub'`, which is identical to `codingProfile.labels.hubTitle` ('Learning Hub'). The fallthrough assertion (when personal has no hubTitle override) `expect(composed3.labels.hubTitle).toBe('Learning Hub')` would have passed even if the learning overlay were ignored entirely, because the base value is also 'Learning Hub'. The worker noticed this without reviewer prompting, edited the fixture to `'Learning Session Hub'`, and updated the assertion to match. The committed file is the corrected version; reviewer verified at line 204 (`hubTitle: 'Learning Session Hub'`) and line 243 (matching `toBe('Learning Session Hub')`).
- Used type annotations (`Record<string, string>`, `readonly OntologyNode[]`) instead of `as any` to satisfy the no-`as any` constraint.
- Snapshot-and-compare via `JSON.parse(JSON.stringify(codingProfile))` to prove `codingProfile` was unmutated post-frozen-input call.
- Reused the existing `makeTestNode` helper rather than redefining fixtures.

**Misses:**
- Final report claimed *"Bash commands are blocked at current permission level"* even though the worker successfully ran edits and grep searches. This appears to be a confused self-report (perhaps inherited from the earlier blocked-by-`minimal`-permission attempts in this conversation, before `PI_PERMISSION_LEVEL=high` was set). The slice's actual edits and tool calls succeeded; only the worker's *self-perception* of verification ability was off.
- Final report stated *"the four tests were already present in the file from a prior session."* They were not. Earlier `PI_PERMISSION_LEVEL=minimal` runs in this conversation made the worker generate the test text in its thinking but never write to disk; when the high-permission run started, the worker may have detected partial state from its own internal context and read it as pre-existing.
- One test title said the same overlay reference was passed twice, but the test actually used two same-kind project overlays in different orders.

**Reviewer fixes:** Codex changed the inaccurate test title to `same-kind project overlays apply input order deterministically at the seam`. The diagnostic flaw the worker noticed was self-fixed during the run. Reviewer ran TypeScript, targeted tests, and full suite externally because the worker reported it could not run them itself; all green.

**Why the score is 9 not 10:** the worker's final report had two factual self-confidence problems (claimed inability to run bash, claimed pre-existing tests). The diff itself is exemplary. A 10/10 here would require the same diff *plus* an accurate final report.

## Slice 2 - GLM 5.1 - Doc 06 Branching/Merge Anchor Guards (9/10)

**Prompt style:** `architecture/doc guard` with explicit single-file scope, exactly-1-test cap, exactly-5-assertions cap, **pre-supplied `.toContain(...)` literal strings**, explicit anti-Mimo cap reminder, instruction to leave the existing 5 anchor tests untouched.

**Goal closed:** add a single capped test that protects the durable section identity of `06_PROFILE_BRANCHING_AND_MERGE.md` - the only refactor doc not yet anchor-guarded after the previous batch. Anchors:

1. `## Architecture Decisions` - durable section header.
2. `personal corrections win, then active project/learning overlay, then base` - composition precedence decision #10.
3. `Active branch rules win for classification inside that branch.` - conflict rules durable phrase.
4. `Add pure composition helpers that produce a composed runtime profile from base + overlays.` - implementation sequence step #3.
5. `## What A Branch Can Change` - durable section header.

**Strengths:**
- Exact spec compliance: 1 it block, 5 assertions, no extras, no helpers introduced, no existing tests modified.
- Used the existing `readDoc` helper rather than re-importing fs/path.
- Anchor strings used verbatim, including the exact substring of decision #10.
- +10 lines total; well under the 8-15 line cap.
- Final report concise and accurate: stated 38 -> 39 in the stage10 file, 406 -> 407 in the full suite.

**Misses:** The initial guard used the exact implementation-sequence line `Add pure composition helpers that produce a composed runtime profile from base + overlays.` That line is less durable now that composition helpers already exist and the doc may later mark that sequence done.

**Reviewer fixes:** Codex replaced that brittle implementation-sequence anchor with durable persistence-boundary anchors: `Do not rush into persistence for branches.` and `Only then consider storage for branches/patch suggestions.`

**Why the score is 9 not 10:** the slice was strong but the prompt did most of the heavy lifting (anchors were pre-supplied). A 10/10 in this lane would require a prompt where the worker also chooses durable anchors from a doc and explains why each is anchor-worthy.

## Slice 3 - Kimi K2.6 - Composition Mixed-Kind / Same-Kind Chain / No-Op (9/10)

**Prompt style:** `test-only guard` with explicit single-file scope, three required tests with exact titles and behaviors, explicit "do not duplicate existing coverage" list, hard caps on test count and line growth, composition_precedence_policy, no-`as any` constraint.

**Goal closed:** three real composition coverage gaps:
1. Mixed three-kind overlay composition (`project + learning + personal` in one call) with per-field precedence and reversed-input-order independence, plus a fourth scenario where personal has no hubTitle so learning surfaces.
2. Three-overlay same-kind later-wins chain (extends the existing 2-overlay same-kind test) with three different input orderings proving order-independence.
3. No-op overlay equivalence (an overlay with no override fields is equivalent to an empty overlay list for labels, ontology arrays, metadata field count, ontology node count, and graph nodeColors).

**Strengths:**
- Added exactly 3 new it blocks, each in its own descriptive describe.
- Reused `makeOverlay` and `makeTestNode` helpers without redefining.
- Reversed input orderings used to prove same-kind later-wins is order-independent (`[a,b,c] -> C`, `[c,b,a] -> A`, `[b,c,a] -> A`).
- Per-field precedence assertions in the 3-kind mix test (hubTitle from personal, itemSingular from learning when personal does not override, mechanism color from personal, `proj_only` itemTypeNodeId still present from project addition).
- Used a `personalNoHub` variant to prove learning surfaces when personal does not override hubTitle.
- **Correctly handled a misstated baseline count in the prompt.** The prompt said "21 -> 24"; the file's actual baseline was 17 -> 20. The worker added exactly the 3 tests as goal-directed and noted the discrepancy in the final report rather than padding to hit the stated number. This is reviewer-friendly behavior.

**Misses:** none.

**Reviewer fixes:** none.

**Why the score is 9 not 10:** strong slice; the only thing keeping it from 10 is that the precedence policy was pre-stated in the prompt with explicit reversed-order test requirements. A 10/10 in this lane would require the worker to also decide *which* additional precedence dimensions to cover beyond the prompt's three.

## Generalized Lessons Promoted From This Batch

These have been added to `model_hr_db.json` (`generalizedLessons`) and to `hrworkflow.md` / `FUTURE_PI_PROMPTING.md`:

1. **`PI_PERMISSION_LEVEL=high` is mandatory.** On Windows, lower levels silently block edit/bash so the worker plans the change but never writes; the slice will exit 0 and produce no diff. The HR ticket itself bounds scope; the runtime permission level should not.

2. **Do not `Tee-Object` Pi `--mode json` on PowerShell 5.1.** PS 5.1 wraps native-exe stdout in a way that can deadlock the run. Observed: a slice that hung 30+ minutes with zero stdout while Pi itself was healthy. Use Git Bash with `> file 2>&1`, or rely on Pi's `--session-dir`.

3. **Frame baseline test counts as approximate, not absolute.** A misstated absolute count can pull a faithful worker toward padding. "add exactly N new tests; baseline is approximately M" works.

4. **Pre-check fallthrough fixture values against base profile defaults.** If a test asserts that an overlay's value surfaces, the overlay value must differ from the base profile's value or the assertion is non-diagnostic.

5. **Require concrete exit codes / pass-fail counts in worker final reports.** Do not accept self-reported "blocked" or "could not run verification" without a concrete error message - otherwise the reviewer ends up running every verification step independently.

## Trust Summary Changes

- **DeepSeek V4 Pro under Pi:** trust extended to *active-profile seam runtime guards* and includes a positive note for *self-correction mid-run on diagnostic flaws*. Risk note added: *can over-state internal blocks in the final report*.
- **GLM 5.1:** track record on architecture guard / anti-regression doc slices remains strong; pre-supplied `.toContain` literal strings work well with this model.
- **Kimi K2.6:** trust extended to *composition-precedence coverage extensions* (mixed-kind, 3-overlay same-kind chain, no-op equivalence). Positive note for *flagging prompt inaccuracies in the final report rather than padding*.

No model moved into a lower-trust category. No model crossed into persistence/migration territory; that lane remains empty for cheap workers.

## What This Batch Did Not Do (Intentionally)

- Did not implement correction persistence, correction UI, overlay runtime source, or relationship-semantics reconciliation. All four are unsettled product decisions and remain with the human and reviewer.
- Did not introduce new production code, helpers, or types. All edits were tests against existing seams and existing docs.
- Did not commit, push, or otherwise touch git state.
- Did not modify `06_PROFILE_BRANCHING_AND_MERGE.md` itself; only added an anchor guard test that asserts current durable strings exist.

## Next Likely Decisions (Unblocked, Not Yet Made)

The decision queue from `NEXT_LLM_CONTEXT.md` is unchanged by this batch:

- whether to commit the current dirty state in both repos
- whether the next product slice is correction persistence, first correction UI, branch/overlay persistence, or checker suggestions
- whether relationship semantics should be reconciled (`prerequisite/related/contrast` vs `is/is not + dynamic labels`) before more implementation
- whether active overlay state should remain test-only or get a real runtime source

The seam now has stronger immutability, frozen-input safety, mixed-kind, and no-op coverage; the durable doc set is fully anchor-guarded. Future work can build on this without re-establishing seam discipline.
