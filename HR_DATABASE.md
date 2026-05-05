# Model HR Database

This folder tracks how cheap/local/open-weight coding models perform on real CodeLens RN refactor slices.

## Files

- `model_hr_db.json` - structured source of truth for LLM search and future analysis.
- `hr_findings_viewer.html` - local clickable/searchable viewer.
- `hrworkflow.md` - process and scoring rubric.
- `APP_INTEGRATION_NOTES.md` - notes for turning the HR data into backend/app concept cards later.
- `FUTURE_PI_PROMPTING.md` - reusable prompting guide for future Pi worker slices.

## Why JSON Instead of SQLite

JSON is better for this workflow right now:

- easy for any LLM to read in one pass
- easy to diff in git
- no install or server
- works with a static HTML viewer
- enough structure for filtering by model, task, rating, tags, and failure mode

Use SQLite later only if the record grows into hundreds or thousands of evaluations.

The current JSON shape is also meant to be app-friendly: evaluations can become cards, and tags can become clickable relationship nodes. See `APP_INTEGRATION_NOTES.md`.

Prompt improvements that apply across models should be promoted into `FUTURE_PI_PROMPTING.md`, not only left inside one model evaluation.
Recent promoted prompt improvements:
- when handoff/status docs are in scope, require latest verification and next-step queue updates
- for profile-label wiring, require both old-string `rg` checks and profile-access `rg` checks
- for profile-label wiring, specify whether labels belong in `DomainLabels` or a feature sub-profile to avoid label sprawl
- for UI label cleanup, specify whether dynamic count/pluralization strings and fallback placeholders are in scope
- required `rg` checks should use exact old UI strings, not approximate paraphrases
- for dynamic/pluralized UI label changes, require rendered examples for singular and plural cases
- do not reuse generic singular/plural profile labels for inline text unless their capitalization exactly matches the old rendered wording
- for read-only label audits, include placeholders, conditional labels, empty states, subtitles, errors, and button text in the search target list
- for composition/precedence helpers, require reversed/input-order tests that prove priority policy, not only same-order overlay tests
- for nested partial override APIs, require type-safe nested examples plus runtime tests for nested fields
- for pure merge helpers, require deep-clone/no-shared-reference checks, not only input snapshot equality
- for Codex-run Pi HR on Windows, use `--thinking high`, `--mode json`, quoted `--tools`, and prompt text from `Get-Content -Raw`
- for new-file worker slices, explicitly require the `edit` tool instead of shell redirection/cat-style file creation
- for TypeScript helper/refactor slices, review touched imports for unused values even when `tsc` passes
- for doc-anchor guard tasks, cap the number of tests/anchors so workers do not turn durable-doc checks into brittle wording locks
- on Windows, set `PI_PERMISSION_LEVEL=high` before launching Pi for HR worker runs; lower levels (`minimal`/`low`) silently block edit/bash so the worker plans the change but never writes (slice will exit 0 with no diff)
- on Windows PowerShell 5.1, do not pipe Pi `--mode json` output through `Tee-Object`; PS 5.1 wraps native-exe stdout in a way that can deadlock the run. Invoke Pi from Git Bash and redirect with `> file 2>&1`, or rely on Pi `--session-dir` for the audit trail
- when a prompt asserts a before/after test count, prefer "add exactly N new tests; baseline is approximately M" framing; absolute counts can be off and may pull a faithful worker toward padding
- keep full `npm:mitsupi` installed, but disable only Mitsupi's `extensions/go-to-bed.ts` from the package manifest; if a package update restores that local quiet-hours blocker, disable only that extension again or use `--no-extensions` as a temporary fallback; do not score the initial refusal as a model-quality failure

## How To Add a New Evaluation

Append a new object to `evaluations` in `model_hr_db.json`.

Required fields:

- `id`
- `date`
- `projectId`
- `model`
- `runner`
- `reviewerModel`
- `task`
- `slice`
- `difficulty`
- `rating`
- `score`
- `outcome`
- `strengths`
- `projectSpecificStrengths`
- `generalizedStackStrengths`
- `generalizedCodingStrengths`
- `misses`
- `reviewerFixes`
- `verification`
- `recommendedUse`
- `avoidFor`
- `promptImprovements`
- `usefulTools`
- `tags`
- `projectTags`
- `stackTags`
- `generalCodingTags`
- `riskTags`

New evaluations should also include prompt-shape fields:

- `promptStyle`
- `taskType`
- `contextProvided`
- `allowedWriteScopeStyle`
- `promptStrictness`
- `evaluationMeaning`
- `promptAdaptationNotes`

Older evaluations may not have these fields yet. Do not treat that as corruption; use the regular strengths, misses, prompt improvements, and tags for older records.

## How Future LLMs Should Use This

Use `strengths` for the direct result on the evaluated task.

Use `projectSpecificStrengths` for signals that depend on CodeLens RN, this branch, or this architecture.

Use `generalizedStackStrengths` for transferable stack-specific signals, such as TypeScript, React Native, Drizzle, SQLite, Vitest, Expo, Zustand, or feature-folder conventions.

Use `generalizedCodingStrengths` for broader coding behavior that transfers across stacks, such as scope control, compatibility reasoning, test discipline, and refactor judgment.

Use `promptStyle`, `promptStrictness`, and `evaluationMeaning` to interpret how transferable a result is. A strong score from `strict bounded ticket` means the model is useful when managed like a scoped worker. A strong score from a looser prompt means the model may have better autonomous planning ability.

## Pause Rule For Scores 7 Or Below

Any real worker result scored `7/10` or below triggers a pause before the next edit-capable slice.

Codex must write a short failure review in the HR record before continuing:

- what went wrong
- whether the issue came from model behavior, prompt design, task choice, or reviewer cleanup
- what code/docs were rejected, reverted, trimmed, repaired, or accepted with caveats
- what prompt improvement or model-assignment rule follows from the result
- whether the model should be restricted or avoided for that task type

This applies even if the final accepted diff is clean after Codex cleanup. The HR score describes the worker result, and a low score means the workflow needs a review checkpoint.

## Tag Categories

Use tag categories so a future LLM can transfer lessons correctly:

- `projectTags`: CodeLens-specific or ontology-refactor-specific labels.
- `stackTags`: stack-specific labels such as `typescript`, `react-native`, `drizzle`, `sqlite`, `vitest`, `expo`, `zustand`.
- `generalCodingTags`: cross-stack coding behaviors. These can transfer to Python, backend services, CLIs, frontend apps, and other projects.
- `riskTags`: areas where the model should be avoided or needs stronger review.

Useful `generalCodingTags`:

- `bounded-refactor`: small controlled changes without scope creep.
- `compatibility-boundary`: preserves public, legacy, persistence, or API boundaries.
- `terminology-rename`: consistent old-name/new-name cleanup.
- `interface-propagation`: follows type/interface changes through call sites.
- `test-discipline`: runs checks and uses failures correctly.
- `scope-control`: avoids unrelated files.
- `reporting`: gives useful final reports.
- `persistence-risk`: needs extra review around data-loss boundaries.

When asked which model to use for a task:

1. Read `model_hr_db.json`.
2. Match the task to prior `generalCodingTags`, `stackTags`, `difficulty`, and `recommendedUse`.
3. Avoid models whose `riskTags` or `avoidFor` match the task.
4. For another project on the same stack, weigh `stackTags`, `generalizedStackStrengths`, and `generalizedCodingStrengths`.
5. For another project on a different stack, weigh `generalCodingTags` and `generalizedCodingStrengths`.
6. Include prompt improvements from similar failures.
7. Match the prompt style to the model's history.
8. Recommend the model with the best relevant rating, not the best general reputation.

`reviewerModel` records which model performed the acceptance review. `reviewerFixes` should separate actual code fixes from documentation/reporting cleanup so future agents can judge whether the worker's implementation itself was clean.

## Current Short Read

- Kimi 2.5 is currently trusted for small bounded Pi refactor slices.
- Mimo 2.5 Pro is currently trusted for medium bounded Pi refactor slices inside one feature. For doc-anchor guard tasks, cap the expected number of tests and anchors because it can over-guard and need reviewer trimming.
- Qwen 3.6 Plus is currently trusted for medium compatibility-preserving TypeScript API/payload rename slices, tiny/small strict profile-label wiring tickets, small TypeScript domain-shape/validation-helper groundwork, small/medium internal profile/core composition helpers, small active-profile helper seam tests, and explicit active-profile source/resolver helpers. Require separate tests for optional/null shapes, precedence rules independent of raw input order, nested partial overrides, deep-clone/reference safety, and reviewer verification of final-report test-count arithmetic. Also review touched imports and require `edit`-tool file creation for new-file slices.
- MiniMax M 2.7 is usable for small UI rename slices, but needs reviewer attention around deprecated compatibility shims.
- DeepSeek V4 Pro under Pi is currently trusted for focused test-only regression/source-guard slices with precise failure modes, and for active-profile seam runtime guards. Verify final-report test counts; it can overstate counts and may also self-report verification commands as "blocked" while edits and bash actually ran successfully. On the upside, it can self-correct mid-run diagnostic flaws (e.g. it caught a fallthrough assertion that would have passed against the base default and rewrote the fixture).
- DeepSeek V4 Flash is usable as a fast read-only scout for label/hardcoding audits, but needs Codex source verification before acting on findings.
- GLM 5.1 is currently trusted for architecture guard, anti-regression documentation, and handoff/doc-sync slices. Verify current changed-file lists against `git diff --name-only`; it can preserve a stale file entry from an older batch.
- Kimi K2.6 is currently trusted for strict bounded ticket work on compatibility-alias, profile-label wiring, active-profile seam tests, correction validation against composed profiles, composition-precedence coverage extensions (mixed-kind, 3-overlay same-kind chains, no-op overlay equivalence), `overrideOntology` composition coverage, and small/medium TypeScript UI/API cleanup slices; require explicit handoff verification updates when docs are in scope. Will correctly flag prompt inaccuracies (e.g. misstated baseline counts) in the final report rather than padding to fit.
- No Pi model has earned trust for persistence, backup, import, restore, or migration tasks yet.
- Persistence, backup, import, restore, and migration tasks should stay with the strongest reviewer unless the worker is only generating a draft.

## 10/10 Evaluation Bar

A 10/10 worker does not just pass tests. It also makes reviewer acceptance obvious:

- exact scope discipline
- explicit scope-exception reporting
- required changes separated from optional improvements
- every remaining legacy name explained by a real boundary
- requested tests and searches run
- complete final report
- no reviewer interpretation needed

If a worker is strong but needs reviewer judgment around scope or optional behavior, score it closer to 8/10.

If a worker scores 7/10 or below, pause the batch and perform the failure review before assigning more work.
