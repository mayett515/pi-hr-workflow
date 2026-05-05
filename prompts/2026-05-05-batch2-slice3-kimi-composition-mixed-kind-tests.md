<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.
Repo: C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
Branch: refactor/ontology-profile

Extend src/features/ontology/__tests__/profileComposition.test.ts with three focused tests that close specific coverage gaps in the existing 21 tests: a 3-kind mixed-overlay end-to-end precedence test, a 3-overlay same-kind precedence chain test, and a no-op overlay (overlay with no override fields) equivalence test.
</task>

<context_files>
Read first:
- src/features/ontology/profileComposition.ts
- src/features/ontology/types.ts
- src/features/ontology/profiles/codingProfile.ts
- src/features/ontology/__tests__/profileComposition.test.ts (CURRENT 21 tests across multiple describe blocks; check before adding to avoid duplication)
- ONTOLOGY_PROFILE_REFACTOR/06_PROFILE_BRANCHING_AND_MERGE.md (decision #10 about composition precedence)
- ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
- ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
- C:\pi-stuff\FUTURE_PI_PROMPTING.md
</context_files>

<implementation_scope>
Allowed write scope (single file):
- src/features/ontology/__tests__/profileComposition.test.ts
</implementation_scope>

<do_not_edit>
Do not edit production code (composeDomainProfile, types.ts, codingProfile, index.ts, or anything else under src/).
Do not edit other test files.
Do not edit any docs.
Do not stage, commit, reset, checkout, or run destructive git commands.
</do_not_edit>

<goal>
Add EXACTLY THREE new tests, each as its own `describe` -> `it` pair OR as three `it(...)` inside one new describe block (your choice; keep style consistent with the surrounding file). No more than 3 new it(...) blocks total.

Required three tests:

1. Title: "mixed three-kind overlays apply project then learning then personal precedence per field"
   - Build three overlays:
     - project overlay: overrideLabels.hubTitle = "P-Hub", overrideGraph.nodeColors = { mechanism: "#111111" }, addItemTypeNodeIds = ["proj_only"]
     - learning overlay: overrideLabels.hubTitle = "L-Hub", overrideLabels.itemSingular = "Idea"
     - personal overlay: overrideLabels.hubTitle = "Me-Hub", overrideGraph.nodeColors = { mechanism: "#222222" }
   - Pass them in TWO orderings to composeDomainProfile and assert SAME outcome both times:
     a) input order [project, learning, personal]
     b) input order [personal, learning, project]
   - In BOTH outcomes, assert:
     - composed.labels.hubTitle === "Me-Hub" (personal wins per kind precedence)
     - composed.labels.itemSingular === "Idea" (learning beats project base; personal did not override)
     - composed.graph.nodeColors.mechanism === "#222222" (personal beats project for graph map merge of same key)
     - composed.ontology.itemTypeNodeIds includes "proj_only" (project addition still applied)
   - Also build a fourth scenario where personal does NOT override hubTitle; assert composed.labels.hubTitle === "L-Hub" (learning wins below personal).

2. Title: "three same-kind project overlays apply later-wins precedence deterministically"
   - Build three project overlays a, b, c each with overrideLabels.hubTitle = "A", "B", "C" respectively.
   - Pass [a, b, c] -> assert composed.labels.hubTitle === "C".
   - Pass [c, b, a] -> assert composed.labels.hubTitle === "A".
   - Pass [b, c, a] -> assert composed.labels.hubTitle === "A".
   - This proves later same-kind wins regardless of entry order, and is independent of the existing 2-overlay same-kind test.

3. Title: "no-op overlay (no override fields) is equivalent to empty overlay list for label and ontology"
   - Build a single overlay { id: "noop-1", kind: "project" } with NO override fields, NO add fields.
   - composedA = composeDomainProfile(base, [overlay])
   - composedB = composeDomainProfile(base, [])
   - Assert composedA and composedB are deep-equal for: labels, ontology.itemTypeNodeIds, ontology.relationshipTypeNodeIds, metadataFields lengths, and ontology.nodes length.
   - Assert overlay was not mutated (snapshot/restore comparison).
   - Both composedA.graph.nodeColors and composedB.graph.nodeColors should be deep-equal to base.graph.nodeColors (but not the same reference - the existing deep-clone tests already cover this; you do NOT need to re-add deep-clone reference checks here, just the equivalence).

Hard caps:
- Exactly 3 new it(...) blocks. No more, no fewer.
- File should grow by no more than ~150 lines.
- ASCII-only. No `as any`. No `@ts-ignore`. No `@ts-expect-error`.
- Reuse the existing `makeTestNode` and `makeOverlay` helpers - do not redefine them.

Avoid duplicating these existing tests:
- "personal overlay label override wins over project overlay" (already covers 2-overlay personal-vs-project)
- "later overlays of the same kind win deterministically" (already covers 2 same-kind)
- "inputs are not mutated" (already covers full immutability snapshot)
- All deep-clone reference tests (already cover not-shared references comprehensively)

Your three new tests must add real new coverage, not paraphrase existing ones.
</goal>

<constraints>
<scope_exception_policy>
Stay inside the allowed write scope. If TypeScript requires a downstream fix outside that scope, stop and report - do not edit production code.
</scope_exception_policy>

<optional_change_policy>
Do not refactor existing tests. Do not move helpers. Do not "improve" the existing fixture functions. Do not add a fourth test "for symmetry".
</optional_change_policy>

<composition_precedence_policy>
The expected precedence within composeDomainProfile is: base, then project overlays in input order, then learning overlays in input order, then personal overlays in input order. Same-kind later wins. Personal beats learning beats project for the same field. Your tests must assert outcomes that hold regardless of raw input order, by testing reversed input orderings explicitly.
</composition_precedence_policy>

<final_review_requirements>
Before final report:
1. Confirm exactly 3 new it(...) blocks were added.
2. Confirm only the allowed test file changed.
3. Run the verification commands.
4. Run the rg checks below.
5. Report the exact new test count, not "about 3".
</final_review_requirements>
</constraints>

<verification>
Run from C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn:

powershell:
- node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
- npm test -- --run src/features/ontology/__tests__/profileComposition.test.ts
- npm test -- --run

Report exit codes / pass-fail counts. profileComposition.test.ts had 21 tests before this slice; expect 24 after.
</verification>

<required_searches_before_final>
Run and report:
- rg -n "as any" src/features/ontology/__tests__/profileComposition.test.ts
- rg -n "@ts-(ignore|expect-error)" src/features/ontology/__tests__/profileComposition.test.ts
- rg -c "^\s*it\(" src/features/ontology/__tests__/profileComposition.test.ts (report exact count)
- rg -n "Me-Hub|L-Hub|noop-1" src/features/ontology/__tests__/profileComposition.test.ts
</required_searches_before_final>

<decision_policy>
If the task requires a product, architecture, persistence, UI, relationship-semantics, app-builder, agent/subagent runtime, MCP, adapter, DSL, open-source, pricing, or analytics decision not explicitly settled in this prompt, stop and report the decision needed. Do not invent the direction.
</decision_policy>

<final_report>
Return:
- files changed (must be exactly one: src/features/ontology/__tests__/profileComposition.test.ts)
- exact count of new it(...) blocks added (must be 3)
- previous and current it(...) total in the file (expected: 21 -> 24)
- tests run and result counts (focused file pass/fail, full suite pass/fail)
- search results from required_searches_before_final
- scope exceptions, if any (expected: none)
- anything not done
</final_report>
