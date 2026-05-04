# CodeLens Fresh Start Prompt

Use this prompt when starting a fresh Codex session for the CodeLens RN ontology-profile refactor and Pi worker-model HR workflow.

```markdown
You are helping on the CodeLens RN ontology-profile refactor.

Repo:
C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn

Branch:
refactor/ontology-profile

Pi HR folder:
C:\pi-stuff

First read:
- C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn\ONTOLOGY_PROFILE_REFACTOR\implementation_handoff.md
- C:\Projects\CodeLensApp\CodeLens-v2\codelens-rn\ONTOLOGY_PROFILE_REFACTOR\05_ANTI_REGRESSION_RULES.md
- C:\pi-stuff\hrworkflow.md
- C:\pi-stuff\HR_DATABASE.md
- C:\pi-stuff\FUTURE_PI_PROMPTING.md
- C:\pi-stuff\model_hr_db.json

Current refactor state:
- Ontology profile seam exists under `src/features/ontology/`.
- Default `codingProfile` owns current coding labels, ontology node labels, graph colors, metadata labels/placeholders, extraction wording, retrieval labels, promotion defaults, and relationship labels.
- Migration 011 added profile compatibility columns and dual-read/dual-write paths.
- Backup/import/export compatibility is complete with explicit raw DB column maps.
- Graph internals use `typeNodeId`.
- Promotion internals use `typeNodeId` / `proposedTypeNodeId`.
- Retrieval payloads use `typeNodeId`; retrieval filters prefer `typeNodeIds` and keep `conceptTypes` as a legacy alias.
- UI primitive is `TypeNodeChip`; `ConceptTypeChip` remains a deprecated wrapper preserving the old `type` prop.
- `ConceptListFilters` prefers `typeNodeIds` and keeps `conceptType` as a legacy alias.
- Current full suite after the last reviewed worker slice: 333/333 tests passed across 49 files.

Do not:
- Rename `LearningConcept.conceptType` yet.
- Rename `ConceptHint.proposedConceptType` yet.
- Rename DB columns or tables.
- Remove old coding-specific columns.
- Give cheap Pi workers persistence, migration, backup/import/restore implementation tasks unless explicitly asked and reviewed strictly.
- Stage, commit, push, reset, checkout, or do destructive git operations without explicit user approval.

How this workflow works:
1. The user picks a Pi/OpenCode model for a bounded worker slice.
2. Codex writes a worker prompt using a recorded prompt style, usually `strict bounded ticket`.
3. The worker edits the repo and returns a report.
4. Codex reviews the actual diff, verifies TypeScript/tests, fixes only small reviewer polish if needed, and rates the worker.
5. Codex updates both CodeLens refactor docs and the Pi HR database/docs.

When creating a worker prompt:
- Say which prompt style you are using and why.
- Include exact repo path, branch, files to read, allowed write scope, do-not-edit list, compatibility boundaries, verification commands, rg/search requirements, and final report shape.
- Record the prompt style later in `model_hr_db.json`.

When reviewing a worker report:
- Inspect the real diff, not only the report.
- Verify scope, compatibility boundaries, tests, and remaining legacy names.
- Rate the model in the HR style: score, strengths, misses, reviewer fixes, recommended use, avoid-for list, prompt improvements, tags, and prompt-style metadata.
- Update `C:\pi-stuff\model_hr_db.json`.
- If the trust summary or prompt rules changed, update `hrworkflow.md`, `HR_DATABASE.md`, `FUTURE_PI_PROMPTING.md`, and `hr_findings_viewer.html`.
- If the CodeLens refactor state changed, update `ONTOLOGY_PROFILE_REFACTOR\implementation_handoff.md` and, if relevant, `05_ANTI_REGRESSION_RULES.md`.

Current model signals:
- Kimi K2.6: strong for strict bounded compatibility-alias/refactor tickets.
- Kimi 2.5: strong for small bounded naming refactors.
- Mimo 2.5 Pro: strong for medium bounded feature-local refactors.
- Qwen 3.6 Plus: strong for compatibility-preserving TypeScript API/payload renames.
- MiniMax M2.7: usable for small UI rename slices, but watch compatibility shims.
- DeepSeek V4 Pro under Pi: strong for focused test-only regression guards; avoid persistence.
- GLM 5.1: strong for architecture guard and anti-regression docs; expensive, so use selectively.

If asked for the next worker prompt, prefer a small bounded slice unless the user explicitly asks for broader work.
```
