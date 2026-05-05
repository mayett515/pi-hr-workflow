# Slice 2 - Profile Composition Override Ontology Coverage

You are a Pi worker on the CodeLens RN ontology-profile refactor.

Repo: `C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn`
Branch: `refactor/ontology-profile`

The human explicitly asked to continue this work now. If your local environment asks for midnight continuation confirmation, first run:

```bash
echo confirm-that-we-continue-after-midnight
```

Then continue the task.

<task>
Extend `src/features/ontology/__tests__/profileComposition.test.ts` with focused tests for the existing `overrideOntology` composition path.
</task>

<context_files>
Read first:

- `ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md`
- `ONTOLOGY_PROFILE_REFACTOR/06_PROFILE_BRANCHING_AND_MERGE.md`
- `src/features/ontology/profileComposition.ts`
- `src/features/ontology/types.ts`
- `src/features/ontology/__tests__/profileComposition.test.ts`
</context_files>

<allowed_write_scope>
Only edit:

- `src/features/ontology/__tests__/profileComposition.test.ts`
</allowed_write_scope>

<required_tests>
Add exactly 3 new `it(...)` tests:

1. `overrideOntology.itemTypeNodeIds` and `overrideOntology.relationshipTypeNodeIds` merge with the base, dedupe duplicates, and do not mutate the base.
2. `overrideOntology.nodes` adds a new ontology node and deep-clones mutable nested node fields (`useWhen`, `examples`, `relatedNodeIds`, `contrastNodeIds`, `doNotUseWhen`, and `evidenceIds`).
3. `overrideOntology` additions compose cleanly with typed overlay additions (`addOntologyNodes`, `addItemTypeNodeIds`, `addRelationshipTypeNodeIds`) without duplicate item/relationship ids.
</required_tests>

<compatibility_boundaries>
- Do not change `composeDomainProfile()` or production source.
- Do not turn `overrideOntology` into persistence, patch storage, or a UI/runtime activation source.
- Do not reinterpret relationship semantics or introduce a final global taxonomy.
- Do not add branch persistence or profile overlay storage.
</compatibility_boundaries>

<implementation_notes>
- Reuse existing `makeTestNode()` and `makeOverlay()` fixtures where practical.
- Keep the tests focused on current documented behavior only.
- Do not add brittle assertions about every node count unless needed for the specific behavior.
- Do not use `as any`, `@ts-ignore`, or `@ts-expect-error`.
</implementation_notes>

<decision_policy>
If this task appears to require a product decision about persistence, UI/runtime activation, relationship semantics, correction/checker behavior, agent/subagent runtime, MCP, adapters, or DSL/runtime direction, stop and report the needed decision. Do not invent the direction.
</decision_policy>

<verification>
Run:

```powershell
node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
npm test -- --run src/features/ontology/__tests__/profileComposition.test.ts
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
