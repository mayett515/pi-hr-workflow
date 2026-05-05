<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.
Repo: C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
Branch: refactor/ontology-profile

Add focused tests that prove profile composition does not share mutable nested references with base profiles or overlays.
</task>

<context_files>
Read first:
- ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
- ONTOLOGY_PROFILE_REFACTOR/implementation_handoff.md
- C:\pi-stuff\FUTURE_PI_PROMPTING.md
- src/features/ontology/profileComposition.ts
- src/features/ontology/__tests__/profileComposition.test.ts
- src/features/ontology/profiles/codingProfile.ts
</context_files>

<implementation_scope>
Allowed write scope:
- src/features/ontology/__tests__/profileComposition.test.ts
</implementation_scope>

<do_not_edit>
Do not edit production code.
Do not edit docs.
Do not edit DB, persistence, UI, backup/import/export, MCP, adapters, Racket/DSL, app-builder, or agent runtime code.
Do not stage, commit, reset, checkout, or run destructive git commands.
</do_not_edit>

<goal>
Add tests for deep-clone/no-shared-reference behavior around composeDomainProfile.

Required coverage:
- Empty overlays produce a runtime profile whose mutable nested graph maps are not the same object references as the base graph maps.
- Empty overlays produce cloned ontology nodes and cloned node arrays, not the same object references as the base ontology nodes/arrays.
- An overlay-added ontology node is cloned into the result: result node, useWhen, examples, relatedNodeIds, contrastNodeIds must not be the same references as the overlay node.
- Metadata field override with enumOptions is cloned: output field, appliesTo, examples, enumOptions, and enum option objects must not share references with the overlay field.

Keep tests focused and readable. It is fine to add helper fixture data inside the existing test file.
</goal>

<constraints>
If a new test fails because production code has a real clone bug, stop and report the failing test. Do not edit production code in this slice.
Do not use `as any`.
Keep comments ASCII-only.
</constraints>

<verification>
Run:
- node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
- npm test -- --run src/features/ontology/__tests__/profileComposition.test.ts
</verification>

<required_searches_before_final>
Run:
- rg -n "as any" src/features/ontology/__tests__/profileComposition.test.ts
- rg -n "composeDomainProfile|ProfileOverlay" src/features/ontology/__tests__/profileComposition.test.ts
</required_searches_before_final>

<decision_policy>
If the task requires a product, architecture, persistence, UI, relationship-semantics, app-builder, agent/subagent runtime, MCP, adapter, DSL, open-source, pricing, or analytics decision not explicitly settled in this prompt, stop and report the decision needed. Do not invent the direction.
</decision_policy>

<final_report>
Return:
- files changed
- tests added
- tests/searches run and results
- scope exceptions, if any
- anything not done
</final_report>
