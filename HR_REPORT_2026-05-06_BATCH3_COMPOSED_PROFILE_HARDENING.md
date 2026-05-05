# HR Report - 2026-05-06 Batch 3: Composed Profile Hardening

Repo: `C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn`
Branch: `refactor/ontology-profile`

Runner: Pi CLI 0.72.0 with `opencode-go` models, `--thinking high`, `--mode json`, `PI_PERMISSION_LEVEL=high`.
Reviewer: Codex GPT-5.

## Why This Batch Existed

Batch 2 closed remaining seam/doc-anchor gaps after the first active-profile overlay work. Batch 3 continued only with bounded hardening around already-settled behavior:

1. Correction validation should validate against the exact profile it is given, including explicitly composed overlay profiles.
2. `overrideOntology` had implementation paths with less direct test coverage.
3. `getOntologyNode` / `getOntologyNodeLabel` needed explicit helper tests proving profile-parameter behavior and no hidden overlay state leak.
4. Refactor docs still needed to reflect Batch 2 and Batch 3 rather than the older 402-test state.

No product decision was made. This batch did not implement persistence, UI, checker execution, branch storage, adapters, MCP, Racket/DSL runtime, relationship-taxonomy changes, agent/subagent runtime, or self-building-app runtime.

## Slices

| # | Slice | Model | Result | HR score | Reviewer note |
|---|---|---|---|---:|---|
| 1 | Correction validation with composed profiles | Kimi K2.6 | Accepted | 9/10 | Initial one-shot blocked by local quiet-hours guard; same-session confirmation completed correctly. |
| 2 | `overrideOntology` composition coverage | Kimi K2.6 | Accepted after tiny fixture cleanup | 9/10 | Code/tests strong; Codex replaced one late-night-specific fixture phrase. |
| 3 | Active-profile ontology helper seam | Qwen 3.6 Plus | Accepted | 8/10 | Code/tests good; final-report count arithmetic drifted against historical counts. |
| 4 | Refactor doc sync | GLM 5.1 | Accepted after doc cleanup | 8/10 | Good doc sync; Codex removed one stale changed-file entry. |

No score was 7/10 or below, so the pause gate did not trigger.

## Verification

Final reviewer verification after all slices:

```text
node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
Result: clean

npm test -- --run src/features/ontology/__tests__/profileComposition.test.ts src/features/ontology/__tests__/activeProfile.test.ts src/features/ontology/__tests__/corrections.test.ts src/__tests__/stage10-architecture-guards.test.ts
Result: 93/93 passed across 4 files

npm test -- --run
Result: 419/419 passed across 52 files
```

Additional hygiene:

```text
changed files ASCII-only: yes
as any / @ts-ignore / @ts-expect-error in changed test files: none
git diff --check: clean, only normal LF-to-CRLF warnings
```

## Slice 1 - Kimi K2.6 - Corrections With Composed Profiles

Changed file:

```text
src/features/ontology/__tests__/corrections.test.ts
```

Added exactly 3 tests:

1. Overlay-only `correctedTypeNodeId` passes against `getActiveDomainProfile([overlay])` and fails against base `codingProfile`.
2. Overlay-only `previousTypeNodeId` passes against the composed profile and fails against base `codingProfile`.
3. Validation does not mutate the composed profile or the overlay input used to create it.

Why accepted:

- No production source changed.
- It used the public active-profile seam, not a private helper.
- It proved the important boundary: correction evidence is profile-parameter-driven and does not imply global profile mutation.
- Verification passed: tsc clean, focused tests 28/28, full suite 413/413 at slice time.

HR note:

- The first non-interactive Pi run exited 0 without edits because the local quiet-hours/go-to-bed extension wanted a second explicit confirmation in the same session. This is workflow friction, not a model-quality failure. The resumed same session completed correctly.

Score: 9/10.

## Slice 2 - Kimi K2.6 - `overrideOntology` Coverage

Changed file:

```text
src/features/ontology/__tests__/profileComposition.test.ts
```

Added exactly 3 tests:

1. `overrideOntology.itemTypeNodeIds` and `overrideOntology.relationshipTypeNodeIds` merge with base, dedupe, and do not mutate base.
2. `overrideOntology.nodes` adds a new node and deep-clones mutable nested fields, including boundary-rule `evidenceIds`.
3. `overrideOntology` additions compose with typed `addOntologyNodes`, `addItemTypeNodeIds`, and `addRelationshipTypeNodeIds` without duplicate ids.

Reviewer cleanup:

- Codex changed one fixture string from a late-night-specific phrase to a neutral profile-branch phrase.

Why accepted:

- No production source changed.
- Tests target existing implementation behavior without turning `overrideOntology` into persistence or runtime activation.
- Verification passed: tsc clean, profileComposition 23/23, full suite 416/416 at slice time.

Score: 9/10.

## Slice 3 - Qwen 3.6 Plus - Active-Profile Helper Seam

Changed file:

```text
src/features/ontology/__tests__/activeProfile.test.ts
```

Added exactly 3 tests:

1. `getOntologyNode(nodeId, composedProfile)` returns an overlay-added node, while default `getOntologyNode(nodeId)` does not see it.
2. `getOntologyNodeLabel(nodeId, composedProfile)` uses an overlay override of an existing base node label, while the default profile still uses the base label.
3. Two different composed profiles with the same added node id return their own labels and do not leak into each other or into the default profile.

Why accepted:

- No production source changed.
- The tests prove the helper functions are profile-parameter driven, not global-state driven.
- Verification passed: tsc clean, activeProfile 15/15, full suite 419/419 at slice time.

Miss:

- Final report briefly mixed historical and current suite-count arithmetic. The command output itself was correct.

Score: 8/10.

## Slice 4 - GLM 5.1 - Refactor Doc Sync

Changed files:

```text
ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
ONTOLOGY_PROFILE_REFACTOR/TOMORROW_START.md
ONTOLOGY_PROFILE_REFACTOR/implementation_handoff.md
```

Updated:

- current changed-files list
- Batch 2 and Batch 3 completion summaries
- latest verification counts: 93/93 focused and 419/419 full
- handoff bullets for corrections, active-profile seam hardening, `overrideOntology`, and doc 06 anchors
- next-decision warning: do not continue directly into correction UI or persistence

Reviewer cleanup:

- Codex removed `ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md` from the current changed-files list because it was stale from an older batch and not modified in the current worktree.
- Codex adjusted the local tool-folder note to reflect that Pi prompts/logs live in `C:\pi-stuff`, not a repo `.pi/` folder.

Why accepted:

- No source/product behavior changed.
- It preserved future architecture as future direction only.
- It did not invent the next product path.
- Verification passed: tsc clean, focused 93/93, full 419/419, stage10 39/39.

Score: 8/10.

## Workflow Lessons

1. Quiet-hours/go-to-bed extension:
   If non-interactive Pi exits 0 without edits because of a local late-night guard, do not score that as model failure. Either resume the same session with explicit continuation confirmation or run bounded HR workers with `--no-extensions` after the human explicitly asked to continue.

2. Qwen 3.6 Plus:
   Good for small active-profile/helper seam tests, but reviewer should verify final-report count arithmetic directly from command output.

3. GLM 5.1:
   Good for doc sync, but require `git diff --name-only` cross-checks for "current changed files" sections.

4. Kimi K2.6:
   Remains strong for strict bounded ontology test hardening, including correction validation and composition coverage.

## Not Done

- No correction persistence.
- No correction UI.
- No checker/suggestion model.
- No branch/overlay persistence.
- No real overlay activation source.
- No relationship-semantics rewrite.
- No MCP, adapter, Racket/DSL runtime, agent/subagent runtime, or self-building-app runtime.

The next major implementation path remains a human/Codex decision gate.
