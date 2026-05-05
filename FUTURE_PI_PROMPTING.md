# Future Pi Prompting Guide

This file captures general prompt improvements for future Pi worker slices. It is not model-specific.

## Default Prompt Skeleton

Every bounded worker prompt should include:

```markdown
<task>
What to do, repo path, branch, and current refactor state.
</task>

<context_files>
Exact files to read first.
</context_files>

<implementation_scope>
Allowed write scope.
</implementation_scope>

<do_not_edit>
Files or systems that are out of scope.
</do_not_edit>

<goal>
Concrete expected direction.
</goal>

<constraints>
Compatibility, safety, and behavior constraints.
</constraints>

<required_searches_before_final>
rg/search commands that prove old terminology or risky patterns were checked.
</required_searches_before_final>

<verification>
Exact TypeScript/test/lint commands.
</verification>

<final_report>
Required final report shape.
</final_report>
```

## Prompt Style Selection

Before writing a worker prompt, choose and record a prompt style. Tell the user which style you are using and why.

Common styles:

- `strict bounded ticket`: exact context files, exact allowed write scope, explicit compatibility boundaries, exact verification. Use for implementation slices where the worker should behave like an engineer assigned a ticket.
- `test-only guard`: production code is out of scope; the worker adds source-level or runtime regression guards for a known failure mode.
- `architecture/doc guard`: documentation and architecture guard tests only; useful after implementation slices to prevent backsliding.
- `loose exploratory`: read-only or planning work where the model is allowed to discover options. Do not use this for persistence or broad code edits.
- `read-only audit`: no writes allowed; worker inventories source files, classifies findings, and recommends bounded follow-up tickets.

Record these fields in `model_hr_db.json` after the run:

```json
{
  "promptStyle": "strict bounded ticket",
  "taskType": "compatibility-alias-refactor",
  "contextProvided": ["implementation_handoff.md", "05_ANTI_REGRESSION_RULES.md", "target source files"],
  "allowedWriteScopeStyle": "explicit file list",
  "promptStrictness": "high",
  "evaluationMeaning": "Tests whether the model can execute a scoped professional ticket with compatibility boundaries.",
  "promptAdaptationNotes": ["Keep this model on explicit-scope tickets until it proves broader planning."]
}
```

This makes HR comparisons fair. Do not compare a high-strictness ticket result directly against a loose autonomous planning result without noting the difference.

## Dynamic Prompt Adaptation

Use the HR record to adapt prompts:

- Model over-edits: stricter allowed scope and explicit do-not-edit list.
- Model misses compatibility: add preferred-only, legacy-only, and both-present tests.
- Model reports poorly: require final report sections and rg summaries.
- Model is good at guard tests: assign post-implementation architecture guard slices.
- Model is weak near persistence: keep it away from migrations/import/export, or make it draft-only.
- Model is doing a label audit: explicitly include placeholders, conditional labels, empty states, subtitles, errors, and button text in the search target list.

## Add These for Exported Type Changes

Use this when a slice changes exported TypeScript types, payloads, DTOs, public props, or shared interfaces:

```markdown
<scope_exception_policy>
Stay inside the allowed write scope.
If TypeScript requires a downstream call-site fix outside that scope, make the smallest necessary edit and report it under "Scope exceptions".
Do not make unrelated out-of-scope edits.
</scope_exception_policy>
```

Why: exported type changes often need downstream call-site repairs. The worker should not silently drift.

## Add These for Optional Improvements

Use this when the prompt contains optional work:

```markdown
<optional_change_policy>
Do optional improvements only if they are explicitly listed in the prompt.
If you do an optional improvement, report it separately from required changes and explain what test covers it.
</optional_change_policy>
```

Why: optional work can hide behavior changes. Keep it visible.

## Add These When Handoff Docs Are In Scope

Use this when the worker is allowed to edit `implementation_handoff.md`, status docs, or HR docs:

```markdown
<handoff_update_policy>
If you update handoff/status documentation, update every affected durable section:
- implemented-so-far / changed-files bullets
- latest verification block and test counts
- next-step queue, marking completed items done and adding remaining follow-ups

Do not leave the latest verification block pointing at an older slice.
In the final report, state exactly which handoff/status sections you updated.
</handoff_update_policy>
```

Why: strong workers have repeatedly updated implementation bullets but left stale verification/next-step text behind.

## Add These for Terminology Refactors

Use this for old-name/new-name cleanup:

```markdown
<terminology_audit>
Before final report, run rg for both old and new terminology inside the allowed scope.
Explain every remaining old-name hit.
An old name may remain only at a compatibility, persistence, public API, or explicitly documented legacy boundary.
</terminology_audit>
```

Suggested searches:

```powershell
rg -n "oldName|newName" path\to\scope
```

## Add These for Compatibility Aliases

Use this when introducing a new preferred field while preserving an old field:

```markdown
<compatibility_alias_policy>
Keep the old field as a documented compatibility alias.
Add tests for:
- preferred field only
- legacy alias only
- both fields present
Document the tie-break or union behavior explicitly.
</compatibility_alias_policy>
```

## Add These for Deprecated Wrapper Shims

Use this when renaming a component, function, or module while leaving a deprecated old name:

```markdown
<deprecated_shim_policy>
If you leave a deprecated shim, preserve the old import name and the old call/prop API.
Do not only re-export the new implementation if the new implementation has different props or parameters.
Add or update a test/type check that proves the old API still works, or explicitly report why no current caller needs it.
</deprecated_shim_policy>
```

Why: module compatibility and API compatibility are different. A shim that preserves only the import path can still break old callers.

## Add These for Composition Helpers

Use this when a slice implements profile/core composition, overlay merging, priority ordering, config merging, or other pure merge helpers:

```markdown
<composition_precedence_policy>
State whether precedence follows raw input order, kind priority, user priority, or another rule.
Add tests that pass inputs in the opposite order from the expected priority.
Example: if personal overlays must win over project overlays, test personal-before-project and project-before-personal.
</composition_precedence_policy>

<nested_override_policy>
If nested partial overrides are supported, add both TypeScript-safe examples and runtime tests for nested fields.
Do not rely on a shallow Partial type when nested keys are expected to be overrideable.
</nested_override_policy>

<deep_clone_policy>
For pure merge helpers, prove more than "inputs were not mutated".
Add checks or review notes showing output arrays, maps, and nested option objects do not share mutable references with inputs.
</deep_clone_policy>
```

Why: composition helpers can look correct while hiding the most expensive future bugs in precedence, nested partial typing, or shared mutable references.

## Add These for Persistence-Adjacent Work

Use this when the task touches DB rows, migrations, backup/import/export, restore, codecs, or raw SQL:

```markdown
<persistence_boundary_policy>
State every data shape involved:
- raw DB row shape
- Drizzle insert/select shape
- backup/import shape
- codec/domain shape
Do not wipe, delete, migrate, or transform persisted data until incoming data has been validated.
Add tests against the real boundary shape, not only domain-shaped objects.
</persistence_boundary_policy>
```

For now, do not give Pi persistence tasks unless Codex is ready to review very strictly.

## Final Report Upgrade

Prefer this final report block:

```markdown
<final_report>
Return:
- files changed
- required changes made
- optional changes made, if any
- scope exceptions, if any
- tests run and results
- rg/search output summary
- legacy names intentionally left and why
- compatibility boundaries preserved
- anything not done
</final_report>
```

## HR Feedback Loop

After Codex review, update:

- `model_hr_db.json`
- `hr_findings_viewer.html`
- `HR_DATABASE.md` if the model trust summary changed
- `hrworkflow.md` if the rubric changed

Record prompt improvements in the evaluation. Future prompts should reuse those improvements when the task has matching tags.

## Codex-Run Pi CLI Pattern

When Codex delegates a worker slice through the local Pi CLI, prefer non-interactive mode with a saved prompt file:

```powershell
pi --model <model> --thinking high --session-dir C:\pi-stuff\sessions --tools read,grep,find,ls,edit,bash -p @C:\pi-stuff\prompts\<ticket>.md
```

Use exact model ids when possible:

```powershell
pi --list-models qwen
pi --list-models opencode-go
pi --model opencode-go/qwen3.6-plus --thinking high --session-dir C:\pi-stuff\sessions --tools read,grep,find,ls,edit,bash -p @C:\pi-stuff\prompts\<ticket>.md
```

Here `opencode-go/...` is the OpenCode model provider inside Pi. Do not use the separate `opencode` CLI for this HR workflow.

Interactive `/model` or TUI selection is fine when the human drives Pi, but Codex-run HR work should use explicit Pi CLI model ids for reproducibility.

Use read-only tools for audits:

```powershell
pi --model <model> --thinking high --session-dir C:\pi-stuff\sessions --tools read,grep,find,ls -p @C:\pi-stuff\prompts\<ticket>.md
```

Do not ask Pi to make product decisions. Worker prompts should say:

```markdown
<decision_policy>
If the task requires a product, architecture, persistence, UI, relationship-semantics, app-builder,
agent/subagent runtime, MCP, adapter, DSL, open-source, pricing, or analytics decision not explicitly
settled in this prompt, stop and report the decision needed. Do not invent the direction.
</decision_policy>
```

This keeps Pi useful as the implementation worker while Codex and the human remain the orchestrator/owner.

## Multi-Slice Prompt Batches

When the user asks for multiple slices, Codex should propose a batch before launching workers:

```markdown
<batch_plan>
Batch goal:

Slice 1:
- model:
- prompt style:
- type: implementation | guard-test | docs | read-only audit
- goal:
- allowed write scope:
- do-not-edit:
- verification:
- big decision needed first: yes/no

Slice 2:
...
</batch_plan>
```

Worker prompts inside a batch must still be independent strict tickets. Do not give one Pi worker several unrelated edits in one prompt.

For progress updates, use:

```text
Batch progress: <done>/<total> done
Done:
- Slice N: result, tests, HR score
Current:
- Slice N: model and goal
Remaining:
- Slice N: model and goal
Blocked decisions:
- question, if any
```

Run edit-capable Pi slices sequentially in the shared worktree. A failed slice should be reviewed and either repaired, rejected, or re-ticketed before the next edit worker starts.

## Current Best Practices From Trials

- Add an explicit `rg` audit for naming slices.
- Add scope-exception reporting when changing exported types.
- Ask the worker to separate required changes from optional improvements.
- Require tests for preferred field, legacy alias, and both-present behavior when aliases exist.
- Keep persistence and backup/import tasks out of cheap-worker hands unless the prompt demands raw-shape tests and validate-before-wipe behavior.
- For read-only label audits, require the worker to classify placeholders, conditional labels, empty states, subtitles, errors, and button text; Codex should verify a sample before turning findings into tickets.
- When docs are in scope, explicitly require updating latest verification text and next-step queues, not only implementation bullets.
- For profile-label wiring, require old-string `rg` checks plus profile-access `rg` checks, and ask the worker to explain every remaining hardcoded hit.
- For profile-label wiring, state whether labels should go in `DomainLabels` or a feature sub-profile; do not let many feature-specific labels silently accumulate in one flat label bag.
- For UI label cleanup, explicitly state whether dynamic count/pluralization strings and fallback placeholders are in scope.
- Use exact old UI strings in required `rg` checks, not approximate paraphrases.
- For dynamic/pluralized UI label changes, require rendered before/after examples for singular and plural cases; profile value tests alone can miss capitalization or grammar regressions.
- Do not reuse generic singular/plural profile labels for inline text unless their capitalization exactly matches the old rendered wording.
- When a field is described as optional/null, require separate tests for both omitted and explicit `null` values; nullable-only tests do not prove optional semantics.
- For validation helper slices, state whether runtime validation of closed union fields is required or whether TypeScript-only enforcement is acceptable.
- For composition/precedence helpers, require reversed/input-order tests that prove the priority rule. A test named "personal wins" is not enough if its assertion still follows raw input order.
- For nested partial override APIs, require type support plus runtime nested-field assertions; shallow partial tests can miss the intended caller shape.
- For pure merge helpers, require deep-clone/no-shared-reference checks for nested maps, arrays, and option objects, not only input snapshot equality.
- Ban `as any` in worker tests unless the final report explicitly justifies why it is needed.
- Ask workers to keep docs and test comments ASCII-only unless the target file already requires non-ASCII text.
- For test-only guard tasks, require the final report to separate the actual number of new `it(...)` test blocks from the number of logical assertions covered inside those tests.
- Ask guard-test workers to describe residual risk accurately: source-level guards prevent specific regressions, but they do not prove future persistence/UI behavior is complete or risk-free.
