# Pi Prompt Builder - Harness Documentation

## What This Is

`pi_prompt_template_builder.html` is a single-file interactive app that builds system prompts for the Pi local coding agent. It uses Vue 3 (CDN) and runs entirely in the browser with no build step.

## How It Works

### Template Engine

The app defines 10 prompt blocks (identity, todos, permissions, checkpoint, lsp, tests, review, commit, web, loop). Each block has:

- An XML tag section (`<todo_protocol>`, `<permission_policy>`, etc.)
- Metadata: level (required/recommended/optional), tags, which feature it depends on
- A description, usage guide, and example

When toggles change, the app rebuilds the full Markdown+XML prompt from the active blocks and writes it to the textarea.

### AI Enhancement (The Harness)

The "AI Enhance" button sends the current prompt to an LLM with a hidden **harness prompt** that instructs the model to:

1. Tighten language and remove redundancy
2. Add missing edge cases local coding agents commonly need
3. Improve XML tag boundaries
4. Reorder sections for better model comprehension
5. Add a `<memory_policy>` section for project memory files
6. Strengthen behavioral constraints (hallucinating tool runs, jumping to conclusions, mixing unrelated commits)

The harness is stored in the `HARNESS_PROMPT` constant in the `<script>` section. It includes the user's current prompt and their enabled tools list.

### Supported API Providers

| Provider | Default Base URL | Notes |
|----------|-----------------|-------|
| OpenAI / Compatible | `https://api.openai.com/v1/chat/completions` | Needs API key. Works with any OpenAI-compatible endpoint (LM Studio, etc.) |
| OpenRouter | `https://openrouter.ai/api/v1/chat/completions` | Needs API key (`sk-or-...`). Sends `HTTP-Referer` and `X-Title` headers. Access to 200+ models. |
| OpenCode | `https://api.opencode.ai/v1/chat/completions` | Needs API key. OpenAI-compatible endpoint. Change base URL if self-hosting. |
| Ollama (local) | `http://localhost:11434/api/chat` | No API key needed. No CORS issues since it's local. |
| Claude (Anthropic) | `https://api.anthropic.com/v1/messages` | Needs API key. May have CORS issues from browser - uses the `anthropic-dangerous-direct-browser-access` header. |

### Streaming

All providers use streaming (SSE or NDJSON). The modal shows text arriving character by character with a cursor animation.

### Editable Prompt

The textarea is editable. If the user manually edits it, a "edited manually" indicator appears with a "re-sync from template" link. Changing toggles automatically re-syncs (clearing manual edits).

## The Harness Prompt

The exact harness prompt is defined in `HARNESS_PROMPT` in the HTML file. Key instructions:

- Preserve XML tag structure
- Don't add capabilities the user hasn't toggled on
- Don't remove core sections (identity, permission policy, final response format)
- Return ONLY refined Markdown, no explanation, no code fences
- `Markdown = structure`, `XML = behavioral boundaries`

## Customizing the Harness

To change what the AI optimizer focuses on, edit the `HARNESS_PROMPT` constant near the top of the `<script>` block. Variables:

- `{PROMPT}` - replaced with the current generated prompt
- `{TOOLS}` - replaced with the list of enabled tool names

## Architecture

```
User toggles/blocks
        |
        v
  Template Engine (buildTemplatePrompt)
        |
        v
  editablePrompt (ref)
        |            |
        v            v
  Textarea      AI Enhance button
                     |
                     v
              Harness + prompt --> LLM API
                     |
                     v
              Streaming result --> Modal (side-by-side diff)
                     |
              Apply --> editablePrompt
              Copy  --> clipboard
              Discard
```

## Files

- `pi_prompt_template_builder.html` - The entire app (single file, Vue 3 CDN, no build step)
- `AGENTS.md` - This file
- `EXTENDING.md` - How to add new tools, prompt blocks, and AI providers to the prompt builder

## Extending

If you need to add a new Pi tool, prompt block, AI provider, or tag badge to the prompt builder, read **EXTENDING.md** first. It has step-by-step instructions, field references, and the template assembly order. Any AI agent (including Pi itself) can read that file and know exactly what to edit in the HTML.