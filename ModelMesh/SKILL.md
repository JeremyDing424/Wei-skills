---
name: model-mesh
description: ModelMesh — unified skill for delegating tasks to specialized execution partners. Routes coding tasks (implement, refactor, test, fix bugs, write functions) to Codex, and design tasks (UI mockups, HTML pages, SVG icons, color palettes, typography) to Gemini. Use this skill whenever the user wants to execute a coding or design task through a specialized agent — even if they don't explicitly say "Codex" or "Gemini". Trigger for phrases like "implement this", "write tests for", "design a page", "create an icon", "fix this bug", "refactor this code", or "make a mockup".
---

# ModelMesh — Codex & Gemini

Route tasks to the right execution partner. When in doubt about which to use, consult the Decision Matrix below. For ambiguous tasks, prefer Codex.

## Decision Matrix

| Task Type | Partner |
|-----------|---------|
| Implement features / write code | Codex |
| Refactor, debug, fix bugs | Codex |
| Write / update tests | Codex |
| Code review or analysis | Codex |
| **Modify existing files** (CSS, HTML, JS, any language) | **Codex** |
| **Add CSS styles to an existing project** | **Codex** |
| Create NEW HTML page / mockup from scratch | Gemini |
| Create NEW SVG icons / illustrations | Gemini |
| Color palettes / typography advice | Gemini |
| UI design feedback (no code) | Gemini |

**Critical routing rule**: If the task involves **modifying existing project files** — even if it's "just CSS" or "just styling" — always route to **Codex**. Gemini is only for creating brand-new standalone design artifacts (HTML files, SVGs) from scratch.

**Ambiguous words** — these words alone don't determine the partner. Look at the full intent:
- `CSS`, `style`, `color`, `button appearance` → **Codex** if modifying an existing file; **Gemini** if creating a new design artifact
- `form`, `button`, `page`, `component` → **Codex** if about logic/validation/handlers or editing existing code
- `form`, `button`, `page`, `component` → **Gemini** if creating a brand-new standalone mockup

---

## Script Location

```
~/.claude/skills/ModelMesh/scripts/execute.sh
```

Verify installation:
```bash
~/.claude/skills/ModelMesh/scripts/execute.sh --check
```

---

## Part 1: Codex — Code Execution

### Critical Rules
- Only interact with Codex through the bundled `execute.sh` script. Never call `codex` CLI directly.
- Run the script once per task. On success (exit 0), read the output file. Do NOT retry.
- Keep the task prompt focused (~500 words max). Describe WHAT, not HOW.
- **Do NOT use `--file`** — it adds the full file size to the context threshold and causes failures on large files. Instead, include the absolute file path directly in the prompt text. Codex will locate and read the file itself.

### Usage

```bash
# Minimal
execute.sh "Your request"

# With file context — include absolute path in the prompt, Codex will read it
execute.sh "Refactor /project/src/components/UserList.tsx and /project/src/components/UserDetail.tsx to use the new API"

# Multi-turn (continue previous session)
execute.sh "Also add retry logic" --session <session_id>
```

### Output

```
session_id=<thread_id>
output_path=<path to markdown file>
```

Read `output_path` for Codex's response. Save `session_id` for follow-up calls.

### Codex Options

| Flag | Description |
|------|-------------|
| `--partner <codex\|gemini>` | Force specific partner (use when auto-detection mis-routes) |
| `--workspace <path>` | Target workspace (default: current dir) |
| `--session <id>` | Resume a previous conversation |
| `--reasoning <level>` | `low` / `medium` / `high` (default: medium) |
| `--read-only` | Analysis only, no file changes |
| `--model <name>` | Override model |

Use `--reasoning high` for debugging, complex refactoring, or code review.

### Examples

```bash
# Batch refactoring (include path in prompt, no --file needed)
execute.sh "Convert all class components in /project/src/components to functional components with hooks"

# Test writing
execute.sh "Write unit tests for UserService in /project/src/services/UserService.ts covering all public methods and error cases"

# Bug fix
execute.sh "Fix the memory leak in /project/src/websocket/handler.ts — listeners aren't cleaned up on disconnect." \
  --reasoning high
```

---

## Part 2: Gemini Designer — Design Execution

### Critical Rules
- Only interact with Gemini through `execute.sh`. Never call the API directly.
- Describe **what** the design is for, not how it should look (unless the user specified).
- Gemini API key must be set: `GEMINI_API_KEY` env var, `.env.local`, or `~/.config/gemini-designer/api_key`.

### Usage

```bash
# HTML page
execute.sh "Design a modern landing page for a SaaS product called FlowSync" --html

# SVG icon
execute.sh "Create a minimal settings gear icon, 24x24, stroke style" --svg

# Design advice (text)
execute.sh "Suggest a color palette and typography for a developer blog"

# Custom output path (type auto-inferred from extension)
execute.sh "Design a pricing card" -o ./designs/pricing-card.html
```

### Output

```
output_path=<path to output file>
```

### Gemini Options

| Flag | Description |
|------|-------------|
| `--html` | Output self-contained HTML file |
| `--svg` | Output SVG code |
| `-o / --output <path>` | Custom output path |
| `--model <name>` | Override model |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `GEMINI_API_KEY` | API key (required) |
| `GOOGLE_GEMINI_BASE_URL` | API base URL (optional) |
| `GEMINI_MODEL` | Model name (optional) |

### Examples

```bash
# Landing page
execute.sh "Landing page for a project management tool. Hero, features, pricing, CTA." --html

# Icon set
execute.sh "5 SVG icons (16x16): home, settings, users, analytics, notifications. Minimalist line style." --svg

# Design system
execute.sh "B2B SaaS dashboard — suggest color palette, typography, and spacing scale."
```

---

## Unified Workflow

1. **Clarify** — Is it code or design? Check the Decision Matrix. When in doubt, use `--partner codex`.
2. **Prepare** — For Codex: include absolute file paths in the prompt text. For Gemini: write a focused description.
3. **Execute** — Run `execute.sh` with appropriate flags. If auto-detection routes to the wrong partner, override with `--partner codex` or `--partner gemini`.
4. **Review** — Read the output file. For Codex: check code changes. For Gemini: open the design file.
5. **Iterate** — Use `--session` (Codex) or re-run with refinements (Gemini).

---

## Troubleshooting

**`Command not found`**
```bash
chmod +x ~/.claude/skills/ModelMesh/scripts/execute.sh
```

**`Partner script not found`**
Run `execute.sh --check` to see which partner scripts are missing. Ensure ModelMesh is installed in `~/.claude/skills/model-mesh/`.

**Gemini API errors**
- Verify `GEMINI_API_KEY` is set correctly
- Check `GOOGLE_GEMINI_BASE_URL` points to the right endpoint (`/v1/chat/completions`)
