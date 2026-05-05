# HR Workflow for Cheap/Open-Weight Coding Models

This workflow is for testing local or cheap coding models as bounded workers while Codex acts as reviewer.

## Roles

- Worker model: receives one narrow implementation slice and edits the repo.
- Codex reviewer: inspects the actual diff, runs verification, fixes small polish issues if needed, and rates the worker.
- Human owner: chooses which model to try next and decides whether a model is trusted for harder slices.

## Codex-Orchestrated Pi CLI Workflow

Codex may run Pi directly when the user asks to delegate a bounded worker slice.

Use the Pi CLI in non-interactive mode so the run is reproducible:

```powershell
pi --model <provider/model-or-pattern> --thinking <level> --tools read,grep,find,ls,edit,bash -p @<prompt-file>
```

List/select Pi models with:

```powershell
pi --list-models <search>
pi --list-models opencode-go
pi --model opencode-go/qwen3.6-plus --thinking high -p @<prompt-file>
```

In this HR workflow, `opencode-go/...` means the OpenCode model provider exposed inside Pi. Do not use the separate `opencode` CLI for HR worker runs.

Do not rely on interactive `/model` selection for HR runs unless the human is driving the Pi TUI. For Codex-run HR work, use explicit Pi CLI model ids so the evaluation is reproducible.

Recommended session shape:

```powershell
pi --model <model> --thinking high --session-dir C:\pi-stuff\sessions --tools read,grep,find,ls,edit,bash -p @C:\pi-stuff\prompts\<ticket>.md
```

Codex should not use Pi as a second orchestrator. Pi is the bounded worker. Codex remains the reviewer and decision gate.

Before launching Pi, Codex must:

- read `model_hr_db.json`, `HR_DATABASE.md`, and `FUTURE_PI_PROMPTING.md`
- recommend the model and prompt style
- inspect current `git status --short`
- write or show the strict bounded ticket
- define exact allowed write scope
- define exact do-not-edit boundaries
- define exact verification commands and rg/search checks
- ask the human before any big product or architecture decision

Big decisions stay with the human and Codex, not Pi:

- relationship semantics such as `is` / `is not`
- persistence, backup, import, restore, migrations, or data-loss boundaries
- first UI/product surface for correction flows
- app-builder, agent/subagent runtime, MCP, adapter, or DSL direction
- public/open-source strategy
- pricing, hosting, telemetry, analytics, or product positioning
- any scope expansion outside the agreed ticket

After Pi finishes, Codex must:

- inspect the actual diff
- reject or fix scope violations
- run TypeScript, targeted tests, full tests when appropriate, and required rg/search checks
- update CodeLens handoff/docs if the slice is accepted
- update `model_hr_db.json`, `hr_findings_viewer.html`, and HR markdown docs if this was a real worker evaluation
- clearly separate Pi's work from Codex reviewer fixes in the HR record

If Pi requires an architectural choice to continue, the correct behavior is to stop and ask the human. Do not let the worker invent the product direction.

## Multi-Slice Batch Workflow

When the user asks for several slices, such as "give me 4 slices", Codex should create a batch plan before running workers.

Batch plan format:

```text
Batch: <short goal>

Slice 1
  model:
  prompt style:
  goal:
  allowed write scope:
  do-not-edit:
  verification:
  decision needed before run:

Slice 2
  ...
```

Rules:

- propose up to 4 slices at once
- mark which slices are implementation, guard/test, docs, or read-only audit
- choose the model from HR history for each slice
- ask the human before any slice that contains a big decision
- run edit-capable workers one at a time in the shared worktree
- do not start the next edit slice until Codex has reviewed, tested, and either accepted or rejected the previous slice
- read-only audits may be batched more freely, but their findings still need Codex verification before implementation
- after each accepted worker slice, update CodeLens handoff/docs and Pi HR if it was a real model evaluation

Progress updates to the user should use this shape:

```text
Batch progress: 2/4 done
Done:
- Slice 1: <result>, tests, HR score
- Slice 2: <result>, tests, HR score

Current:
- Slice 3: <running model>, <goal>

Remaining:
- Slice 4: <goal/model>

Blocked decisions:
- <question for human, if any>
```

If a later slice depends on a big product/architecture decision, pause the batch and ask the human before continuing.

## HR Database Files

Use these files to remember model performance across tasks:

- `model_hr_db.json` - structured source of truth for LLMs. Read this when choosing which model should handle a future slice.
- `hr_findings_viewer.html` - clickable/searchable local viewer for humans. Open this in a browser to browse results.
- `HR_DATABASE.md` - instructions for adding new model evaluations and using the database.
- `FUTURE_PI_PROMPTING.md` - reusable prompting patterns for future Pi worker slices.

When a worker model finishes a slice, add or update an evaluation in `model_hr_db.json`. Record the model, runner, task, rating, strengths, project-specific strengths, generalized stack strengths, generalized coding strengths, misses, reviewer fixes, verification, recommended use, avoid-for list, and prompt improvements.

Also record `reviewerModel`. For this project, current acceptance reviews are done by `Codex GPT-5`. In `reviewerFixes`, be specific about whether Codex changed code, tests, docs, or only the HR record.

When asking "which model should handle this task?", compare the new task against previous `tags`, `difficulty`, `recommendedUse`, and `avoidFor` fields. Prefer the model with the best relevant history, not the model with the best general reputation.

For another project on the same stack, use `generalizedStackStrengths` plus `generalizedCodingStrengths` as the main signal. Use `projectSpecificStrengths` only when the new project has a similar architecture or boundary problem.

## Transfer Tags

Each evaluation should include four tag layers:

- `projectTags`: local to this project or branch.
- `stackTags`: reusable within a stack, such as TypeScript, React Native, Drizzle, SQLite, Vitest, Expo, Zustand.
- `generalCodingTags`: reusable across stacks, including Python or backend projects.
- `riskTags`: tasks where the model needs stricter review or should not be used.

Use `generalCodingTags` when deciding whether a model can transfer to a different stack. For example, a model that scored well on `bounded-refactor`, `compatibility-boundary`, `interface-propagation`, and `test-discipline` in TypeScript may also be a decent worker for a bounded Python refactor, provided the prompt includes Python-specific checks.

Current general coding tags:

- `bounded-refactor`
- `compatibility-boundary`
- `terminology-rename`
- `interface-propagation`
- `test-discipline`
- `scope-control`
- `reporting`
- `persistence-risk`
- `composition-helper`
- `precedence-rules`
- `deep-clone-immutability`

## Worker Prompt Shape

Every worker prompt should include:

- repo path and branch
- exact context files to read first
- allowed write scope
- explicit do-not-edit list
- small implementation goal
- compatibility boundaries
- exact verification commands
- required final report format

Use XML blocks for hard boundaries:

```markdown
<task>...</task>
<context_files>...</context_files>
<implementation_scope>...</implementation_scope>
<goal>...</goal>
<constraints>...</constraints>
<verification>...</verification>
<final_report>...</final_report>
```

## Prompt Style Records

Every model evaluation should record the prompt style, because an 8/10 result from a loose exploratory prompt means something different than an 8/10 result from a strict ticket.

Use these fields in `model_hr_db.json` for new evaluations:

- `promptStyle`: short name, such as `strict bounded ticket`, `test-only guard`, `architecture/doc guard`, or `loose exploratory`.
- `taskType`: what kind of work the prompt tested, such as `compatibility-alias-refactor`, `terminology-rename`, `source-guard-tests`, or `persistence-boundary-review`.
- `contextProvided`: files or context classes the worker received.
- `allowedWriteScopeStyle`: `explicit file list`, `feature glob`, `repo-wide read only`, or similar.
- `promptStrictness`: `low`, `medium`, or `high`.
- `evaluationMeaning`: one sentence explaining what the score proves.
- `promptAdaptationNotes`: how to prompt this model next time based on the result.

When choosing a model later, compare both the model score and the prompt style. A model that only performs well with high-strictness prompts can still be useful, but should keep receiving bounded tickets.

## Dynamic Prompt Adaptation

Prompts should adapt to the model and task, not stay one-size-fits-all.

- Strong but wandering model: use an explicit file list, hard do-not-edit scope, and mandatory rg summary.
- Good test/doc model: give guard or documentation-hardening tasks and require source-level assertions.
- Weak compatibility model: require preferred-only, legacy-only, and both-present tests before accepting implementation.
- Persistence-risk model: avoid implementation; if used at all, require raw DB shape tests and validate-before-wipe rules.
- Strong inference model: allow a little less file-by-file context only when the task is non-destructive and easy to review.
- Composition-helper model: require reversed/input-order precedence tests, nested partial override tests, and deep-clone/no-shared-reference checks.

Tell the user which prompt style you are using and why before sending a worker prompt. This keeps the HR result interpretable.

## Slice Design

Good trial slices are:

- small enough to review in one diff
- narrow enough that the correct files are obvious
- meaningful enough to expose reasoning quality
- guarded by targeted tests

Avoid giving new models:

- DB migrations
- backup/import/export logic
- broad renames across the whole app
- unclear architecture decisions
- tasks where failure can corrupt user data

## Review Checklist

Codex reviewer checks:

- Did the worker read and follow the handoff/context?
- Did it stay inside allowed write scope?
- Did it preserve compatibility boundaries?
- Did it avoid unrelated refactors?
- Did it run the requested verification?
- Do reported tests match actual repo state?
- Are remaining legacy names justified by a real boundary?
- Did the change improve the refactor without making tomorrow harder?

## Ratings

Strong:
- scoped diff
- understands compatibility boundaries
- tests pass
- no invented architecture
- easy to review

Medium:
- mostly correct but leaves naming mess, weak docs, or small missed assertions
- needs reviewer cleanup
- safe for very small slices only

Bad:
- touches forbidden files
- changes persistence without a migration plan
- weakens/deletes tests
- renames app-wide compatibility types too early
- reports tests it did not run
- introduces broad abstractions without need

Dangerous:
- destructive commands
- data-loss risk
- backup/import regression
- secret/env exposure
- large uncontrolled edits

## 10/10 Worker Standard

A 10/10 worker result means Codex can accept a similar slice with very little reviewer suspicion.

The worker must:

- stay inside the allowed write scope
- if it must touch outside scope, stop or explicitly report the scope exception before claiming completion
- separate required changes from optional improvements
- avoid optional behavior changes unless the prompt asked for them or the worker clearly justifies and tests them
- preserve all compatibility, persistence, and public API boundaries
- run the exact requested verification commands
- run the requested search/rg checks and explain every remaining hit
- update handoff/docs only where requested
- produce a complete final report with files changed, tests run, remaining legacy names, and scope exceptions
- leave no reviewer interpretation needed about why a boundary was crossed

8/10 means the implementation is strong but reviewer judgment was still needed. 10/10 means the worker made the review boring.

## Prompt Adjustments Toward 10/10

See `FUTURE_PI_PROMPTING.md` for the reusable version of these patterns.

For future Pi worker prompts, add these blocks when the slice changes exported types or could affect downstream consumers:

```markdown
<scope_exception_policy>
Stay inside the allowed write scope.
If TypeScript requires a downstream call-site fix outside that scope, make the smallest necessary edit and report it under "Scope exceptions".
Do not make unrelated out-of-scope edits.
</scope_exception_policy>

<optional_change_policy>
Do optional improvements only if they are explicitly listed in the prompt.
If you do an optional improvement, report it separately from required changes and explain what test covers it.
</optional_change_policy>

<final_review_requirements>
Before final report:
1. Run the requested tests.
2. Run the requested rg/search checks.
3. Inspect the diff.
4. List any files changed outside the allowed write scope and why.
5. List any optional behavior changes and what verifies them.
</final_review_requirements>
```

Use these especially for:

- exported TypeScript type changes
- API/payload compatibility slices
- UI behavior changes
- persistence-adjacent code
- cross-feature call-site repairs
- handoff/status documentation updates
- profile-label wiring slices

For handoff/status documentation updates, explicitly require workers to update latest verification blocks and next-step queues. Strong workers have repeatedly updated implementation bullets while leaving stale verification text for Codex to fix.

## Current Model Notes

Kimi 2.5:
- Result on graph `typeNodeId` slice: strong.
- Stayed scoped, preserved compatibility, ran tests.
- Needed minor reviewer cleanup for leftover naming in a helper and warning text.
- Good candidate for bounded refactor slices.

Mimo 2.5 Pro:
- Result on promotion `typeNodeId` / `proposedTypeNodeId` slice: strong.
- Stayed scoped and preserved LearningConcept, capture hint, and DB cache compatibility boundaries.
- Needed reviewer cleanup in handoff docs, not code.
- Good candidate for medium bounded refactor slices inside one feature.

Qwen 3.6 Plus:
- Result on retrieval `typeNodeId` / `typeNodeIds` slice: strong.
- Result on tiny `GraphLegend` title profile-label slice: strong, 9/10; needed only one redundant profile accessor cleanup.
- Result on ontology correction evidence groundwork slice: strong, 8/10; stayed domain-only and scoped, but Codex fixed `reason` from required nullable to optional nullable and added the omitted-reason test.
- Result on profile overlay composition helper slice: strong, 8/10; stayed scoped and verified, but Codex fixed personal overlay precedence, nested partial override typing/tests, deep-clone behavior, and minor test/doc cleanup.
- Preserved `conceptTypes` as a compatibility alias while adding preferred `typeNodeIds`.
- Followed TypeScript payload rename through codecs, row mappers, formatters, tests, and necessary downstream consumers.
- Good candidate for medium compatibility-preserving API/payload rename slices, tiny/small strict profile-label wiring tickets, small TypeScript domain-shape/validation-helper groundwork, and small/medium internal profile/core composition helpers. Prompt composition tasks with explicit precedence, nested partial override, and deep-clone checks.

MiniMax M 2.7:
- Result on `TypeNodeChip` UI primitive rename slice: medium.
- Followed the scope exception policy well and updated current call sites.
- Needed reviewer code polish because the deprecated `ConceptTypeChip` shim preserved the old import name but not the old prop API.
- Usable for small UI rename slices, but prompt it explicitly when compatibility wrappers must preserve old props.

DeepSeek V4 Pro under Pi:
- Result on TypeNodeChip compatibility guard tests: strong.
- Result on ontology correction evidence architecture guards: strong, 8/10.
- Stayed test-only, scoped, and targeted the exact previous shim failure mode.
- Continues to be reliable for one-file source guard work, though Codex had to correct final-report test counts and overconfident residual-risk wording.
- Much better behavior under Pi with tight prompts than the earlier OpenCode CLI run.
- Good candidate for focused regression guard tests before giving it implementation work.

GLM 5.1:
- Result on ontology-profile naming boundary architecture guards: strong.
- Added precise source-level guards and anti-regression docs without globally banning legacy names.
- Needed only a one-word documentation typo fix.
- Good candidate for architecture guard and anti-regression documentation slices.

Kimi K2.6:
- Result on `ConceptListFilters.typeNodeIds` compatibility alias slice: strong.
- Result on review UI `ReviewProfile` label wiring slice: strong.
- Result on graph UI `GraphProfile` label wiring slice: strong.
- Result on small Learning UI `DomainLabels` wiring slice: strong, 9/10.
- Result on Learning UI helper-label/nested flashback slice: strong, 9/10.
- Result on graph helper-label nested `GraphProfile` slice: strong, 8/10; needed one handoff count correction.
- Result on dynamic/fallback label wiring slice: medium, 7/10; required a reviewer production fix because count labels reused capitalized generic labels where the old UI rendered lowercase words.
- Followed a strict bounded ticket prompt with explicit files, compatibility aliases, rg summary, TypeScript, targeted tests, and full suite.
- Usually keeps scope well, but dynamic/pluralized UI text needs explicit rendered-output checks for singular and plural cases.
- Good candidate for future compatibility-alias refactors, profile-label wiring, and small/medium TypeScript UI/API cleanup slices when given a professional bounded ticket.

DeepSeek V4 Flash:
- Result on read-only hardcoded domain-label audit: medium.
- Correctly identified review UI as the largest remaining unprofiled label seam and avoided broad rename recommendations.
- Missed a few labels (`ReflectionInput` placeholder and `Close`) and had one final-report field error, so use it as a scout with Codex verification.
- Good candidate for fast read-only audits before assigning implementation to a stricter bounded-ticket worker.

## Current Refactor Next-Step Queue

1. Profile composition helpers are implemented and verified; decide next whether to wire a first runtime seam, persist overlay/branch state, or continue toward correction persistence/checker.
2. Keep Racket/DSL, source adapters, MCP, file watchers, and Kortex-over-codebase sync as documented architecture, not next-slice implementation.
3. Before changing relationship semantics, explicitly reconcile current `prerequisite` / `related` / `contrast` compatibility with the newer `is` / `is not` boundary-anchor direction.
4. Later only: second-profile support, overlay activation UI, adapter write-back, and cleanup migrations after compatibility is proven.

## Rule of Thumb

If the worker crosses a persistence or backup boundary without being explicitly asked, reject the slice and review carefully.
