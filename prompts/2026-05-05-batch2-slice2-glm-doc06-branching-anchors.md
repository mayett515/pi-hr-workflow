<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.
Repo: C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
Branch: refactor/ontology-profile

Add capped, focused stage10 doc-anchor guard tests that protect the durable section identity of ONTOLOGY_PROFILE_REFACTOR/06_PROFILE_BRANCHING_AND_MERGE.md. The previous batch already added doc-anchor blocks for docs 05/07/08/09/NEXT_LLM_CONTEXT.md; doc 06 is the only durable refactor doc not yet anchor-guarded.
</task>

<context_files>
Read first:
- ONTOLOGY_PROFILE_REFACTOR/06_PROFILE_BRANCHING_AND_MERGE.md (the doc to anchor)
- src/__tests__/stage10-architecture-guards.test.ts (look for the existing block: `describe('Kortex durable doc future-direction anchor guards', ...)` at the bottom, lines ~369-419, currently 5 tests covering docs 07/08/09/NEXT_LLM_CONTEXT/05). DO NOT MODIFY the existing 5 tests in that block; only ADD a new test.
- ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
- ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
- C:\pi-stuff\FUTURE_PI_PROMPTING.md
- C:\pi-stuff\HR_REPORT_2026-05-05_ONTOLOGY_GUARD_BATCH.md (read the Mimo failure summary - over-guarded with too many anchors; we do NOT repeat that)
</context_files>

<implementation_scope>
Allowed write scope (single file):
- src/__tests__/stage10-architecture-guards.test.ts
</implementation_scope>

<do_not_edit>
Do not edit any other source file.
Do not edit any markdown doc, including 06_PROFILE_BRANCHING_AND_MERGE.md.
Do not modify the existing 5 tests inside `describe('Kortex durable doc future-direction anchor guards', ...)`. Only ADD ONE new it(...) block to that describe at the end.
Do not edit any other describe block in stage10-architecture-guards.test.ts.
Do not stage, commit, reset, checkout, or run destructive git commands.
</do_not_edit>

<goal>
Add EXACTLY ONE new `it(...)` block at the end of the existing `describe('Kortex durable doc future-direction anchor guards', ...)` block, titled:

  "keeps profile branching, layering, and merge anchors in doc 06"

Inside that single test, assert exactly FIVE durable anchor strings are present in 06_PROFILE_BRANCHING_AND_MERGE.md. Use the existing `readDoc('06_PROFILE_BRANCHING_AND_MERGE.md')` helper pattern from the file. The five anchors must be:

1. The "Architecture Decisions" durable header. Anchor literal: "## Architecture Decisions"
2. The composition precedence decision phrase from decision #10. Anchor literal (use `.toContain`, exact substring): "personal corrections win, then active project/learning overlay, then base"
3. The conflict-rules durable phrase. Anchor literal: "Active branch rules win for classification inside that branch."
4. The implementation-sequence durable phrase from "Implementation Implications". Anchor literal: "Add pure composition helpers that produce a composed runtime profile from base + overlays."
5. The "What A Branch Can Change" durable section header. Anchor literal: "## What A Branch Can Change"

That is FIVE assertions, in ONE test, in ONE new it(...) block. No more.

Hard caps:
- Exactly 1 new it(...) block.
- Exactly 5 expect(...).toContain(...) assertions inside it.
- No new describe blocks.
- No new helpers.
- No new imports (the test file already imports fs/path/vitest).
- ASCII-only. No `as any`, no @ts-ignore, no @ts-expect-error.
- Total added lines should be approximately 8-15 lines.

Anti-pattern reminder (the Mimo failure mode): doc-anchor guards must protect durable section IDENTITY (5 anchors max), not pin every sentence. Any anchor you add must be a substring of an actual durable header or a durable conceptual phrase that, if removed, would mean the section's intent is gone. Do NOT anchor to one-off prose, examples, or numeric values that may be edited.
</goal>

<constraints>
<scope_exception_policy>
Stay strictly inside the allowed write scope. If TypeScript or Vitest forces any change outside the allowed file, stop and report - do not edit anything else.
</scope_exception_policy>

<optional_change_policy>
Do not refactor existing anchor tests. Do not factor out a shared helper. Do not improve naming. Do not add a sixth or seventh anchor "for completeness".
</optional_change_policy>

<final_review_requirements>
Before final report:
1. Verify ONLY src/__tests__/stage10-architecture-guards.test.ts changed (run git status / git diff --stat or `rg --files -l` style check).
2. Verify the new test contains exactly 5 assertions.
3. Verify all 5 anchor strings exist verbatim in 06_PROFILE_BRANCHING_AND_MERGE.md (the new test must pass).
4. Run the full verification commands.
</final_review_requirements>
</constraints>

<verification>
Run from C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn:

powershell:
- node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
- npm test -- --run src/__tests__/stage10-architecture-guards.test.ts
- npm test -- --run

Report exit codes / pass-fail counts. The stage10 file should have exactly ONE more passing test than before.
</verification>

<required_searches_before_final>
Run and report:
- rg -n "06_PROFILE_BRANCHING_AND_MERGE" src/__tests__/stage10-architecture-guards.test.ts
- rg -n "^    it\(" src/__tests__/stage10-architecture-guards.test.ts (count - report number; previous batch left this at 5 inside the doc-anchor describe; expect 6 after this slice)
- rg -n "Architecture Decisions|What A Branch Can Change" ONTOLOGY_PROFILE_REFACTOR/06_PROFILE_BRANCHING_AND_MERGE.md
</required_searches_before_final>

<decision_policy>
If the task requires a product, architecture, persistence, UI, relationship-semantics, app-builder, agent/subagent runtime, MCP, adapter, DSL, open-source, pricing, or analytics decision not explicitly settled in this prompt, stop and report the decision needed. Do not invent the direction.
</decision_policy>

<final_report>
Return:
- files changed (must be exactly one: src/__tests__/stage10-architecture-guards.test.ts)
- exact count of new it(...) blocks added (must be 1)
- exact count of assertions inside the new test (must be 5)
- tests run and result counts (focused stage10 file pass/fail with before/after counts, full suite pass/fail)
- search results from required_searches_before_final
- scope exceptions, if any (expected: none)
- anything not done
</final_report>
