<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.
Repo: C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
Branch: refactor/ontology-profile

Add small architecture guard tests that ensure durable docs keep the new Kortex future-direction anchors visible.
</task>

<context_files>
Read first:
- ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
- ONTOLOGY_PROFILE_REFACTOR/implementation_handoff.md
- ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
- ONTOLOGY_PROFILE_REFACTOR/07_KORTEX_CORE_AND_CHILD_CORES.md
- ONTOLOGY_PROFILE_REFACTOR/08_KORTEX_LANGUAGE_LAYER_AND_ADAPTERS.md
- ONTOLOGY_PROFILE_REFACTOR/09_KORTEX_OVER_EXISTING_SYSTEMS.md
- src/__tests__/stage10-architecture-guards.test.ts
</context_files>

<implementation_scope>
Allowed write scope:
- src/__tests__/stage10-architecture-guards.test.ts
</implementation_scope>

<do_not_edit>
Do not edit production code.
Do not edit docs.
Do not edit DB, persistence, UI, backup/import/export, MCP, adapters, Racket/DSL, app-builder, or agent runtime code.
Do not stage, commit, reset, checkout, or run destructive git commands.
</do_not_edit>

<goal>
Add docs-aware architecture guard tests that read the durable markdown files and assert the future-direction anchors remain present:
- 07_KORTEX_CORE_AND_CHILD_CORES.md contains "Agent Execution Ontology" and "Self-Building App Framework Direction".
- 08_KORTEX_LANGUAGE_LAYER_AND_ADAPTERS.md contains future operation anchors "DefineAgentCore" and "DefineAppCore".
- 09_KORTEX_OVER_EXISTING_SYSTEMS.md contains "Kortex Over Self-Building Apps".
- NEXT_LLM_CONTEXT.md contains the important agent/subagent caution and important self-building-app caution.
- 05_ANTI_REGRESSION_RULES.md contains the future boundary wording added by the previous docs slice.

Keep the tests robust enough to catch accidental deletion, but not so brittle that normal wording edits break them.
</goal>

<constraints>
Test-only slice. Do not edit docs or production code if a guard fails; report the missing anchor instead.
Keep comments ASCII-only.
</constraints>

<verification>
Run:
- node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
- npm test -- --run src/__tests__/stage10-architecture-guards.test.ts
</verification>

<required_searches_before_final>
Run:
- rg -n "Agent Execution Ontology|Self-Building App Framework|DefineAgentCore|DefineAppCore|Kortex Over Self-Building Apps|future boundary" ONTOLOGY_PROFILE_REFACTOR src/__tests__/stage10-architecture-guards.test.ts
</required_searches_before_final>

<decision_policy>
If the task requires a product, architecture, persistence, UI, relationship-semantics, app-builder, agent/subagent runtime, MCP, adapter, DSL, open-source, pricing, or analytics decision not explicitly settled in this prompt, stop and report the decision needed. Do not invent the direction.
</decision_policy>

<final_report>
Return:
- files changed
- guards added
- tests/searches run and results
- scope exceptions, if any
- anything not done
</final_report>
