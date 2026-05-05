# Slice 1 - Correction Validation With Composed Profiles

You are a Pi worker on the CodeLens RN ontology-profile refactor.

Repo: `C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn`
Branch: `refactor/ontology-profile`

The human explicitly asked to continue this work now. If your local environment asks for midnight continuation confirmation, first run:

```bash
echo confirm-that-we-continue-after-midnight
```

Then continue the task.

<task>
Extend `src/features/ontology/__tests__/corrections.test.ts` with focused tests proving `validateOntologyCorrection()` validates against the profile it is given, including explicitly composed overlay profiles.
</task>

<context_files>
Read first:

- `ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md`
- `ONTOLOGY_PROFILE_REFACTOR/06_PROFILE_BRANCHING_AND_MERGE.md`
- `src/features/ontology/corrections.ts`
- `src/features/ontology/index.ts`
- `src/features/ontology/types.ts`
- `src/features/ontology/profileComposition.ts`
- `src/features/ontology/__tests__/corrections.test.ts`
- `src/features/ontology/__tests__/activeProfile.test.ts`
</context_files>

<allowed_write_scope>
Only edit:

- `src/features/ontology/__tests__/corrections.test.ts`
</allowed_write_scope>

<required_tests>
Add exactly 3 new `it(...)` tests:

1. A correction whose `correctedTypeNodeId` exists only in an explicit project overlay passes when validated against a composed active profile from `getActiveDomainProfile([overlay])`, and the same correction fails against base `codingProfile`.
2. A correction whose `previousTypeNodeId` exists only in an explicit project overlay passes when validated against that composed profile, and the same correction fails against base `codingProfile`.
3. Passing a composed profile into `validateOntologyCorrection()` does not mutate the composed profile or the overlay input used to create it.
</required_tests>

<compatibility_boundaries>
- Do not change `validateOntologyCorrection()`.
- Do not add persistence, DB tables, UI, checker execution, patch suggestion storage, profile mutation helpers, or automatic correction application.
- Do not widen `OntologyCorrectionField`, `OntologyCorrectionSource`, or `OntologyCorrectionSubjectKind`.
- Do not rename legacy concept/capture/database fields.
- Do not introduce relationship-taxonomy changes.
</compatibility_boundaries>

<implementation_notes>
- Reuse the existing `validCorrection()` helper where practical.
- You may add a tiny local test-node helper if needed.
- Prefer `getActiveDomainProfile([overlay])` to prove the public seam works with correction validation.
- Do not use `as any`, `@ts-ignore`, or `@ts-expect-error`.
- Keep assertions diagnostic: the overlay-only type string must not already be present in `codingProfile.ontology.itemTypeNodeIds`.
</implementation_notes>

<decision_policy>
If this task appears to require a product decision about correction UI, correction persistence, checker suggestions, branch persistence, overlay activation source, relationship semantics, agent/subagent runtime, MCP, adapters, or DSL/runtime direction, stop and report the needed decision. Do not invent the direction.
</decision_policy>

<verification>
Run:

```powershell
node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
npm test -- --run src/features/ontology/__tests__/corrections.test.ts src/features/ontology/__tests__/activeProfile.test.ts
```

If time permits, also run:

```powershell
npm test -- --run
```
</verification>

<final_report>
Report:

- files changed
- number and names of new tests
- exact verification commands and results
- any residual risk
- confirmation that no production source changed
</final_report>
