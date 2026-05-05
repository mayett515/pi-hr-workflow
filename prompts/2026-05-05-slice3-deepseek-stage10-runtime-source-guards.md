<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.
Repo: C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
Branch: refactor/ontology-profile

Add source-level architecture guards that prevent premature runtime implementation of future Kortex directions.
</task>

<context_files>
Read first:
- ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
- ONTOLOGY_PROFILE_REFACTOR/implementation_handoff.md
- ONTOLOGY_PROFILE_REFACTOR/07_KORTEX_CORE_AND_CHILD_CORES.md
- ONTOLOGY_PROFILE_REFACTOR/08_KORTEX_LANGUAGE_LAYER_AND_ADAPTERS.md
- ONTOLOGY_PROFILE_REFACTOR/09_KORTEX_OVER_EXISTING_SYSTEMS.md
- src/__tests__/stage10-architecture-guards.test.ts
- src/features/ontology/index.ts
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
Add focused source guards for current-stage boundaries:
- getActiveDomainProfile must not introduce module-level mutable active overlay/profile state or setter functions such as setActiveDomainProfile, setActiveProfile, activeOverlays, or activeProfileStore.
- Future operation names from docs must not appear under src production code yet: DefineAgentCore, SetExecutionConstraint, GrantOperation, ForbidOperation, RequireApproval, DefineAppCore, DefineAppEntity, DefineAppWorkflow, AssignSubagent.
- No source implementation of profile overlay persistence table/string names yet: profile_overlays, profile_branches, active_profile_overlay.

Keep guard wording precise. Allowed hits in test files are fine when you filter them intentionally.
</goal>

<constraints>
Test-only slice. Do not edit production code if a guard initially fails; instead report the offender.
Keep comments ASCII-only.
</constraints>

<verification>
Run:
- node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
- npm test -- --run src/__tests__/stage10-architecture-guards.test.ts
</verification>

<required_searches_before_final>
Run:
- rg -n "DefineAgentCore|DefineAppCore|SetExecutionConstraint|GrantOperation|ForbidOperation|RequireApproval|AssignSubagent" src
- rg -n "profile_overlays|profile_branches|active_profile_overlay" src
- rg -n "setActiveDomainProfile|setActiveProfile|activeOverlays|activeProfileStore" src/features/ontology
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
