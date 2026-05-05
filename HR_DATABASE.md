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
- Mimo 2.5 Pro is currently trusted for medium bounded Pi refactor slices inside one feature.
- Qwen 3.6 Plus is currently trusted for medium compatibility-preserving TypeScript API/payload rename slices, tiny/small strict profile-label wiring tickets, small TypeScript domain-shape/validation-helper groundwork, and small/medium internal profile/core composition helpers. Require separate tests for optional/null shapes, precedence rules independent of raw input order, nested partial overrides, and deep-clone reference safety.
- MiniMax M 2.7 is usable for small UI rename slices, but needs reviewer attention around deprecated compatibility shims.
- DeepSeek V4 Pro under Pi is currently trusted for focused test-only regression guard slices with precise failure modes.
- DeepSeek V4 Flash is usable as a fast read-only scout for label/hardcoding audits, but needs Codex source verification before acting on findings.
- GLM 5.1 is currently trusted for architecture guard and anti-regression documentation slices.
- Kimi K2.6 is currently trusted for strict bounded ticket work on compatibility-alias, profile-label wiring, and small/medium TypeScript UI/API cleanup slices; require explicit handoff verification updates when docs are in scope.
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
