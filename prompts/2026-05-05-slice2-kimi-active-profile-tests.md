<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.
Repo: C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
Branch: refactor/ontology-profile

Add focused tests for the explicit active-profile overlay seam.
</task>

<context_files>
Read first:
- ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
- ONTOLOGY_PROFILE_REFACTOR/implementation_handoff.md
- C:\pi-stuff\FUTURE_PI_PROMPTING.md
- src/features/ontology/index.ts
- src/features/ontology/profileComposition.ts
- src/features/ontology/__tests__/activeProfile.test.ts
- src/features/ontology/__tests__/profileComposition.test.ts
</context_files>

<implementation_scope>
Allowed write scope:
- src/features/ontology/__tests__/activeProfile.test.ts
</implementation_scope>

<do_not_edit>
Do not edit production code.
Do not edit docs.
Do not edit DB, persistence, UI, backup/import/export, MCP, adapters, Racket/DSL, app-builder, or agent runtime code.
Do not stage, commit, reset, checkout, or run destructive git commands.
</do_not_edit>

<goal>
Add focused tests proving:
- Personal overlays win through getActiveDomainProfile(overlays) even when the personal overlay is listed before a project overlay.
- Two calls to getActiveDomainProfile([overlay]) return separate composed runtime objects and do not cache/mutate the prior result.
- A composed active profile does not share mutable graph nested maps with codingProfile.
- The overlay input object is not mutated by getActiveDomainProfile([overlay]).

Keep existing tests and add only narrow assertions around the active-profile seam.
</goal>

<constraints>
If a new test fails because production code has a real bug, stop and report the failing test. Do not edit production code in this slice.
Do not use `as any`.
Keep comments ASCII-only.
</constraints>

<verification>
Run:
- node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
- npm test -- --run src/features/ontology/__tests__/activeProfile.test.ts
</verification>

<required_searches_before_final>
Run:
- rg -n "as any" src/features/ontology/__tests__/activeProfile.test.ts
- rg -n "getActiveDomainProfile|ProfileOverlay" src/features/ontology/__tests__/activeProfile.test.ts
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
