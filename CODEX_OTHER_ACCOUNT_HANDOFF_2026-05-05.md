# Codex Other Account Handoff - 2026-05-05

This file is for a fresh Codex session from another account with no chat history.

Read this first, then inspect the worktree. Do not assume anything is committed.

## Model To Use

Use Codex / GPT-5 as the main reviewer and decision-maker.

Use Pi workers only for bounded implementation/test/doc slices. If Pi is used, run real HR worker slices with:

```powershell
$prompt = Get-Content -Raw C:\pi-stuff\prompts\<ticket>.md
pi --model <opencode-go/model-id> --thinking high --session-dir C:\pi-stuff\sessions --tools "read,grep,find,ls,edit,bash" --mode json -p $prompt
```

Never use low thinking for a scored HR worker run. Low thinking is allowed only for disposable CLI syntax probes.

New HR rule: if a real worker scores 7/10 or below, pause the batch before launching another edit-capable slice. Write a failure review first.

## Repos And Paths

Main repo:

```text
C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
branch: refactor/ontology-profile
```

Pi/HR repo:

```text
C:\pi-stuff
```

Local visualization file:

```text
C:\pi-stuff\model_hr_visualization.html
```

The visualization server was stopped. No server should be listening on port `8765`.

## Product Context

The work is about Kortex, not only a CodeLens refactor.

Current framing:

- Kortex Core is the reusable ontology / graph / versioned reasoning core.
- CodeLens/coding is the first serious child core or wrapper around Kortex.
- The current React Native app is a demonstration and working child implementation, not the full boundary of the system.
- The profile/ontology system should stay usable outside CodeLens later.

Future architecture ideas are preserved in docs, not implemented yet:

- agent/subagent execution ontology
- tags/subtags as behavior and execution constraints
- `is`, `is not`, and `extends` as future semantic/policy anchors
- self-building app framework direction
- Racket/Kortex DSL as a future language layer
- Kortex over existing systems and codebases through adapters
- MCP/source sync/write-back as future adapter work

Important: do not implement agent runtime, subagent orchestration, MCP policy tools, Racket/DSL runtime, app-builder runtime, source adapters, source write-back, or overlay persistence unless the user explicitly approves a new scope.

## Current Implementation State

Profile composition exists:

- `ProfileOverlayKind`
- `ProfileOverlay<TItemTypeNodeId>`
- `composeDomainProfile(base, overlays)`

Active profile seam exists:

- `getActiveDomainProfile()` returns `codingProfile` directly
- `getActiveDomainProfile([])` returns `codingProfile` directly
- `getActiveDomainProfile(overlays)` composes overlays explicitly
- no global active overlay state
- no persistence
- no UI activation
- no automatic profile mutation

Correction evidence groundwork exists, but it is domain-only:

- `OntologyCorrectionEvidence`
- `OntologyCorrectionSubjectKind`
- `OntologyCorrectionField`
- `OntologyCorrectionSource`
- `validateOntologyCorrection()`

No correction persistence/checker/UI has been implemented yet.

## Latest Five-Slice Pi Batch

The user asked for 5 Pi slices. Codex orchestrated and reviewed them.

Slice results:

1. Qwen 3.6 Plus - profile composition deep-clone tests - HR score 9
2. Kimi K2.6 - active-profile seam tests - HR score 9
3. DeepSeek V4 Pro - future runtime source guards - HR score 8
4. GLM 5.1 - anti-regression future boundary docs - HR score 9
5. Mimo v2.5 Pro - durable doc-anchor guards - HR score 6 after Codex cleanup

The Mimo score is important:

- Mimo did not delete or rewrite architecture docs.
- Mimo's allowed scope was `src/__tests__/stage10-architecture-guards.test.ts`.
- Its bad output was that it over-added 36 doc-anchor tests and guarded too many exact phrases.
- It also added non-ASCII dash comments.
- Codex removed the oversized block and replaced it with 5 focused tests.
- The remaining accepted code is the cleaned Codex-reviewed version, not the messy Mimo version.
- HR now records that Mimo should not receive uncapped doc-anchor guard tasks.

Do not panic about Mimo "changing architecture". It did not change the architecture docs. It only added overly strict tests that read those docs. Those tests were trimmed.

The intentional architecture-doc edit was GLM's slice in:

```text
ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
```

That edit added Kortex Core boundary rules and future architecture guardrails. It was reviewed and accepted.

## Current Dirty Files In Main Repo

Expected tracked changes:

```text
ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
ONTOLOGY_PROFILE_REFACTOR/implementation_handoff.md
src/__tests__/stage10-architecture-guards.test.ts
src/features/ontology/__tests__/activeProfile.test.ts
src/features/ontology/__tests__/profileComposition.test.ts
```

Expected untracked local tool folder:

```text
.claude/
```

Do not commit `.claude/` unless the user explicitly asks.

## Current Dirty Files In Pi Repo

Expected tracked changes:

```text
FUTURE_PI_PROMPTING.md
HR_DATABASE.md
hr_findings_viewer.html
hrworkflow.md
model_hr_db.json
```

Expected new files/folders:

```text
HR_REPORT_2026-05-05_ONTOLOGY_GUARD_BATCH.md
model_hr_visualization.html
prompts/
sessions/
.claude/
```

`prompts/` contains the five slice tickets.
`sessions/` contains raw Pi JSON logs.
Do not delete these unless the user asks.

## Verification Already Run

In the main repo:

```powershell
node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
npm test -- --run src/features/ontology/__tests__/profileComposition.test.ts src/features/ontology/__tests__/activeProfile.test.ts src/__tests__/stage10-architecture-guards.test.ts
npm test -- --run
git diff --check
```

Results:

```text
TypeScript clean
focused profile/guard tests: 63/63 passed across 3 files
stage10 architecture guards: 38/38 passed
full suite: 402/402 passed across 52 files
git diff --check: clean
changed files ASCII-clean
no "as any" in changed test files
```

In the Pi repo:

```powershell
node parse checks for model_hr_db.json and hr_findings_viewer.html embedded JSON passed
git diff --check passed
```

The full-suite output includes Node SQLite experimental warnings. Those warnings are pre-existing/runtime warnings, not test failures.

## Files To Read First In A New Session

Main repo:

1. `ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md`
2. `ONTOLOGY_PROFILE_REFACTOR/implementation_handoff.md`
3. `ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md`
4. `ONTOLOGY_PROFILE_REFACTOR/07_KORTEX_CORE_AND_CHILD_CORES.md`
5. `ONTOLOGY_PROFILE_REFACTOR/08_KORTEX_LANGUAGE_LAYER_AND_ADAPTERS.md`
6. `ONTOLOGY_PROFILE_REFACTOR/09_KORTEX_OVER_EXISTING_SYSTEMS.md`
7. `src/__tests__/stage10-architecture-guards.test.ts`
8. `src/features/ontology/__tests__/profileComposition.test.ts`
9. `src/features/ontology/__tests__/activeProfile.test.ts`

Pi repo:

1. `C:\pi-stuff\HR_DATABASE.md`
2. `C:\pi-stuff\hrworkflow.md`
3. `C:\pi-stuff\FUTURE_PI_PROMPTING.md`
4. `C:\pi-stuff\model_hr_db.json`
5. `C:\pi-stuff\HR_REPORT_2026-05-05_ONTOLOGY_GUARD_BATCH.md`

## Mimo Cleanup Details

Accepted remaining guard block in `stage10-architecture-guards.test.ts`:

```text
Kortex durable doc future-direction anchor guards
```

It should contain 5 tests:

```text
keeps core, agent, and self-building app anchors in doc 07
keeps future operation anchors in doc 08
keeps self-building app overlay anchor in doc 09
keeps agent and self-building app cautions in NEXT_LLM_CONTEXT.md
keeps future boundary guardrails in anti-regression rules
```

If a new session sees dozens of exact phrase doc-anchor tests, something is wrong. The cleaned state is 5 focused tests, not 36.

## What The HR Updates Mean

Pi HR now includes five new evaluations:

```text
Qwen 3.6 Plus: 9 strong
Kimi K2.6: 9 strong
DeepSeek V4 Pro: 8 strong
GLM 5.1: 9 strong
Mimo v2.5 Pro: 6 medium
```

Important rule added everywhere:

```text
If a real worker result scores 7/10 or below, pause the batch.
Write a failure review before assigning another edit-capable slice.
```

This was added because Mimo's 6/10 result showed that a worker can remain in scope but still produce a guard design that is too noisy.

## Next Likely Work

Do not jump straight into implementation unless the user asks.

Likely next decision gates:

- whether to commit the current main repo and Pi repo changes
- whether the next product slice is correction persistence, first correction UI, branch/overlay persistence, or checker suggestions
- whether relationship semantics should be reconciled before implementation: current `prerequisite` / `related` / `contrast` compatibility versus future `is` / `is not` plus dynamic labels
- whether active overlay state should remain test-only or get a real runtime source later

If the user asks for more Pi work, propose a small batch but pause on any score <= 7.

## Hard Guardrails

- Do not stage, commit, push, reset, checkout, or delete files unless the user explicitly asks.
- Do not revert user changes.
- Do not implement persistence, backup/import/export, migrations, or data-loss logic through Pi workers.
- Do not let Kortex Core depend on CodeLens UI or coding-only assumptions.
- Do not treat self-updating as hidden source-code mutation.
- Do not reduce future agent/subagent behavior to prompt text only; preserve the path for ontology-backed execution policy.
- Do not reduce future self-building apps to one-shot prompt-to-code generation; preserve the project ontology/coherence layer idea.
- Do not make architecture docs brittle with too many exact phrase guards.
