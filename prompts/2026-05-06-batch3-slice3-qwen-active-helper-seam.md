# Slice 3 - Active Profile Ontology Helper Seam Coverage

You are a Pi worker on the CodeLens RN ontology-profile refactor.

Repo: `C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn`
Branch: `refactor/ontology-profile`

The human explicitly asked to continue this work now. If your local environment asks for midnight continuation confirmation, first run:

```bash
echo confirm-that-we-continue-after-midnight
```

Then continue the task.

<task>
Extend `src/features/ontology/__tests__/activeProfile.test.ts` with focused tests proving the ontology helper functions remain profile-parameter driven and do not rely on hidden active overlay state.
</task>

<context_files>
Read first:

- `ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md`
- `src/features/ontology/index.ts`
- `src/features/ontology/profileComposition.ts`
- `src/features/ontology/types.ts`
- `src/features/ontology/__tests__/activeProfile.test.ts`
</context_files>

<allowed_write_scope>
Only edit:

- `src/features/ontology/__tests__/activeProfile.test.ts`
</allowed_write_scope>

<required_tests>
Add exactly 3 new `it(...)` tests:

1. `getOntologyNode(nodeId, composedProfile)` returns an overlay-added node, while `getOntologyNode(nodeId)` without the composed profile does not see it.
2. `getOntologyNodeLabel(nodeId, composedProfile)` uses an overlay override of an existing node label, while `getOntologyNodeLabel(nodeId)` without the composed profile still uses the base coding label.
3. Helper calls remain parameter-driven across two different composed profiles: two separate overlay profiles with different labels for the same added node must return their own labels and not leak into each other or into the default profile.
</required_tests>

<compatibility_boundaries>
- Do not change `getActiveDomainProfile()`, `getOntologyNode()`, or `getOntologyNodeLabel()`.
- Do not add a global active profile selector, setter, store, cache, persistence, UI, or runtime activation source.
- Do not add source adapters, MCP, agent/subagent runtime, or relationship-taxonomy changes.
</compatibility_boundaries>

<implementation_notes>
- Import `getOntologyNode` from `../index` if needed.
- Reuse the existing `makeTestNode()` helper where practical; update it only if the tests need distinct labels.
- Do not use `as any`, `@ts-ignore`, or `@ts-expect-error`.
- Keep assertions diagnostic and avoid duplicating existing tests unless the helper behavior is new.
</implementation_notes>

<decision_policy>
If this task appears to require a product decision about active overlay source, persistence, UI, branch state, relationship semantics, agent/subagent runtime, MCP, adapters, or DSL/runtime direction, stop and report the needed decision. Do not invent the direction.
</decision_policy>

<verification>
Run:

```powershell
node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
npm test -- --run src/features/ontology/__tests__/activeProfile.test.ts
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
