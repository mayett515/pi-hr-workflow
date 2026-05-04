# Extending the Pi Prompt Builder

This file explains how to add new tools, prompt blocks, and AI providers to the prompt builder. If you install a new Pi extension or want a new behavioral section in your generated prompts, this is where you start.

---

## Quick Reference: What Lives Where

| Piece | File location | What it does |
|-------|-------------|--------------|
| Prompt blocks (sections) | `sections` array in `<script>` | Each object is one card in the sidebar + one XML block in the generated prompt |
| Tool toggles | `tools` array in `<script>` | Each toggle enables/disables one or more sections via the `feature` field |
| AI providers | `getApiConfig()` + `ai` ref + the `<select>` dropdown | Defines how the Enhance button talks to an LLM |
| Template assembly | `buildTemplatePrompt()` | Reads active sections and tools, builds the final Markdown+XML string |
| Harness prompt | `HARNESS_PROMPT` constant | Hidden system prompt sent to the LLM when Enhance is clicked |
| CSS tag colors | `.mini-tag.*` and `.detail-badge.*` in `<style>` | Visual styling for tag badges on blocks and details |
| AGENTS.md | This file | Documentation |

---

## Adding a New Prompt Block

A **prompt block** is one card in the sidebar that describes a behavioral instruction (like "always run tests" or "ask before deleting files"). Each block generates one XML section in the output prompt.

### Step 1: Add the section object

Find the `sections` array inside `<script>`. Add an object like this:

```js
{
  id: 'memory',                // unique id, used everywhere
  title: 'Memory / Context',   // shown on the card and as the heading
  icon: '\u{1F4DA}',           // emoji for the card icon
  tags: ['core'],              // tag badges shown on the card (see tag classes below)
  level: 'recommended',        // required | recommended | optional  (shown as colored badge)
  summary: 'Makes the agent read project memory files before acting.',
  body: 'Full explanation shown in the detail panel when you click the card...',
  example: `<memory_policy>
If the repo has project memory files such as:
- PROJECT.md
- AGENTS.md
- commands.md
Read them before making assumptions.
</memory_policy>`,
  use: 'When to use this block, shown in the detail panel.',
  feature: 'memory'            // links this block to a tool toggle. null = always on
}
```

### Step 2: Link it to a tool toggle (if it should be toggleable)

If the block should turn on/off with a sidebar toggle, set `feature` to a string like `'memory'`. Then add a matching entry to the `tools` array:

```js
{ feature: 'memory', name: 'Project memory', desc: 'Read PROJECT.md, AGENTS.md before acting.', on: true }
```

If the block should **always** appear regardless of toggles (like `identity` or `tests`), set `feature: null`.

### Step 3: Add it to the template output

In `buildTemplatePrompt()`, find where sections are rendered. Most sections are handled automatically by this line:

```js
const tagged = others.filter(s => !special.includes(s.id));
tagged.forEach(s => { parts.push('## ' + s.title + '\n\n' + s.example + '\n'); });
```

If your block needs special ordering (like `tests` which goes near the end), add its `id` to the `special` array and handle it manually.

If your block adds entries to the `<available_tools>` section, add a line to the `toolLines` block:

```js
if (enabledFeatures.value.has('memory')) toolLines.push('- project memory: read PROJECT.md, AGENTS.md, commands.md before assumptions');
```

### Step 4: Add a tag CSS class (if using a new tag name)

If you used a tag string that doesn't already have a CSS class, add one in `<style>`:

```css
.mini-tag.memory { background: rgba(108,140,255,.12); color: var(--accent); }
```

Existing tag classes: `mitsupi`, `pi-hooks`, `core`, `verification`, `git`, `search`, `autonomy`, `quality`, `todo`, `safety`

### Step 5: Verify

Open the HTML in a browser. You should see:
- New card in the sidebar
- Clicking it shows the detail panel
- Toggle (if you added one) enables/disables the section in the generated prompt
- The generated prompt includes your XML block when active

---

## Adding a New AI Provider

The Enhance button supports 5 providers out of the box. To add another:

### Step 1: Add an `<option>` to the provider dropdown

Find the `<select v-model="ai.provider">` in the HTML and add:

```html
<option value="myprovider">My Provider</option>
```

### Step 2: Add default model and base URL

In the `watch` on `ai.value.provider`, add a case:

```js
} else if (p === 'myprovider') {
  ai.value.model = 'my-model-name';
  ai.value.baseUrl = 'https://myprovider.example.com/v1/chat/completions';
}
```

### Step 3: Add the API config

In `getApiConfig()`, add a branch for your provider. If it uses the OpenAI chat completions format (SSE with `data: {"choices":[{"delta":{"content":"..."}}]}`):

```js
} else if (p === 'myprovider') {
  if (!url) url = 'https://myprovider.example.com/v1/chat/completions';
  headers = {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer ' + ai.value.apiKey,
    // any custom headers here
  };
  body = {
    model: ai.value.model,
    messages: [
      { role: 'system', content: systemMsg },
      { role: 'user', content: userMsg }
    ],
    temperature: 0.4,
    stream: true
  };
}
```

The streaming parser already handles OpenAI-format, OpenRouter-format, Ollama JSON, and Anthropic SSE. If your provider uses one of those formats, it will work automatically. If it uses a different format, add a new `else if` branch in the streaming parser section.

### Step 4: Update the placeholder hint

In the `apiKeyPlaceholder` computed, add:

```js
if (p === 'myprovider') return 'my-key-format-...';
```

### Step 5: Add to the key check (if key is required)

The current check `ai.value.provider !== 'ollama'` skips the key requirement only for Ollama. If your provider also doesn't need a key, add it there.

---

## Field Reference

### Section object fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | Unique identifier. Used for selection state, template ordering, and feature linking. |
| `title` | string | yes | Display heading in sidebar card and detail panel. Also used as the `##` heading in generated prompt. |
| `icon` | string | yes | Emoji shown on the sidebar card and detail panel. |
| `tags` | string[] | yes | Badge labels. Must match a CSS class (`.mini-tag.XXX`). |
| `level` | string | yes | One of `required`, `recommended`, `optional`. Shown as colored badge. |
| `summary` | string | yes | One-line description shown under the card title in the sidebar. |
| `body` | string | yes | Full explanation shown in the detail panel when the card is selected. |
| `example` | string | yes | The XML block text that gets inserted into the generated prompt. Must be wrapped in an XML tag. |
| `use` | string | yes | Short "when to use this" text shown in the detail panel. |
| `feature` | string or null | yes | Links this block to a tool toggle. `null` means always on. Must match a `feature` value in the `tools` array. |

### Tool toggle fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `feature` | string | yes | Links to `section.feature`. When toggle is off, any section with this feature is excluded from the prompt. |
| `name` | string | yes | Display name shown next to the toggle switch. |
| `desc` | string | yes | Short description under the toggle name. |
| `on` | boolean | yes | Default state. `true` = enabled. |

### AI provider ref fields

| Field | Default | Description |
|-------|---------|-------------|
| `provider` | `'openai'` | Current provider selection. |
| `apiKey` | `''` | Stored in browser memory only, never persisted. |
| `model` | `'gpt-4o'` | Model string sent to the API. |
| `baseUrl` | `''` | Override endpoint. Empty = use provider default. |

---

## Template Assembly Order

`buildTemplatePrompt()` assembles the prompt in this order:

1. `# Pi Coding Agent System Instructions` heading
2. Opening paragraph
3. `## Identity` (always, if section exists)
4. `## Installed Tools` - dynamic based on toggles
5. All other active sections in array order, except:
6. `## Verification Loop` (tests) goes near the end
7. `## Final Response Format` (always, hardcoded)

If you need a section in a specific position, either:
- Reorder it in the `sections` array
- Add its `id` to the `special` array and place it manually in `buildTemplatePrompt()`

---

## The Harness Prompt

The `HARNESS_PROMPT` constant is the hidden system message sent to the LLM when "AI Enhance" is clicked. It contains two template variables:

- `{PROMPT}` - replaced with the current generated prompt text
- `{TOOLS}` - replaced with a comma-separated list of enabled tool names

To customize what the AI optimizer focuses on, edit `HARNESS_PROMPT`. Keep these rules:
- Tell it to return ONLY refined Markdown (no explanation, no code fences)
- Tell it to preserve XML tag structure
- Tell it not to add capabilities the user hasn't toggled on
- Tell it not to remove core sections (identity, permissions, final response format)

---

## Tag Badges

Each tag string in a section's `tags` array becomes a small colored badge on the card and detail panel. These CSS classes already exist:

| Tag | CSS class | Color |
|-----|-----------|-------|
| mitsupi | `.mini-tag.mitsupi` | Purple (`#a78bfa`) |
| pi-hooks | `.mini-tag.pi-hooks` | Blue (`#60a5fa`) |
| core | `.mini-tag.core` | Pink (`#f0abfc`) |
| verification | `.mini-tag.verification` | Green (`#4ade80`) |
| git | `.mini-tag.git` | Yellow (`#fbbf24`) |
| search | `.mini-tag.search` | Red (`#f87171`) |
| autonomy | `.mini-tag.autonomy` | Accent blue (`#6c8cff`) |
| quality | `.mini-tag.quality` | Green (`#4ade80`) |
| todo | `.mini-tag.todo` | Purple (`#a78bfa`) |
| safety | `.mini-tag.safety` | Red (`#f87171`) |

To add a new tag color, add a CSS rule:

```css
.mini-tag.YOURTAG { background: rgba(R,G,B,.12); color: #HEXCODE; }
```

And for the detail badges:

```css
.detail-badge.YOURTAG { background: rgba(R,G,B,.12); color: #HEXCODE; border: 1px solid rgba(R,G,B,.25); }
```

---

## Adding a New Theme

The app supports 3 themes via CSS custom properties on `body[data-theme]`:

| Theme ID | Name | Feel |
|----------|------|------|
| `midnight` | Midnight | Cold dark blue (default) |
| `sepia` | Warm Sepia | Dark amber, warm, easy on eyes |
| `paper` | Paper | Light background, ink-like text |

### How themes work

All colors are CSS custom properties (variables) defined under `:root` (midnight default). Each theme overrides the same variables in `body[data-theme="THEME_ID"]`.

The Vue app stores the current theme in `theme` ref and persists it to `localStorage` key `pi-prompt-theme`. The `setTheme()` function sets `document.body.dataset.theme` which activates the matching CSS selector.

### To add a new theme

1. Add a `body[data-theme="yourid"]` block in the `<style>` section that overrides every CSS variable from `:root`
2. Add an entry to the `themes` array in `setup()`:
   ```js
   { id: 'yourid', name: 'Your Theme Name', icon: '\u{1F???}' }
   ```
3. The theme switcher in the header renders buttons from this array automatically

### Variables to override

All of these must be set in your theme block for it to look right:

```
--bg, --bg-2, --panel, --panel-2, --panel-3,
--text, --text-2, --muted,
--border, --border-2,
--accent, --accent-2, --accent-glow, --accent-glow-2,
--good, --good-bg, --warn, --warn-bg, --danger, --danger-bg,
--code-bg,
--tag-mitsupi, --tag-pihooks, --tag-core,
--ai-gradient,
--orb-1, --orb-2, --header-bg, --title-gradient
```