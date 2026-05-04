# Future App Integration Notes

The HR database is intentionally shaped like a lightweight ontology so it can later be uploaded to a backend or rendered inside an app UI.

## Current Local Shape

- `model_hr_db.json` is the source of truth.
- Evaluations are records.
- Tags are relationship nodes.
- Strengths, misses, prompt improvements, and reviewer fixes are evidence fields.
- `hr_findings_viewer.html` is a local prototype for clickable navigation.

## Concept-Card Mapping

If this becomes part of a CodeLens-style app, map each model evaluation to a concept/card:

- Card title: `{model} - {task}`
- Type/category: `model_evaluation`
- Summary: `outcome`
- Evidence: `strengths`, `misses`, `reviewerFixes`, `verification`
- Project tags: `projectTags`
- Stack tags: `stackTags`
- General coding tags: `generalCodingTags`
- Risk tags: `riskTags`
- Recommended use: `recommendedUse`
- Avoid for: `avoidFor`
- Prompt improvement hints: `promptImprovements`
- Prompt style: `promptStyle`
- Task type: `taskType`
- Evaluation meaning: `evaluationMeaning`
- Prompt adaptation notes: `promptAdaptationNotes`

## Relationship Ideas

Tags can become clickable graph nodes:

- Model -> performed task
- Model -> has strength
- Model -> has miss
- Model -> useful for tag
- Model -> risky for tag
- Task -> requires tag
- Prompt improvement -> addresses miss
- Prompt style -> changes evaluation meaning
- Prompt adaptation note -> improves future task fit

This makes it possible to ask:

- Which model should handle a bounded Python refactor?
- Which model is best for TypeScript interface propagation?
- Which model should not touch persistence?
- What prompt rule would reduce this model's usual misses?
- What would this model need to do to earn 10/10 on this task type?
- Did the model succeed because it is broadly autonomous, or because the prompt was a strict bounded ticket?
- Which prompt style should I use for this model on the next slice?

## Backend Later

A backend can store the JSON almost directly. Suggested entities:

- `models`
- `evaluations`
- `tasks`
- `tags`
- `findings`
- `verification_runs`
- `prompt_improvements`
- `prompt_styles`
- `prompt_adaptation_notes`

Keep the local JSON format as the import/export format so LLMs can still read it without a database client.
