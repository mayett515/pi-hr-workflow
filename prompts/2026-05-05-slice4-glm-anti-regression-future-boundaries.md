<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.
Repo: C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn
Branch: refactor/ontology-profile

Update the anti-regression rules doc so future workers preserve the new Kortex architecture boundaries.
</task>

<context_files>
Read first:
- ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md
- ONTOLOGY_PROFILE_REFACTOR/implementation_handoff.md
- ONTOLOGY_PROFILE_REFACTOR/07_KORTEX_CORE_AND_CHILD_CORES.md
- ONTOLOGY_PROFILE_REFACTOR/08_KORTEX_LANGUAGE_LAYER_AND_ADAPTERS.md
- ONTOLOGY_PROFILE_REFACTOR/09_KORTEX_OVER_EXISTING_SYSTEMS.md
- ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
</context_files>

<implementation_scope>
Allowed write scope:
- ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
</implementation_scope>

<do_not_edit>
Do not edit source code.
Do not edit tests.
Do not edit DB, persistence, UI, backup/import/export, MCP, adapters, Racket/DSL, app-builder, or agent runtime code.
Do not stage, commit, reset, checkout, or run destructive git commands.
</do_not_edit>

<goal>
Add concise anti-regression rules for the future architecture directions already documented:
- Kortex Core must not depend on CodeLens UI or coding-only assumptions.
- Agent/subagent execution ontology is future architecture only for now: no runtime/orchestration/permission enforcement/MCP policy tools without explicit approval.
- Self-building app framework is future architecture only for now: no app-builder runtime, code-generation orchestration, generated-app persistence, or source write-back without explicit approval.
- Language/DSL direction is future architecture only for now: no Racket runtime or DSL implementation in this branch without explicit approval.
- Overlay-over-existing-systems is future architecture only for now: no source sync, static analysis adapters, file watchers, MCP, or write-back without explicit approval.
- Active-profile overlays are explicit-only at this stage: no hidden global active overlay state, no persistence, no automatic profile mutation.

Add matching refactor gates where useful. Keep the doc concise and easy for future agents to scan.
</goal>

<constraints>
Docs-only slice. Preserve existing sections and wording unless adding the new guardrails requires a small local edit.
Keep comments/text ASCII-only.
</constraints>

<verification>
Run:
- rg -n "agent|subagent|self-building|Racket|MCP|active overlay|profile overlay|write-back|source sync" ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md
</verification>

<decision_policy>
If the task requires a product, architecture, persistence, UI, relationship-semantics, app-builder, agent/subagent runtime, MCP, adapter, DSL, open-source, pricing, or analytics decision not explicitly settled in this prompt, stop and report the decision needed. Do not invent the direction.
</decision_policy>

<final_report>
Return:
- files changed
- doc sections added/changed
- searches run and results
- scope exceptions, if any
- anything not done
</final_report>
