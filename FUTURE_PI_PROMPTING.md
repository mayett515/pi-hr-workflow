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

## Current Best Practices From Trials

- Add an explicit `rg` audit for naming slices.
- Add scope-exception reporting when changing exported types.
- Ask the worker to separate required changes from optional improvements.
- Require tests for preferred field, legacy alias, and both-present behavior when aliases exist.
- Keep persistence and backup/import tasks out of cheap-worker hands unless the prompt demands raw-shape tests and validate-before-wipe behavior.
