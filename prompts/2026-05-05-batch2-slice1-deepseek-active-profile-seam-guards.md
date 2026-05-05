<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.
Repo: C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
Branch: refactor/ontology-profile

Add focused tests that close three specific coverage gaps in the active-profile overlay seam: frozen-overlay-input safety, three-kind mixed overlay composition at the seam, and direct reference-equality between the no-arg and empty-array forms of getActiveDomainProfile.
</task>

<context_files>
Read first:
- src/features/ontology/index.ts
- src/features/ontology/profileComposition.ts
- src/features/ontology/types.ts
- src/features/ontology/profiles/codingProfile.ts
- src/features/ontology/__tests__/activeProfile.test.ts (current 8 tests; do not duplicate existing coverage)
- ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
- ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
- C:\pi-stuff\FUTURE_PI_PROMPTING.md
</context_files>

<implementation_scope>
Allowed write scope (single file):
- src/features/ontology/__tests__/activeProfile.test.ts
</implementation_scope>

<do_not_edit>
Do not edit production code (no edits in src/features/ontology/index.ts, profileComposition.ts, types.ts, profiles/codingProfile.ts, or anywhere else in src/ outside the allowed test file).
Do not edit other test files.
Do not edit any docs.
Do not edit DB, persistence, UI, backup/import/export, MCP, adapters, Racket/DSL, app-builder, or agent runtime code.
Do not stage, commit, reset, checkout, or run destructive git commands.
</do_not_edit>

<goal>
Add exactly four focused tests inside the existing `describe('getActiveDomainProfile overlay seam', ...)` block. No new describe blocks. No new helper exports. You may extend the local `makeTestNode` helper if needed, or add a small in-file fixture.

Required four tests, in this order:

1. "no-arg and empty-array calls return the same singleton reference" - assert `getActiveDomainProfile() === getActiveDomainProfile([])` and that both `===` `codingProfile`.

2. "frozen overlay input does not throw and produces correct composition" - construct an overlay (with overrideLabels.hubTitle plus overrideGraph.nodeColors plus addOntologyNodes containing one frozen node), pass `Object.freeze(overlay)` (also Object.freeze the nested overrideLabels object, the nested overrideGraph object, the nested overrideGraph.nodeColors object, the nested addOntologyNodes array, and the inner overlay node) to getActiveDomainProfile, then assert: no throw, composed.labels.hubTitle is the overlay value, composed.graph.nodeColors.mechanism is the overlay value, the added ontology node is present in composed.ontology.nodes, and codingProfile remains unmutated (deep-equal to a JSON snapshot taken before the call).

3. "mixed three-kind overlay composition applies project then learning then personal precedence at the seam" - build three overlays (project / learning / personal). Each overrides labels.hubTitle. Personal also adds one ontology node and one itemTypeNodeId. Pass them in input order [project, learning, personal] AND in input order [personal, learning, project]; assert that in BOTH cases composed.labels.hubTitle ends up as the personal value, and the personal-added ontology node id appears in composed.ontology.itemTypeNodeIds. Then for one of the orderings, also assert that learning's hubTitle appeared "below" personal in precedence by checking that swapping personal's overrideLabels off (rebuild a 4th overlay set with personal having no hubTitle override) yields learning's value.

4. "passing the same overlay reference twice in one call is treated as two ordered overlays of that kind" - construct two project overlays a and b with overrideLabels.hubTitle set to "A" and "B" respectively; pass [a, b]; assert composed.labels.hubTitle === "B" (later same-kind wins). Then pass [b, a] and assert composed.labels.hubTitle === "A". This is the seam-level mirror of the same-kind precedence rule already covered in profileComposition.test.ts; the new test confirms the seam preserves it.

Hard caps:
- Exactly 4 new it(...) blocks. No more, no fewer.
- Total file should grow by no more than ~120 lines.
- ASCII-only. No `as any`. No `@ts-ignore`. No `@ts-expect-error`.
</goal>

<constraints>
<scope_exception_policy>
Stay inside the allowed write scope. If TypeScript requires a downstream call-site fix outside that scope, stop and report - do not edit production code in this slice.
</scope_exception_policy>

<optional_change_policy>
Do not add optional improvements. Do not refactor existing tests. Do not rename variables in existing tests. Do not add unrelated assertions.
</optional_change_policy>

<final_review_requirements>
Before final report:
1. Run the verification commands.
2. Run the rg checks listed below.
3. Inspect the diff to confirm only the allowed test file changed.
4. Confirm exactly 4 new it(...) blocks were added.
5. Report the EXACT count, not an approximation.
</final_review_requirements>

If any new test fails because production code has a real bug (for example composition is not actually frozen-input safe), STOP and report the failing test in the final report. Do not edit production code in this slice.
</constraints>

<verification>
Run from C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn:

powershell:
- node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
- npm test -- --run src/features/ontology/__tests__/activeProfile.test.ts
- npm test -- --run

Report exit codes/result counts for each.
</verification>

<required_searches_before_final>
Run from the repo root and report counts:
- rg -n "as any" src/features/ontology/__tests__/activeProfile.test.ts
- rg -n "@ts-(ignore|expect-error)" src/features/ontology/__tests__/activeProfile.test.ts
- rg -n "^  it\(" src/features/ontology/__tests__/activeProfile.test.ts | wc -l   (Windows: use Measure-Object)
- rg -n "Object\.freeze" src/features/ontology/__tests__/activeProfile.test.ts
</required_searches_before_final>

<decision_policy>
If the task requires a product, architecture, persistence, UI, relationship-semantics, app-builder, agent/subagent runtime, MCP, adapter, DSL, open-source, pricing, or analytics decision not explicitly settled in this prompt, stop and report the decision needed. Do not invent the direction.
</decision_policy>

<final_report>
Return:
- files changed (must be exactly one: the allowed test file)
- exact count of new it(...) blocks added (must be 4)
- tests run and result counts (focused file pass/fail, full suite pass/fail)
- search results from required_searches_before_final
- scope exceptions, if any
- legacy names intentionally left and why (expected: none added)
- anything not done
</final_report>
