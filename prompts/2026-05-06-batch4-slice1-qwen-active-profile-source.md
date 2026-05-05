# 2026-05-06 Batch 4 Slice 1 - Qwen Active Profile Source

<task>
You are a Pi worker on the CodeLens RN ontology-profile refactor.

Repo: `C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn`
Branch: `refactor/ontology-profile`

Add an internal-only explicit active-profile source helper. This is the next small step after the existing `getActiveDomainProfile(overlays?)` seam: callers should be able to pass a structured source object containing a base profile and overlays, without adding any persistence, UI, global state, automatic activation, or profile mutation.
</task>

<context_files>
Read first:

1. `ONTOLOGY_PROFILE_REFACTOR/NEXT_LLM_CONTEXT.md`
2. `ONTOLOGY_PROFILE_REFACTOR/06_PROFILE_BRANCHING_AND_MERGE.md`
3. `ONTOLOGY_PROFILE_REFACTOR/05_ANTI_REGRESSION_RULES.md`
4. `src/features/ontology/types.ts`
5. `src/features/ontology/index.ts`
6. `src/features/ontology/profileComposition.ts`
7. `src/features/ontology/__tests__/activeProfile.test.ts`
8. `src/__tests__/stage10-architecture-guards.test.ts`
</context_files>

<implementation_scope>
Allowed write scope:

- `src/features/ontology/types.ts`
- `src/features/ontology/index.ts`
- `src/features/ontology/profileActivation.ts` (new file if useful)
- `src/features/ontology/__tests__/profileActivation.test.ts` (new file)
- `src/features/ontology/__tests__/activeProfile.test.ts` only if needed for `getActiveDomainProfile` compatibility coverage

Do not edit files outside this list unless TypeScript requires the smallest possible export/call-site fix. Report any scope exception.
</implementation_scope>

<do_not_edit>
Do not edit:

- DB schema, migrations, backup, import/export, or persistence files
- UI files
- correction evidence files unless TypeScript requires an import-only fix
- graph, learning, promotion, retrieval, review, chat, or app shell files
- architecture docs or HR docs
</do_not_edit>

<goal>
Implement a tiny pure helper around the existing composition seam:

1. Add an exported type in `types.ts`:

```ts
export interface ActiveDomainProfileSource<TItemTypeNodeId extends string = string> {
  baseProfile: DomainProfile<TItemTypeNodeId>;
  overlays?: readonly ProfileOverlay<TItemTypeNodeId>[] | null | undefined;
}
```

If exactOptionalPropertyTypes requires a slightly different spelling, keep the same semantics.

2. Add a pure resolver, preferably in new `src/features/ontology/profileActivation.ts`:

```ts
export function resolveActiveDomainProfile<TItemTypeNodeId extends string = string>(
  source: ActiveDomainProfileSource<TItemTypeNodeId>,
): DomainProfile<TItemTypeNodeId>
```

Behavior:

- If `source.overlays` is omitted, `null`, or empty, return `source.baseProfile` by reference.
- If overlays are present, return `composeDomainProfile(source.baseProfile, source.overlays)`.
- Do not mutate `source`, `source.baseProfile`, or any overlay.
- Do not cache results.
- Do not add module-level mutable active overlay/profile state.

3. Re-export the type and resolver from `src/features/ontology/index.ts`.

4. Refactor `getActiveDomainProfile(overlays?)` to use the resolver internally while preserving existing behavior:

- `getActiveDomainProfile()` returns `codingProfile` by reference.
- `getActiveDomainProfile([])` returns `codingProfile` by reference.
- `getActiveDomainProfile([overlay])` composes explicitly supplied overlays.
</goal>

<constraints>
- Keep this internal-only and pure.
- No DB, UI, persistence, storage, global store, selector hook, setter, or automatic activation.
- Do not add names banned by existing architecture guards: `setActiveDomainProfile`, `setActiveProfile`, `activeOverlays`, `activeProfileStore`.
- Do not add profile overlay persistence names/tables.
- Do not implement branch export, compare mode, correction checker, app-builder, agent/subagent runtime, MCP, adapter, Racket, or DSL behavior.
- Preserve all current coding behavior.
- Avoid `as any`, `@ts-ignore`, and `@ts-expect-error`.
- Keep new code/comments ASCII-only.
</constraints>

<required_tests>
Add focused tests in `src/features/ontology/__tests__/profileActivation.test.ts` for:

1. `resolveActiveDomainProfile` returns the exact base profile reference when overlays are omitted.
2. It returns the exact base profile reference when overlays are `null`.
3. It returns the exact base profile reference when overlays are an empty array.
4. It composes explicit overlays and returns a different object with the overlay changes.
5. It does not mutate the source object, base profile, or overlay input.
6. Repeated calls with the same source and overlays return equal but distinct composed objects, proving no cache/global state.
7. It works with a non-default base profile object, not only the imported `codingProfile` singleton.

If you touch `activeProfile.test.ts`, only add a minimal compatibility assertion proving `getActiveDomainProfile` still preserves no-arg/empty-array/reference behavior and explicit-overlay behavior through the resolver.
</required_tests>

<required_searches_before_final>
Run and summarize:

```powershell
rg -n "setActiveDomainProfile|setActiveProfile|activeOverlays|activeProfileStore" src/features/ontology src/__tests__/stage10-architecture-guards.test.ts
rg -n "profile_overlays|profile_branches|active_profile_overlay" src
rg -n "resolveActiveDomainProfile|ActiveDomainProfileSource|profileActivation" src/features/ontology
```

The first search should only find existing stage10 guard text if any, not production implementation.
The second search should only find tests/guards if any, not production persistence.
</required_searches_before_final>

<verification>
Run:

```powershell
node node_modules/typescript/bin/tsc -p tsconfig.json --noEmit
npm test -- --run src/features/ontology/__tests__/profileActivation.test.ts src/features/ontology/__tests__/activeProfile.test.ts src/__tests__/stage10-architecture-guards.test.ts
```
</verification>

<decision_policy>
If the task seems to require a product, architecture, persistence, UI, relationship-semantics, app-builder, agent/subagent runtime, MCP, adapter, DSL, open-source, pricing, or analytics decision not explicitly settled in this prompt, stop and report the decision needed. Do not invent the direction.
</decision_policy>

<final_report>
Return:

- files changed
- required changes made
- optional changes made, if any
- scope exceptions, if any
- tests run and results
- rg/search output summary
- compatibility boundaries preserved
- anything not done
</final_report>
