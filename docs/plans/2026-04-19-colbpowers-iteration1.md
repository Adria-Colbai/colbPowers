# colbPowers Iteration 1 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build iteration 1 of colbPowers — a spec-first Claude Code plugin that seeds from superpowers, prunes cross-harness baggage, adds a colbPowers bootstrap + 6 native skills + 6 slash commands + 1 reviewer subagent, and passes the smoke tests defined in the spec §10.

**Architecture:** colbPowers is a Claude Code plugin with the standard layout (`.claude-plugin/`, `skills/`, `commands/`, `hooks/`, `agents/`, `templates/`). The orchestrator is the main conversation, bootstrapped by a `SessionStart` hook that injects the `using-colbpowers` skill. Slash commands map 1:1 to native skills; auto-activating skills handle natural-language invocation. Subagents are dispatched only for Developer-execution and Reviewer-audit phases.

**Tech Stack:** Bash (hook script), Markdown + YAML frontmatter (skills, commands, agents), JSON (plugin manifest, active context). No runtime dependencies, no build step.

---

## Reference Spec

Design: `docs/specs/2026-04-19-colbpowers-plugin-design.md`. Every artifact, phase, and skill listed in §4–§8 of the spec has a corresponding task below.

## File Structure

All paths below are relative to `/home/adria/COLBAI/TFG/colbPowers/`.

**Created (new to colbPowers):**
- `.gitignore`
- `.claude-plugin/plugin.json` — manifest (replaces superpowers')
- `.claude-plugin/marketplace.json` — local dev marketplace
- `CLAUDE.md` — contributor guidelines for colbPowers
- `README.md` — usage docs
- `LICENSE` — MIT
- `NOTICE` — superpowers attribution
- `hooks/session-start` — rewritten for colbPowers + Claude-Code-only
- `skills/using-colbpowers/SKILL.md` — bootstrap
- `skills/init-constitution/SKILL.md` — Architect Workflow A
- `skills/update-features/SKILL.md` — Architect Workflow B
- `skills/update-constitution/SKILL.md` — Architect Workflow C
- `skills/create-feature/SKILL.md` — scaffolds feature dir
- `skills/audit-project/SKILL.md` — Reviewer wrapper
- `agents/colb-reviewer.md` — Reviewer subagent
- `commands/colb:init.md`, `colb:roadmap.md`, `colb:standards.md`, `colb:plan.md`, `colb:execute.md`, `colb:review.md`
- `templates/constitution_template.md`, `features_template.md`, `design_template.md`, `tasks_template.md`, `active_context_template.json`
- `tests/smoke-test.sh` — structural validation script

**Modified (from seed):**
- `hooks/hooks.json` — verify matches target structure

**Deleted (from seed):**
- `.git/`, `.opencode/`, `.codex/`, `.cursor-plugin/`, `.github/`, `.gitattributes`, `.version-bump.json`
- `gemini-extension.json`, `package.json`
- `scripts/`, `tests/` (seed tests — we write our own), `docs/` (superpowers docs — keep only our own `docs/specs/` and `docs/plans/`)
- `CHANGELOG.md`, `RELEASE-NOTES.md`, `AGENTS.md`, `GEMINI.md`, `CODE_OF_CONDUCT.md`
- `skills/using-superpowers/`, `skills/writing-skills/`
- `commands/brainstorm.md`, `commands/execute-plan.md`, `commands/write-plan.md`

---

## Phase A — Repo Foundation

### Task 1: Seed the repo from superpowers cache

**Files:**
- Create: `/home/adria/COLBAI/TFG/colbPowers/*` (full copy of superpowers cache, minus existing `docs/` which already has our spec)

- [ ] **Step 1: Copy superpowers cache into colbPowers, preserving existing docs/**

```bash
cp -rn /home/adria/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.7/. /home/adria/COLBAI/TFG/colbPowers/
```

Note: `-n` prevents overwriting our existing `docs/specs/2026-04-19-colbpowers-plugin-design.md`. Because the superpowers `docs/` has different subdirectories (`docs/superpowers/`, `docs/plans/`, `docs/windows/`, `docs/testing.md`, `docs/README.codex.md`, `docs/README.opencode.md`), the copy merges cleanly.

- [ ] **Step 2: Verify the copy landed**

```bash
ls /home/adria/COLBAI/TFG/colbPowers/
```

Expected output includes: `.claude-plugin`, `.opencode`, `agents`, `commands`, `docs`, `hooks`, `skills`, `templates-does-not-exist-yet`, `LICENSE`, `README.md`, plus the pre-existing `docs/specs/` from our earlier work.

- [ ] **Step 3: Confirm design doc preserved**

```bash
ls /home/adria/COLBAI/TFG/colbPowers/docs/specs/
```

Expected: `2026-04-19-colbpowers-plugin-design.md`.

- [ ] **Step 4: No commit yet — git init happens in Task 2 after prune**

Rationale: we prune first so the initial commit is clean and small.

---

### Task 2: Prune unused files and initialize git

**Files:**
- Delete: `.opencode/`, `.codex/`, `.cursor-plugin/`, `.github/`, `.gitattributes`, `.version-bump.json`, `gemini-extension.json`, `package.json`, `scripts/`, `tests/`, `CHANGELOG.md`, `RELEASE-NOTES.md`, `AGENTS.md`, `GEMINI.md`, `CODE_OF_CONDUCT.md`, `skills/using-superpowers/`, `skills/writing-skills/`, `commands/brainstorm.md`, `commands/execute-plan.md`, `commands/write-plan.md`, `docs/README.codex.md`, `docs/README.opencode.md`, `docs/windows/`, `docs/superpowers/`, `docs/testing.md`
- Create: `.gitignore`

- [ ] **Step 1: Delete cross-harness directories**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
rm -rf .opencode .codex .cursor-plugin .github
```

- [ ] **Step 2: Delete superpowers-specific config and scripts**

```bash
rm -f .gitattributes .version-bump.json gemini-extension.json package.json
rm -rf scripts tests
```

- [ ] **Step 3: Delete superpowers-specific top-level docs**

```bash
rm -f CHANGELOG.md RELEASE-NOTES.md AGENTS.md GEMINI.md CODE_OF_CONDUCT.md
```

- [ ] **Step 4: Delete superpowers-specific docs subtree (keep our own)**

```bash
rm -f docs/README.codex.md docs/README.opencode.md docs/testing.md
rm -rf docs/windows docs/superpowers
```

Verify:
```bash
ls docs/
```
Expected: `plans`, `specs` (only our directories remain).

- [ ] **Step 5: Delete skills and commands we replace**

```bash
rm -rf skills/using-superpowers skills/writing-skills
rm -f commands/brainstorm.md commands/execute-plan.md commands/write-plan.md
```

- [ ] **Step 6: Remove inherited .git**

```bash
rm -rf .git
```

- [ ] **Step 7: Write .gitignore**

Create `/home/adria/COLBAI/TFG/colbPowers/.gitignore`:

```
.env
.env.*
.venv/
__pycache__/
node_modules/
*.log
.DS_Store
.claude-local/
```

- [ ] **Step 8: Initialize git fresh and stage everything**

```bash
git init -q
git add .
git status --short | head -20
```

Expected: all files shown as `A` (added).

- [ ] **Step 9: Commit the seed**

```bash
git commit -q -m "Seed colbPowers from superpowers 5.0.7 and prune cross-harness files"
git log --oneline
```

Expected: one commit on `main` (or `master`) branch.

---

### Task 3: Rebrand the plugin manifest

**Files:**
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Rewrite plugin.json**

Overwrite `/home/adria/COLBAI/TFG/colbPowers/.claude-plugin/plugin.json` with:

```json
{
  "name": "colbPowers",
  "description": "Spec-first Claude Code plugin: constitution-driven governance, feature-scoped design docs, TDD subagent execution, and independent audit review.",
  "version": "0.1.0",
  "author": {
    "name": "Adrià Cebrián",
    "email": "adria.cebrian@colb.ai"
  },
  "license": "MIT",
  "keywords": [
    "spec-driven-development",
    "constitution",
    "tdd",
    "subagents",
    "governance",
    "workflow"
  ]
}
```

- [ ] **Step 2: Rewrite marketplace.json**

Overwrite `/home/adria/COLBAI/TFG/colbPowers/.claude-plugin/marketplace.json` with:

```json
{
  "name": "colbPowers-local",
  "description": "Local development marketplace for colbPowers",
  "owner": {
    "name": "Adrià Cebrián",
    "email": "adria.cebrian@colb.ai"
  },
  "plugins": [
    {
      "name": "colbPowers",
      "description": "Spec-first Claude Code plugin combining constitution-driven governance with superpowers-style TDD execution.",
      "version": "0.1.0",
      "source": "./",
      "author": {
        "name": "Adrià Cebrián",
        "email": "adria.cebrian@colb.ai"
      }
    }
  ]
}
```

- [ ] **Step 3: Validate JSON**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
python3 -c "import json; json.load(open('.claude-plugin/plugin.json')); json.load(open('.claude-plugin/marketplace.json')); print('OK')"
```

Expected: `OK`.

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/
git commit -q -m "Rebrand plugin manifest for colbPowers"
```

---

### Task 4: Write project-level documentation

**Files:**
- Modify: `README.md`
- Modify: `LICENSE`
- Create: `CLAUDE.md`
- Create: `NOTICE`

- [ ] **Step 1: Write LICENSE**

Overwrite `/home/adria/COLBAI/TFG/colbPowers/LICENSE`:

```
MIT License

Copyright (c) 2026 Adrià Cebrián

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Write NOTICE**

Create `/home/adria/COLBAI/TFG/colbPowers/NOTICE`:

```
colbPowers

A Claude Code plugin for spec-first development, combining colbSpec's
constitution-driven governance with superpowers' TDD execution discipline.

---

This plugin seeds and adapts work from:

  Superpowers
  https://github.com/obra/superpowers
  Copyright (c) Jesse Vincent
  Licensed under the MIT License

The following skills in skills/ are copies (verbatim or adapted) of skills
originally published in the Superpowers plugin:

  brainstorming
  writing-plans
  subagent-driven-development
  executing-plans
  test-driven-development
  verification-before-completion
  systematic-debugging
  using-git-worktrees
  finishing-a-development-branch
  requesting-code-review
  receiving-code-review
  dispatching-parallel-agents

The hooks/session-start script is adapted from the Superpowers hook of the
same name.

All adaptations remain under the MIT License. See LICENSE for terms.
```

- [ ] **Step 3: Write README.md**

Overwrite `/home/adria/COLBAI/TFG/colbPowers/README.md`:

```markdown
# colbPowers

A Claude Code plugin for spec-first development. Combines constitution-driven governance with TDD execution discipline.

## What it does

colbPowers structures each project around four artifact layers:

- **`constitution.md`** — project-wide law (vision, governance, tech stack).
- **`features.md`** — roadmap of feature names, intents, and dependencies.
- **`features/<feature>/design.md`** — per-feature design doc (from brainstorming).
- **`features/<feature>/tasks.md`** — per-feature implementation plan (bite-sized TDD tasks).

Three conceptual phases drive the workflow:

| Phase | Mechanism | Artifacts |
|---|---|---|
| **Architect** | In-conversation skills | `constitution.md`, `features.md` |
| **Developer — planning** | In-conversation (brainstorming + writing-plans) | `design.md`, `tasks.md` |
| **Developer — execution** | Dispatched subagents (one per task) | source files, tests |
| **Reviewer** | Dispatched audit subagent | pass/warn/fail report |

The orchestrator is your main Claude Code conversation — it reads `.specs/memory/` at session start and proactively suggests the next phase action.

## Installation (local dev)

```
/plugin install /home/adria/COLBAI/TFG/colbPowers
```

Or register the local marketplace:

```
/plugin marketplace add /home/adria/COLBAI/TFG/colbPowers
/plugin install colbPowers@colbPowers-local
```

## Slash commands

| Command | Phase |
|---|---|
| `/colb:init` | Architect — create constitution + initial features roadmap |
| `/colb:roadmap` | Architect — add or update features in the roadmap |
| `/colb:standards` | Architect — update constitution (with conflict detection) |
| `/colb:plan <feature>` | Developer planning — brainstorm design + write tasks |
| `/colb:execute <feature>` | Developer execution — dispatch task subagents |
| `/colb:review [<feature>]` | Reviewer — dispatch audit subagent |

## Status

Iteration 1. Native skills functional; bundled superpowers skills behave as-copied (artifact-path adaptation arrives in iteration 2).

## License

MIT. Skills adapted from [obra/superpowers](https://github.com/obra/superpowers) (MIT). See `NOTICE` for attribution.
```

- [ ] **Step 4: Write CLAUDE.md**

Create `/home/adria/COLBAI/TFG/colbPowers/CLAUDE.md`:

```markdown
# colbPowers — Contributor Guidelines

## What this plugin is

colbPowers is a spec-first Claude Code plugin. It combines the artifact discipline of colbSpec (constitution / features / design / tasks) with execution skills adapted from superpowers (brainstorming, TDD, subagent-driven development, review).

The plugin is general-purpose — installable on any project. Project content lives in `.specs/memory/` in the consuming project, never inside this repo.

## Design source of truth

- `docs/specs/` — design documents for colbPowers itself. Read the latest before proposing changes.
- `docs/plans/` — implementation plans. Tasks here are tracked via checkbox syntax.

## When modifying skills

- The six native skills (`using-colbpowers`, `init-constitution`, `update-features`, `update-constitution`, `create-feature`, `audit-project`) are colbPowers-specific. Treat them as code: changes need justification, and behavior tests should accompany logic changes.
- The bundled superpowers skills are adapted copies. If you pull improvements from upstream, note the source commit in the commit message. Do not reintroduce superpowers-specific artifact paths that conflict with our `.specs/memory/` convention.

## Commits

- Keep commits small and scoped. One task per commit where practical.
- Commit messages start with the phase (e.g., `architect:`, `developer:`, `reviewer:`, `plugin:`, `docs:`).

## Testing

- `tests/smoke-test.sh` validates structural correctness (plugin manifest, skill frontmatter, hook output).
- Run it before every commit that touches plugin metadata, hooks, or skill frontmatter.

## What does not belong here

- GitHub-specific integration (issues sync, remote spec sync). The plugin is zero-dependency; such features belong in a companion plugin.
- Cross-harness adapters for iteration 1. OpenCode / Codex / Cursor / Gemini support is deferred; reintroduce under `.opencode/`, `.codex/`, etc., only when explicitly planned.
- Project-specific content. Nothing in this repo should assume a particular consumer project's tech stack, team, or workflow.
```

- [ ] **Step 5: Commit documentation**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add README.md LICENSE NOTICE CLAUDE.md
git commit -q -m "docs: rewrite README, LICENSE, NOTICE, CLAUDE.md for colbPowers"
```

---

## Phase B — Infrastructure

### Task 5: Create templates directory

**Files:**
- Create: `templates/constitution_template.md`
- Create: `templates/features_template.md`
- Create: `templates/design_template.md`
- Create: `templates/tasks_template.md`
- Create: `templates/active_context_template.json`

- [ ] **Step 1: Create templates directory**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
mkdir -p templates
```

- [ ] **Step 2: Copy constitution_template.md from ProjectAssistant**

Write `/home/adria/COLBAI/TFG/colbPowers/templates/constitution_template.md`:

```markdown
# Project Constitution Template

## 1. The Whole Picture
> [Insert a high-level description of the project vision, goals, and scope here. What is the MVP? What problem does it solve?]

## 2. Core Rules (Governance)
- **Mandatory Testing**: Every feature MUST include unit tests.
- **Environment Isolation**: Usage of specific runners (e.g., `uv run`) or containers.
- **Clean Code Principle**: [Define core coding principles, e.g., separation of concerns, single responsibility].

## 3. Tech Stack & Coding Standards
### Technology
- **Language**: [e.g., Python 3.11+]
- **Frameworks**: [e.g., Streamlit, FastAPI, Django]
- **Tooling**: [e.g., UV, Poetry, Docker]

### Style Guidelines
- **Naming**: [e.g., Snake_case for Python, camelCase for JS]
- **Type Hinting**: [Requirements for type safety]
- **Project Structure**: [Rules about folder hierarchy and imports]
```

- [ ] **Step 3: Write features_template.md**

Write `/home/adria/COLBAI/TFG/colbPowers/templates/features_template.md`:

```markdown
# Project Features Template

## 1. [Feature Name]
**Description**
[A brief description of the feature and its purpose.]

**Functional Requirements**
- **[Requirement 1]**: [Detail what the system must do].
- **[Requirement 2]**: [Detail inputs, outputs, or behaviors].

**Technical Constraints**
- [Constraint 1, e.g., specific libraries to use or avoid].
- **Dependencies**: [List other features or modules this feature depends on].

## 2. [Next Feature Name]
...
```

- [ ] **Step 4: Write design_template.md (new)**

Write `/home/adria/COLBAI/TFG/colbPowers/templates/design_template.md`:

```markdown
# Design: [Feature Name]

**Status:** draft | approved | superseded
**Author:** [Name]
**Date:** YYYY-MM-DD
**Linked feature:** `features.md` entry "[Feature Name]"

## 1. Purpose

[One paragraph. What does this feature accomplish? Why does it matter? What is explicitly out of scope?]

## 2. Non-Goals

- [Thing this design deliberately does not do.]

## 3. Constraints from the Constitution

[Cite specific rules from `constitution.md` that shape this design. Example: "Constitution §2.1 requires TDD — every public function needs a failing test before implementation."]

## 4. Architecture

[2–4 paragraphs. The shape of the solution. Key components, their responsibilities, how they interact. Prefer a small number of well-bounded units over a large number of tangled ones.]

### 4.1 Components

| Component | Responsibility | Depends on |
|---|---|---|
| [Name] | [One sentence] | [Other components or external libs] |

### 4.2 Data flow

[How data moves through the components. A short narrative or a diagram.]

## 5. Error Handling

[How failures surface. What is retried, what is logged, what propagates.]

## 6. Testing Strategy

[What unit tests cover. What integration points need testing. Where fakes or mocks apply.]

## 7. Open Questions

- [Unknowns that do not block the plan but should be resolved before or during execution.]

## 8. Success Criteria

- [ ] [Observable behavior 1 — what a user or another system can do once this feature ships.]
- [ ] [Observable behavior 2]
```

- [ ] **Step 5: Copy tasks_template.md from ProjectAssistant**

Write `/home/adria/COLBAI/TFG/colbPowers/templates/tasks_template.md`:

```markdown
# Implementation Plan: [Feature Name]

- [ ] **Scaffold**: Initial file setup.
    -   Create `src/features/[feature_name]/models.py`.
    -   Create `src/features/[feature_name]/[other_files].py`.

- [ ] **Validation**: Data integrity and business rules.
    -   [Rule 1]
    -   [Rule 2]

- [ ] **Core Logic**: Main implementation tasks.
    -   Implement [Function 1].
    -   Implement [Function 2].

- [ ] **Interfaces**: CLI, API, or UI integration.
    -   [Interface Task 1]

- [ ] **Testing**: Unit and integration tests.
    -   Create `tests/features/[feature_name]/test_models.py`.
    -   Create `tests/features/[feature_name]/test_core.py`.
```

- [ ] **Step 6: Copy active_context_template.json**

Write `/home/adria/COLBAI/TFG/colbPowers/templates/active_context_template.json`:

```json
{
  "active_feature_name": "<Name>",
  "feature_folder_path": "<AgreedPath>",
  "status": "planning"
}
```

- [ ] **Step 7: Commit templates**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add templates/
git commit -q -m "plugin: add project-scaffolding templates (constitution, features, design, tasks, active_context)"
```

---

### Task 6: Rewrite hooks/session-start for colbPowers

**Files:**
- Modify: `hooks/session-start`
- Verify: `hooks/hooks.json` (should already match target)

- [ ] **Step 1: Inspect existing hooks.json and confirm target form**

```bash
cat /home/adria/COLBAI/TFG/colbPowers/hooks/hooks.json
```

Expected content:
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

We need to simplify this: superpowers uses a `run-hook.cmd` wrapper for Windows support. For Claude-Code-only Linux/macOS, invoke `session-start` directly.

- [ ] **Step 2: Rewrite hooks.json**

Overwrite `/home/adria/COLBAI/TFG/colbPowers/hooks/hooks.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/session-start\"",
            "async": false
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 3: Delete run-hook.cmd**

```bash
rm -f /home/adria/COLBAI/TFG/colbPowers/hooks/run-hook.cmd
```

- [ ] **Step 4: Rewrite session-start script**

Overwrite `/home/adria/COLBAI/TFG/colbPowers/hooks/session-start`:

```bash
#!/usr/bin/env bash
# SessionStart hook for colbPowers plugin
# Injects the using-colbpowers bootstrap skill into the main conversation.

set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"

bootstrap_skill="${PLUGIN_ROOT}/skills/using-colbpowers/SKILL.md"
if [ ! -f "$bootstrap_skill" ]; then
  echo '{"hookSpecificOutput":{"hookEventName":"SessionStart","additionalContext":""}}'
  exit 0
fi

bootstrap_content=$(cat "$bootstrap_skill")

escape_for_json() {
  local s="$1"
  s="${s//\\/\\\\}"
  s="${s//\"/\\\"}"
  s="${s//$'\n'/\\n}"
  s="${s//$'\r'/\\r}"
  s="${s//$'\t'/\\t}"
  printf '%s' "$s"
}

bootstrap_escaped=$(escape_for_json "$bootstrap_content")

session_context="<EXTREMELY_IMPORTANT>\nYou have colbPowers.\n\n**Below is the full content of your 'colbPowers:using-colbpowers' skill — your introduction to this plugin's spec-first workflow. For all other skills, use the 'Skill' tool.**\n\n${bootstrap_escaped}\n</EXTREMELY_IMPORTANT>"

printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$session_context"

exit 0
```

- [ ] **Step 5: Make it executable**

```bash
chmod +x /home/adria/COLBAI/TFG/colbPowers/hooks/session-start
ls -l /home/adria/COLBAI/TFG/colbPowers/hooks/session-start
```

Expected: first character `-rwxr-xr-x` or similar with `x` bits set.

- [ ] **Step 6: Smoke-test the hook output (before the skill body exists, it will produce an empty additionalContext — that is expected here)**

```bash
CLAUDE_PLUGIN_ROOT=/home/adria/COLBAI/TFG/colbPowers /home/adria/COLBAI/TFG/colbPowers/hooks/session-start | python3 -c "import json, sys; d = json.load(sys.stdin); print('OK' if 'hookSpecificOutput' in d else 'FAIL')"
```

Expected: `OK`. (The `additionalContext` will be empty because we have not yet created `skills/using-colbpowers/SKILL.md` — the script handles that gracefully via the existence check.)

- [ ] **Step 7: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add hooks/
git commit -q -m "plugin: rewrite session-start hook for colbPowers (Claude-Code-only, emits using-colbpowers)"
```

---

## Phase C — Skills

### Task 7: Write the `using-colbpowers` bootstrap skill

**Files:**
- Create: `skills/using-colbpowers/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p /home/adria/COLBAI/TFG/colbPowers/skills/using-colbpowers
```

- [ ] **Step 2: Write SKILL.md**

Create `/home/adria/COLBAI/TFG/colbPowers/skills/using-colbpowers/SKILL.md`:

```markdown
---
name: using-colbpowers
description: Bootstrap skill for colbPowers. Loaded automatically at session start. Establishes the spec-first orchestration behavior — detect project state from .specs/memory/, proactively suggest the next phase action, route natural-language intent into the correct colbPowers skill, and request confirmation before dispatching subagents.
---

# Using colbPowers

You are operating as the **colbPowers orchestrator**: the main Claude Code conversation in a project that has colbPowers installed. You coordinate a spec-first workflow across four artifact layers — constitution, features, design, tasks — and three conceptual phases: Architect, Developer, Reviewer.

This skill is already loaded. Do not re-invoke it.

## The Four Artifacts

All project state lives in `.specs/memory/` in the current working directory:

| File | Layer | Meaning |
|---|---|---|
| `constitution.md` | Project-wide | The law: vision, governance, tech stack, coding standards. |
| `features.md` | Project-wide | The roadmap: feature names, intents, dependencies. |
| `features/<feature>/design.md` | Per-feature | The design doc from brainstorming. |
| `features/<feature>/tasks.md` | Per-feature | The implementation plan (bite-sized TDD tasks). |
| `active_context.json` | Project-wide | Which feature is in progress and what phase it is in. |

## On Every Session Start

When this bootstrap loads, read `.specs/memory/` to determine the project state, then greet the user with the correct proactive suggestion from the table below.

| Detected state | Say to the user |
|---|---|
| `.specs/memory/` does not exist, or `constitution.md` is missing | "No constitution yet — this project hasn't been initialized with colbPowers. Run `/colb:init` when ready, or tell me what you're building and I'll draft one." |
| `constitution.md` exists, `features.md` missing or empty | "Constitution is in place. The feature roadmap is empty — ready to define what we're building? Run `/colb:roadmap` or describe the features you want." |
| `active_context.json` has `status: planning` and `design.md` is missing | "Feature `<name>` is in planning. Want to brainstorm its design? Run `/colb:plan <name>`." |
| `design.md` exists, `tasks.md` missing | "Design for `<name>` is written. Ready to generate the task plan — run `/colb:plan <name>` to continue, or let me know if you want to revise the design first." |
| `tasks.md` exists, `src/features/<name>/` does not | "Plan for `<name>` is ready. Run `/colb:execute <name>` to dispatch subagents, or let me know if you want to revise the plan first." |
| Feature has source code and all tasks check off as complete in `tasks.md` | "`<name>` looks done. Run `/colb:review <name>` to audit against the constitution, or merge/ship if you're already confident." |

If the state does not match any row, do not offer a phase suggestion; respond normally.

## Phase Mechanisms

| Phase | Mechanism | Why |
|---|---|---|
| **Architect** (constitution, features, standards) | In-conversation skills | Chat nuance matters for governance writing; isolation brings no benefit. |
| **Developer — planning** (brainstorm, write tasks) | In-conversation skills | Chat history produces coherent design + plan docs. |
| **Developer — execution** (per-task work) | Dispatched subagents | Each task gets its own TDD loop and isolated context. |
| **Reviewer** (audit) | One dispatched subagent | Fresh context = unbiased audit. |

## Subagent Dispatch Rule

**Always ask before dispatching a subagent.** Subagent work is high-cost and opaque from the user's view. Present a one-sentence summary of what the subagent will do, then wait for a clear "yes, proceed."

This rule applies to:
- `/colb:execute <feature>` — dispatches one subagent per task in `tasks.md`.
- `/colb:review <feature>` — dispatches the `colb-reviewer` subagent.

In-conversation skills (`init-constitution`, `update-features`, `update-constitution`, `create-feature`, `brainstorming`, `writing-plans`) do not require this extra confirmation step beyond their own skills' behavior.

## Slash Commands

| Command | Skill(s) invoked |
|---|---|
| `/colb:init` | `init-constitution` |
| `/colb:roadmap` | `update-features` |
| `/colb:standards` | `update-constitution` |
| `/colb:plan <feature>` | `create-feature` → `brainstorming` → `writing-plans` |
| `/colb:execute <feature>` | `subagent-driven-development` |
| `/colb:review [<feature>]` | `audit-project` |

Natural-language requests map to the same skills via the auto-activating skill system. If a user says "add a feature for user auth," invoke `update-features` directly — do not require them to type the slash command.

## Skill Priority

When multiple skills could apply:

1. colbPowers-native skills (`init-constitution`, etc.) take precedence for phase-boundary actions.
2. Bundled superpowers skills (`brainstorming`, `writing-plans`, `test-driven-development`, etc.) handle execution discipline inside a phase.
3. General-purpose skills handle everything else.

## Red Flags

If the user asks to:
- Start implementing before the constitution exists → **stop**, suggest `/colb:init` first.
- Skip tests → check the constitution. If TDD is mandated, flag the conflict and ask before proceeding.
- Modify `.specs/memory/` content directly → fine, but warn if it will diverge from what the skills expect (e.g., features.md without the template structure).

## What Does Not Belong Here

- Committing spec files to git. colbPowers does not assume the consumer commits `.specs/memory/`; leave that to the project's own decisions.
- GitHub integration. colbPowers is zero-dependency.
- Cross-harness behavior. This plugin targets Claude Code only for iteration 1.
```

- [ ] **Step 3: Verify frontmatter is valid YAML**

```bash
python3 -c "
import sys
with open('/home/adria/COLBAI/TFG/colbPowers/skills/using-colbpowers/SKILL.md') as f:
    content = f.read()
assert content.startswith('---\n'), 'missing opening frontmatter'
end = content.index('\n---\n', 4)
fm = content[4:end]
assert 'name:' in fm and 'description:' in fm, 'frontmatter missing required fields'
print('OK')
"
```

Expected: `OK`.

- [ ] **Step 4: Re-run the session-start smoke test — now it should include bootstrap content**

```bash
CLAUDE_PLUGIN_ROOT=/home/adria/COLBAI/TFG/colbPowers /home/adria/COLBAI/TFG/colbPowers/hooks/session-start | python3 -c "
import json, sys
d = json.load(sys.stdin)
ctx = d['hookSpecificOutput']['additionalContext']
assert 'using-colbpowers' in ctx, 'bootstrap not injected'
print('OK')
"
```

Expected: `OK`.

- [ ] **Step 5: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add skills/using-colbpowers/
git commit -q -m "plugin: add using-colbpowers bootstrap skill with orchestrator rules"
```

---

### Task 8: Write the `init-constitution` skill

**Files:**
- Create: `skills/init-constitution/SKILL.md`

- [ ] **Step 1: Create skill directory**

```bash
mkdir -p /home/adria/COLBAI/TFG/colbPowers/skills/init-constitution
```

- [ ] **Step 2: Write SKILL.md**

Create `/home/adria/COLBAI/TFG/colbPowers/skills/init-constitution/SKILL.md`:

```markdown
---
name: init-constitution
description: Architect Workflow A. Use when initializing a new project with colbPowers, when `.specs/memory/constitution.md` is missing, or when the user invokes `/colb:init`. Creates the project Constitution (vision, governance, tech stack) and an initial empty Feature Roadmap. This is the mandatory first step — no other colbPowers phase may proceed until the constitution exists.
---

# init-constitution (Architect Workflow A)

Run this skill when the project has no constitution yet. It is the Architect's Creation Protocol from the colbSpec workflow, rewired to use the colbPowers template files.

## Preconditions

- Current working directory is the project root.
- `.specs/memory/constitution.md` does not exist, OR the user explicitly asked to re-initialize.

If the constitution already exists and the user did not ask to re-initialize, stop and suggest `/colb:standards` (Workflow C — Maintenance) instead.

## Steps

### 1. Gather context

- **Check attached files first.** If the user appended files (requirements docs, existing specs, architecture notes), treat them as the **primary source of context** and read them before anything else.
- Ask the user for missing pieces, one question at a time:
  - What is the project's one-sentence purpose?
  - What is the tech stack? (Language, frameworks, tooling.)
  - Are there mandatory rules? (TDD? Specific runners? Coding style?)
  - What is explicitly out of scope?

### 2. Extract the three pillars

From the gathered context, derive:

- **Vision (The Why)** — one or two paragraphs on what the project exists to do.
- **Governance (The Rules)** — bulleted non-negotiables (testing, environment isolation, clean code principles).
- **Style (The How)** — tech stack + coding conventions (naming, typing, structure).

### 3. Fill gaps with sensible defaults

If the user gave a tech stack but no governance rules, propose defaults aligned with that stack (e.g., "pytest required for Python", "strict TypeScript if TS is chosen"). Flag each default as a default so the user can override.

### 4. Create the directory structure

```bash
mkdir -p .specs/memory/features
```

### 5. Write the constitution

Read `${CLAUDE_PLUGIN_ROOT}/templates/constitution_template.md` as the starting structure. Fill each section with the extracted content. Write to `.specs/memory/constitution.md`.

### 6. Write the initial features.md

Read `${CLAUDE_PLUGIN_ROOT}/templates/features_template.md` and write an empty roadmap to `.specs/memory/features.md`. The roadmap will be populated later via `/colb:roadmap`.

### 7. Write the active_context.json

Copy `${CLAUDE_PLUGIN_ROOT}/templates/active_context_template.json` to `.specs/memory/active_context.json`, with `active_feature_name` set to `null` and `status` set to `idle`.

### 8. Report

Output a bulleted summary of:
- The top 3 governance rules now in force.
- The tech stack.
- The path to the created files.

Then suggest the next step: `/colb:roadmap` to define the first features.

## Output Format

Do not ask for permission to overwrite if `constitution.md` already exists. Instead, stop and direct the user to `/colb:standards`.

All writes go through `Write` or `Edit` tools. Never instruct the user to edit files manually — you do it.
```

- [ ] **Step 3: Verify frontmatter**

```bash
python3 -c "
with open('/home/adria/COLBAI/TFG/colbPowers/skills/init-constitution/SKILL.md') as f:
    c = f.read()
assert c.startswith('---\n') and '\n---\n' in c[4:]
print('OK')
"
```

Expected: `OK`.

- [ ] **Step 4: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add skills/init-constitution/
git commit -q -m "architect: add init-constitution skill (Workflow A — creation protocol)"
```

---

### Task 9: Write the `update-features` skill

**Files:**
- Create: `skills/update-features/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p /home/adria/COLBAI/TFG/colbPowers/skills/update-features
```

- [ ] **Step 2: Write SKILL.md**

Create `/home/adria/COLBAI/TFG/colbPowers/skills/update-features/SKILL.md`:

```markdown
---
name: update-features
description: Architect Workflow B. Use when the user wants to add, update, or remove entries in the feature roadmap — natural phrasing includes "add a feature", "list requirements", "update the roadmap", or the `/colb:roadmap` slash command. Reads the constitution for alignment, updates `.specs/memory/features.md`, records feature dependencies, and notes how external deps will be mocked in tests.
---

# update-features (Architect Workflow B)

Updates the feature roadmap. Runs in-conversation.

## Preconditions

- `.specs/memory/constitution.md` exists. If not, stop and suggest `/colb:init`.

## Steps

### 1. Read the constitution

Load `.specs/memory/constitution.md` to stay aligned with vision and governance.

### 2. Read the current roadmap

Load `.specs/memory/features.md` if it exists. Determine whether the user wants to **add**, **update**, or **remove** an entry.

### 3. Gather details for each feature

Ask one question at a time:

- Feature name (short, kebab-case or Title Case, consistent with existing entries).
- One-paragraph description.
- Functional requirements (bulleted).
- Technical constraints (bulleted). If the feature introduces an external dependency, capture how it will be mocked in tests.
- Dependencies on other features in the roadmap.

### 4. Constraint checks

- **Isolation**: Can this feature be built independently? If not, record the prerequisite in `Dependencies`.
- **Constitution alignment**: Does this feature conflict with any rule in `constitution.md`? If yes, flag it and ask whether to update the constitution (send the user to `/colb:standards`) or drop the conflicting constraint.

### 5. Write the roadmap update

Use `${CLAUDE_PLUGIN_ROOT}/templates/features_template.md` as the structural reference. Edit `.specs/memory/features.md` in place — do not wipe existing entries.

### 6. Report

Summarize the added/updated feature in 3 bullets: name, purpose, key dependency. Then suggest the next step:

- If this is the first feature, suggest `/colb:plan <feature-name>`.
- Otherwise, let the user choose: add more roadmap entries, or start planning the one we just added.

## What not to do

- Do not scaffold source directories here. That is `create-feature`'s job.
- Do not write the design doc. That is `brainstorming`'s job.
- Do not generate tasks. That is `writing-plans`'s job.
```

- [ ] **Step 3: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add skills/update-features/
git commit -q -m "architect: add update-features skill (Workflow B — roadmap definition)"
```

---

### Task 10: Write the `update-constitution` skill

**Files:**
- Create: `skills/update-constitution/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p /home/adria/COLBAI/TFG/colbPowers/skills/update-constitution
```

- [ ] **Step 2: Write SKILL.md**

Create `/home/adria/COLBAI/TFG/colbPowers/skills/update-constitution/SKILL.md`:

```markdown
---
name: update-constitution
description: Architect Workflow C. Use when the user wants to change governance rules, swap tech stack items, or refine the vision — phrasings include "switch to", "update coding standards", "change the rule", or the `/colb:standards` slash command. Detects conflicts between new rules and existing ones or existing feature designs, asks the user to resolve, then merges the change into `.specs/memory/constitution.md` preserving unaffected sections.
---

# update-constitution (Architect Workflow C)

Modifies the project constitution with conflict detection.

## Preconditions

- `.specs/memory/constitution.md` exists. If not, stop and suggest `/colb:init`.

## Steps

### 1. Read the current constitution

Load `.specs/memory/constitution.md`. Identify the section the user's change applies to (governance rule? tech stack? style?).

### 2. Classify the change

- **Modification** — changing an existing rule (e.g., "switch from pytest to unittest").
- **Addition** — adding a new constraint (e.g., "max line length 100").
- **Removal** — deprecating a rule.

### 3. Conflict detection

Check the proposed change against:

- Other sections of the constitution (does the new rule contradict an existing one?).
- Existing feature roadmap entries in `.specs/memory/features.md` (does the change break a feature's technical constraints?).
- Existing per-feature designs in `.specs/memory/features/*/design.md` (does the change invalidate a design assumption?).

If a conflict is detected, list each conflict with its source and ask the user:

- "Override the existing rule?"
- "Update the affected feature design?"
- "Drop the new request?"

Do not merge until the user resolves each conflict.

### 4. Merge

Use `Edit` to weave the change into the correct section of `.specs/memory/constitution.md`. Preserve unaffected content. Do not rewrite the whole file.

### 5. Report

- Summarize what changed in 2–3 bullets.
- List any feature designs that need updating as a result.
- Strongly suggest running `/colb:review` to run a consistency check against current `src/`.

## What not to do

- Do not silently override rules. Every conflict gets user confirmation.
- Do not wipe the file on merge.
```

- [ ] **Step 3: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add skills/update-constitution/
git commit -q -m "architect: add update-constitution skill (Workflow C — maintenance with conflict detection)"
```

---

### Task 11: Write the `create-feature` skill

**Files:**
- Create: `skills/create-feature/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p /home/adria/COLBAI/TFG/colbPowers/skills/create-feature
```

- [ ] **Step 2: Write SKILL.md**

Create `/home/adria/COLBAI/TFG/colbPowers/skills/create-feature/SKILL.md`:

```markdown
---
name: create-feature
description: Scaffolds the per-feature directory structure under `.specs/memory/features/<feature>/` with template files for design.md and tasks.md. Use at the start of the Developer planning phase, before brainstorming runs. Called implicitly by `/colb:plan <feature>`. Updates `active_context.json` to mark the feature as the active focus.
---

# create-feature

Creates the per-feature spec folder and updates the active context pointer.

## Preconditions

- `.specs/memory/constitution.md` exists.
- `.specs/memory/features.md` contains an entry matching the requested feature name.

If either precondition fails, stop and report what is missing.

## Steps

### 1. Validate the feature name

- Must match an existing heading in `.specs/memory/features.md` (case-insensitive, whitespace-normalized).
- If ambiguous, list candidates and ask the user to pick one.

### 2. Normalize the feature slug

Convert the feature name to a filesystem-safe slug:
- Lowercase.
- Replace spaces and non-alphanumerics with `-`.
- Collapse consecutive `-`.
- Strip leading/trailing `-`.

Example: "User Authentication (OAuth)" → `user-authentication-oauth`.

### 3. Create the feature directory

```bash
mkdir -p .specs/memory/features/<slug>
```

### 4. Copy templates

- Copy `${CLAUDE_PLUGIN_ROOT}/templates/design_template.md` → `.specs/memory/features/<slug>/design.md`.
- Copy `${CLAUDE_PLUGIN_ROOT}/templates/tasks_template.md` → `.specs/memory/features/<slug>/tasks.md`.

Replace `[Feature Name]` placeholders in both files with the original feature name.

### 5. Update active_context.json

Edit `.specs/memory/active_context.json`:

```json
{
  "active_feature_name": "<original feature name>",
  "feature_folder_path": ".specs/memory/features/<slug>",
  "status": "planning"
}
```

### 6. Report

Tell the user:

- Path to the created `design.md` and `tasks.md`.
- That `active_context.json` is now pointing at this feature.
- That `brainstorming` will run next (via `/colb:plan`) to fill in the design.

## What not to do

- Do not write design content. That is `brainstorming`'s job.
- Do not write tasks. That is `writing-plans`'s job.
- Do not create `src/features/<slug>/`. That is the responsibility of the consuming project's implementation phase.
```

- [ ] **Step 3: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add skills/create-feature/
git commit -q -m "developer: add create-feature skill (scaffolds per-feature spec directory + active context)"
```

---

### Task 12: Write the `audit-project` skill

**Files:**
- Create: `skills/audit-project/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p /home/adria/COLBAI/TFG/colbPowers/skills/audit-project
```

- [ ] **Step 2: Write SKILL.md**

Create `/home/adria/COLBAI/TFG/colbPowers/skills/audit-project/SKILL.md`:

```markdown
---
name: audit-project
description: Reviewer phase wrapper. Use when the user invokes `/colb:review` or asks to "audit", "check compliance", or "verify the feature against the constitution". Dispatches the `colb-reviewer` subagent in a fresh context so the audit is unbiased by the implementation conversation. Produces a pass/warn/fail report.
---

# audit-project

Dispatches the Reviewer subagent.

## Preconditions

- `.specs/memory/constitution.md` exists.
- If a specific feature is named, `.specs/memory/features/<slug>/design.md` and `tasks.md` exist.

## Steps

### 1. Determine audit scope

Ask the user if not already specified:

- **Full project** — audit every feature currently marked complete.
- **Single feature** — audit just `<feature-name>`.

### 2. Gather inputs for the subagent prompt

Collect file paths (do not read content here — the subagent will):

- `.specs/memory/constitution.md`
- `.specs/memory/features.md`
- For full-project: every `.specs/memory/features/*/design.md` and `tasks.md`.
- For single-feature: `.specs/memory/features/<slug>/design.md`, `tasks.md`.
- Relevant `src/` paths.

### 3. Confirm dispatch

Per the subagent dispatch rule in `using-colbpowers`: print a one-sentence summary of what the subagent will do and wait for the user to confirm.

Example: "I'll dispatch `colb-reviewer` to audit `user-authentication-oauth` against the constitution and its design. It runs in a fresh context, so this conversation's history won't bias the review. Proceed?"

### 4. Dispatch the subagent

Invoke the `colb-reviewer` agent via the `Task` / `Agent` tool. Pass a prompt that:

- Specifies the scope (full project or named feature).
- Lists the artifact paths from step 2.
- Asks for a structured pass/warn/fail report.

### 5. Relay the report

When the subagent returns, present its report to the user unchanged. Then:

- **Green (pass)**: suggest `/colb:roadmap` to pick the next feature.
- **Yellow (warn)**: list the warnings and ask the user how to proceed.
- **Red (fail)**: list the blocking issues and suggest remediation (which skill to run next).

## What not to do

- Do not audit the project yourself. The whole point is a fresh-context subagent.
- Do not modify source code here. The Reviewer reports; the user decides whether to fix.
```

- [ ] **Step 3: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add skills/audit-project/
git commit -q -m "reviewer: add audit-project skill (dispatches colb-reviewer subagent)"
```

---

## Phase D — Agents and Commands

### Task 13: Write the `colb-reviewer` subagent

**Files:**
- Create: `agents/colb-reviewer.md`

- [ ] **Step 1: Write the agent definition**

Create `/home/adria/COLBAI/TFG/colbPowers/agents/colb-reviewer.md`:

```markdown
---
name: colb-reviewer
description: colbPowers Reviewer. Dispatched by the `audit-project` skill or `/colb:review` slash command. Audits a project or single feature against the constitution and per-feature design/tasks, emitting a structured pass/warn/fail report. Runs in a fresh context specifically so the audit is unbiased by the implementation conversation.
tools: Bash, Glob, Grep, Read
model: sonnet
color: amber
---

# colb-reviewer

You are the colbPowers Reviewer. You run in a fresh context. You have no history of how this code was written, and that is the point — your audit is unbiased.

## Inputs

The dispatching skill will pass you:

- The scope (full project or a named feature).
- Paths to the constitution, features roadmap, and per-feature artifacts (design, tasks).
- Paths to relevant source under `src/`.

Read every path it gives you before forming conclusions.

## Audit procedure

For each feature in scope:

### 1. Check the promise

Read `.specs/memory/features.md` entry for the feature. Identify:
- Functional requirements.
- Technical constraints.
- Dependencies.

### 2. Check the design

Read `.specs/memory/features/<slug>/design.md`. Confirm:
- Architecture is articulated (not just stubs).
- Success criteria are concrete and observable.

### 3. Check the plan

Read `.specs/memory/features/<slug>/tasks.md`. Confirm:
- Tasks are checked off where source exists.
- Every unchecked task has a reason or is the current in-progress item per `active_context.json`.

### 4. Check the reality

Read the source under `src/` that corresponds to this feature. Confirm:
- Every functional requirement from features.md is implemented.
- Every constitution rule applicable to this code is honored. Specifically:
  - If the constitution mandates tests, check that tests exist and that their filenames correspond to the source files.
  - If the constitution mandates a runner (e.g., `uv run`), check for a runner invocation path.
  - If the constitution mandates a style (e.g., type hints, naming conventions), spot-check conformance.

### 5. Issue classification

Classify every finding as:

- **PASS** — requirement is satisfied.
- **WARN** — requirement is partially satisfied, or drift exists but is non-blocking (e.g., outdated comment).
- **FAIL** — requirement is unsatisfied in a way that violates the constitution.

## Output format

Emit a single markdown report. Structure:

```markdown
# colb-reviewer Report

**Scope:** [full project | feature: <name>]
**Verdict:** GREEN | YELLOW | RED
**Date:** YYYY-MM-DD

## Summary

- [Total features audited]
- [Counts: PASS / WARN / FAIL]

## Findings

### FAIL
- [Finding 1 — what's broken, where (file:line), which constitution/design section it violates, suggested remediation]

### WARN
- [Finding 1 — same shape]

### PASS
- [Concise checklist of what's satisfied]
```

Verdict rule:
- **RED** if any FAIL.
- **YELLOW** if any WARN but no FAIL.
- **GREEN** if all PASS.

## What not to do

- Do not modify source code. You are a reviewer.
- Do not speculate about features not listed in `features.md`.
- Do not re-read files outside the paths you were given unless they are referenced by those paths (e.g., a design.md references src/auth/oauth.py — read that, but not unrelated files).
- Do not invent constitution rules. Quote them directly from `constitution.md`.
```

- [ ] **Step 2: Verify frontmatter**

```bash
python3 -c "
with open('/home/adria/COLBAI/TFG/colbPowers/agents/colb-reviewer.md') as f:
    c = f.read()
assert c.startswith('---\n')
fm_end = c.index('\n---\n', 4)
fm = c[4:fm_end]
for key in ('name:', 'description:', 'tools:', 'model:'):
    assert key in fm, f'missing {key}'
print('OK')
"
```

Expected: `OK`.

- [ ] **Step 3: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add agents/colb-reviewer.md
git commit -q -m "reviewer: add colb-reviewer subagent (fresh-context audit)"
```

---

### Task 14: Write slash command `/colb:init`

**Files:**
- Create: `commands/colb:init.md`

- [ ] **Step 1: Write the command file**

Create `/home/adria/COLBAI/TFG/colbPowers/commands/colb:init.md`:

```markdown
---
description: "Architect Workflow A — create the project Constitution and initial Feature Roadmap. Mandatory first step when initializing a new project with colbPowers."
---

Invoke the `colbPowers:init-constitution` skill and follow its steps.

If `.specs/memory/constitution.md` already exists, stop and inform the user that `/colb:standards` (Workflow C — Maintenance) is the correct tool for updating an existing constitution.

If the user appended files to this command, treat them as primary context per the skill's instructions.
```

- [ ] **Step 2: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add 'commands/colb:init.md'
git commit -q -m "plugin: add /colb:init slash command"
```

---

### Task 15: Write slash command `/colb:roadmap`

**Files:**
- Create: `commands/colb:roadmap.md`

- [ ] **Step 1: Write the command file**

Create `/home/adria/COLBAI/TFG/colbPowers/commands/colb:roadmap.md`:

```markdown
---
description: "Architect Workflow B — add, update, or remove entries in the feature roadmap. Requires constitution to already exist."
---

Invoke the `colbPowers:update-features` skill and follow its steps.

If `.specs/memory/constitution.md` is missing, stop and direct the user to `/colb:init` first.

Ask the user what they want to do (add, update, or remove a feature) if the command was invoked without further context.
```

- [ ] **Step 2: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add 'commands/colb:roadmap.md'
git commit -q -m "plugin: add /colb:roadmap slash command"
```

---

### Task 16: Write slash command `/colb:standards`

**Files:**
- Create: `commands/colb:standards.md`

- [ ] **Step 1: Write the command file**

Create `/home/adria/COLBAI/TFG/colbPowers/commands/colb:standards.md`:

```markdown
---
description: "Architect Workflow C — update the constitution (governance, tech stack, style) with conflict detection."
---

Invoke the `colbPowers:update-constitution` skill and follow its steps.

If `.specs/memory/constitution.md` does not yet exist, stop and direct the user to `/colb:init` first.

When the skill detects a conflict with existing rules, feature roadmap entries, or feature designs, surface every conflict to the user and wait for resolution before merging.
```

- [ ] **Step 2: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add 'commands/colb:standards.md'
git commit -q -m "plugin: add /colb:standards slash command"
```

---

### Task 17: Write slash command `/colb:plan`

**Files:**
- Create: `commands/colb:plan.md`

- [ ] **Step 1: Write the command file**

Create `/home/adria/COLBAI/TFG/colbPowers/commands/colb:plan.md`:

```markdown
---
description: "Developer planning — scaffold a feature directory, brainstorm its design, and generate the tasks plan. Accepts the feature name as an argument."
---

The argument after `/colb:plan` is the feature name. If the user did not supply one, read `.specs/memory/features.md` and ask them to pick from the listed features.

Then run this three-skill sequence in this exact order:

1. Invoke the `colbPowers:create-feature` skill. It scaffolds `.specs/memory/features/<slug>/` with `design.md` and `tasks.md` template stubs and updates `active_context.json`.
2. Invoke the `superpowers:brainstorming` skill. For iteration 1, it still writes to its default location — relay the output file path to the user, and follow up by copying the result into `.specs/memory/features/<slug>/design.md` using the `Edit` or `Write` tool.
3. Invoke the `superpowers:writing-plans` skill. Same path caveat: capture the generated plan and save it to `.specs/memory/features/<slug>/tasks.md`.

(Artifact-path adaptation in the bundled superpowers skills themselves is deferred to iteration 2; the orchestrator handles the relocation for now.)

When complete, update `.specs/memory/active_context.json` — set `status` to `ready-to-execute`.

Suggest `/colb:execute <feature>` as the next step.
```

- [ ] **Step 2: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add 'commands/colb:plan.md'
git commit -q -m "plugin: add /colb:plan slash command (create-feature -> brainstorm -> write tasks)"
```

---

### Task 18: Write slash command `/colb:execute`

**Files:**
- Create: `commands/colb:execute.md`

- [ ] **Step 1: Write the command file**

Create `/home/adria/COLBAI/TFG/colbPowers/commands/colb:execute.md`:

```markdown
---
description: "Developer execution — dispatch one fresh subagent per task in the feature's tasks.md. Asks for confirmation first."
---

The argument after `/colb:execute` is the feature name. If omitted, read `.specs/memory/active_context.json` and use `active_feature_name`. If neither is available, list the features in `.specs/memory/features.md` and ask the user to choose.

Preconditions:

- `.specs/memory/features/<slug>/tasks.md` exists.
- `.specs/memory/constitution.md` exists.

Per the subagent dispatch rule in `using-colbpowers`: before dispatching, summarize the scope in one sentence ("I'll dispatch N subagents for the M tasks in `<feature>/tasks.md`, each with TDD + verification-before-completion. Proceed?") and wait for user confirmation.

On confirmation, invoke the `superpowers:subagent-driven-development` skill. Pass it the path to `tasks.md` as the plan. The skill handles per-task dispatch, two-stage review, and commit cadence.

After each task completes, update `.specs/memory/active_context.json` to reflect progress.

When all tasks complete, update `active_context.json` with `status: complete` and suggest `/colb:review <feature>`.
```

- [ ] **Step 2: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add 'commands/colb:execute.md'
git commit -q -m "plugin: add /colb:execute slash command (dispatches subagent-driven-development)"
```

---

### Task 19: Write slash command `/colb:review`

**Files:**
- Create: `commands/colb:review.md`

- [ ] **Step 1: Write the command file**

Create `/home/adria/COLBAI/TFG/colbPowers/commands/colb:review.md`:

```markdown
---
description: "Reviewer phase — dispatch a fresh-context audit subagent. Argument is optional; omit to audit the full project."
---

The argument after `/colb:review` is the feature name. If omitted, audit the full project (every feature currently marked complete in `active_context.json` history or by source presence).

Invoke the `colbPowers:audit-project` skill and follow its steps. The skill:

1. Validates preconditions.
2. Gathers artifact paths (constitution, features, per-feature design + tasks, relevant src/).
3. Asks the user to confirm subagent dispatch.
4. Dispatches the `colb-reviewer` subagent.
5. Relays the structured pass/warn/fail report to the user.

Do not short-circuit to your own audit — the point of the Reviewer phase is fresh context.
```

- [ ] **Step 2: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add 'commands/colb:review.md'
git commit -q -m "plugin: add /colb:review slash command (dispatches colb-reviewer via audit-project)"
```

---

## Phase E — Integration and Verification

### Task 20: Write the smoke-test script

**Files:**
- Create: `tests/smoke-test.sh`

- [ ] **Step 1: Create tests directory**

```bash
mkdir -p /home/adria/COLBAI/TFG/colbPowers/tests
```

- [ ] **Step 2: Write the smoke test**

Create `/home/adria/COLBAI/TFG/colbPowers/tests/smoke-test.sh`:

```bash
#!/usr/bin/env bash
# colbPowers structural smoke test.
# Validates plugin manifest, skill frontmatter, command frontmatter, and hook output.
# Run from the plugin root.

set -euo pipefail

ROOT="$(cd "$(dirname "$0")/.." && pwd)"
cd "$ROOT"

fail=0

say_ok() { printf '[OK]   %s\n' "$1"; }
say_fail() { printf '[FAIL] %s\n' "$1"; fail=1; }

# 1. Plugin manifest is valid JSON
if python3 -c "import json; json.load(open('.claude-plugin/plugin.json'))" 2>/dev/null; then
  say_ok "plugin.json is valid JSON"
else
  say_fail "plugin.json is not valid JSON"
fi

if python3 -c "import json; json.load(open('.claude-plugin/marketplace.json'))" 2>/dev/null; then
  say_ok "marketplace.json is valid JSON"
else
  say_fail "marketplace.json is not valid JSON"
fi

# 2. hooks/hooks.json is valid JSON
if python3 -c "import json; json.load(open('hooks/hooks.json'))" 2>/dev/null; then
  say_ok "hooks/hooks.json is valid JSON"
else
  say_fail "hooks/hooks.json is not valid JSON"
fi

# 3. session-start is executable
if [ -x hooks/session-start ]; then
  say_ok "hooks/session-start is executable"
else
  say_fail "hooks/session-start is not executable"
fi

# 4. session-start emits valid JSON with hookSpecificOutput
if CLAUDE_PLUGIN_ROOT="$ROOT" ./hooks/session-start | python3 -c "
import json, sys
d = json.load(sys.stdin)
assert 'hookSpecificOutput' in d, 'missing hookSpecificOutput'
assert 'additionalContext' in d['hookSpecificOutput'], 'missing additionalContext'
assert 'using-colbpowers' in d['hookSpecificOutput']['additionalContext'], 'bootstrap skill not injected'
" 2>/dev/null; then
  say_ok "session-start emits valid bootstrap JSON"
else
  say_fail "session-start output malformed or missing bootstrap"
fi

# 5. Every SKILL.md has frontmatter with name + description
for skill in skills/*/SKILL.md; do
  if python3 -c "
import sys
c = open('$skill').read()
assert c.startswith('---\n'), 'no opening fence'
end = c.index('\n---\n', 4)
fm = c[4:end]
assert 'name:' in fm and 'description:' in fm, 'missing name or description'
" 2>/dev/null; then
    say_ok "skill frontmatter: $skill"
  else
    say_fail "skill frontmatter malformed: $skill"
  fi
done

# 6. Every command file has frontmatter with description
for cmd in commands/*.md; do
  if python3 -c "
c = open('$cmd').read()
assert c.startswith('---\n'), 'no opening fence'
end = c.index('\n---\n', 4)
fm = c[4:end]
assert 'description:' in fm, 'missing description'
" 2>/dev/null; then
    say_ok "command frontmatter: $cmd"
  else
    say_fail "command frontmatter malformed: $cmd"
  fi
done

# 7. Every agent file has frontmatter with name + description + tools + model
for ag in agents/*.md; do
  if python3 -c "
c = open('$ag').read()
assert c.startswith('---\n'), 'no opening fence'
end = c.index('\n---\n', 4)
fm = c[4:end]
for key in ('name:', 'description:', 'tools:', 'model:'):
    assert key in fm, f'missing {key}'
" 2>/dev/null; then
    say_ok "agent frontmatter: $ag"
  else
    say_fail "agent frontmatter malformed: $ag"
  fi
done

# 8. Required native skills all present
required_skills="using-colbpowers init-constitution update-features update-constitution create-feature audit-project"
for s in $required_skills; do
  if [ -f "skills/$s/SKILL.md" ]; then
    say_ok "native skill present: $s"
  else
    say_fail "native skill missing: $s"
  fi
done

# 9. Required slash commands all present
required_cmds="colb:init colb:roadmap colb:standards colb:plan colb:execute colb:review"
for c in $required_cmds; do
  if [ -f "commands/$c.md" ]; then
    say_ok "slash command present: $c"
  else
    say_fail "slash command missing: $c"
  fi
done

# 10. Required templates all present
required_templates="constitution_template.md features_template.md design_template.md tasks_template.md active_context_template.json"
for t in $required_templates; do
  if [ -f "templates/$t" ]; then
    say_ok "template present: $t"
  else
    say_fail "template missing: $t"
  fi
done

if [ "$fail" -eq 0 ]; then
  echo
  echo "SMOKE TEST PASSED"
  exit 0
else
  echo
  echo "SMOKE TEST FAILED"
  exit 1
fi
```

- [ ] **Step 3: Make it executable**

```bash
chmod +x /home/adria/COLBAI/TFG/colbPowers/tests/smoke-test.sh
```

- [ ] **Step 4: Run it**

```bash
/home/adria/COLBAI/TFG/colbPowers/tests/smoke-test.sh
```

Expected: every `[OK]` line, ending with `SMOKE TEST PASSED`. If any `[FAIL]` appears, fix the referenced file before proceeding.

- [ ] **Step 5: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add tests/smoke-test.sh
git commit -q -m "tests: add structural smoke test for manifest, skills, commands, agents, templates, hook output"
```

---

### Task 21: Install plugin locally and verify load

**Files:** no new files; verification task.

- [ ] **Step 1: Register the local marketplace in Claude Code**

In a Claude Code session (the user's live terminal), instruct them to run:

```
/plugin marketplace add /home/adria/COLBAI/TFG/colbPowers
```

Expected: Claude Code acknowledges the marketplace registration.

- [ ] **Step 2: Install the plugin**

```
/plugin install colbPowers@colbPowers-local
```

Expected: Claude Code reports successful installation.

- [ ] **Step 3: Verify plugin is listed**

```
/plugin list
```

Expected: `colbPowers` appears with version `0.1.0`.

- [ ] **Step 4: Verify skills load**

Open a new Claude Code session and observe whether the `using-colbpowers` bootstrap fires (message should mention "You have colbPowers" or the skill content). If it does not, inspect the Claude Code hook logs to diagnose.

- [ ] **Step 5: Document the installation result in a session-log file**

Create `/home/adria/COLBAI/TFG/colbPowers/docs/installation-log-2026-04-19.md` with:

- Command sequence run.
- Exact output from `/plugin list`.
- Whether the SessionStart bootstrap appeared in a new session.
- Any deviations from expected behavior.

- [ ] **Step 6: Commit**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add docs/installation-log-2026-04-19.md
git commit -q -m "docs: log initial plugin install + smoke results"
```

---

### Task 22: End-to-end smoke — Architect flow

**Files:** no new files; verification task against a fresh consumer project.

- [ ] **Step 1: Create a disposable consumer project**

```bash
mkdir -p /tmp/colb-smoke-architect && cd /tmp/colb-smoke-architect && git init -q
```

- [ ] **Step 2: Start a Claude Code session in that directory**

Open Claude Code in `/tmp/colb-smoke-architect`.

- [ ] **Step 3: Confirm proactive suggestion**

Expected: the orchestrator greets with the "No constitution yet" suggestion from the `using-colbpowers` state table.

Criterion: if the greeting does not mention missing constitution or `/colb:init`, log the actual output and investigate `using-colbpowers/SKILL.md` state-detection rules.

- [ ] **Step 4: Run `/colb:init`**

Describe a minimal project (e.g., "A command-line note-taker in Python"). Let the skill gather context and write files.

Expected output files:
- `.specs/memory/constitution.md` (populated).
- `.specs/memory/features.md` (template stub).
- `.specs/memory/active_context.json` (status: idle).

Verify:
```bash
ls /tmp/colb-smoke-architect/.specs/memory/
cat /tmp/colb-smoke-architect/.specs/memory/constitution.md | head -10
```

- [ ] **Step 5: Run `/colb:roadmap`**

Add one feature, e.g., "Markdown rendering for notes."

Expected: `features.md` contains the new feature with description, requirements, and dependencies.

- [ ] **Step 6: Record results**

Append outcomes to `/home/adria/COLBAI/TFG/colbPowers/docs/installation-log-2026-04-19.md`:
- Initial greeting text.
- File diffs after `/colb:init`.
- File diffs after `/colb:roadmap`.
- Any friction encountered.

- [ ] **Step 7: Commit the log update**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add docs/installation-log-2026-04-19.md
git commit -q -m "docs: record Architect-flow smoke test results"
```

---

### Task 23: End-to-end smoke — Developer + Reviewer flow

**Files:** no new files; verification task continuing from Task 22.

- [ ] **Step 1: Run `/colb:plan <feature>`**

Using the feature added in Task 22.

Expected:
- `.specs/memory/features/<slug>/` directory exists.
- `design.md` has been brainstormed and populated (via superpowers:brainstorming, then relocated by the orchestrator per the `/colb:plan` command instructions).
- `tasks.md` contains a concrete TDD plan (from superpowers:writing-plans, relocated).
- `active_context.json` reflects `status: ready-to-execute`.

- [ ] **Step 2: Run `/colb:execute <feature>`**

Confirm the subagent dispatch prompt.

Expected:
- The orchestrator summarizes the scope and waits for "proceed."
- On proceed, subagent-driven-development dispatches one subagent per task.
- Source files appear under `src/` as tasks complete.
- `active_context.json` updates after each task.

If the tasks in `tasks.md` from the superpowers skill are not formatted to match what subagent-driven-development expects, that is a known iteration-2 issue — note it in the log but do not block.

- [ ] **Step 3: Run `/colb:review <feature>`**

Confirm the Reviewer dispatch.

Expected:
- The orchestrator summarizes and waits for "proceed."
- `colb-reviewer` subagent runs in fresh context.
- Report returns with verdict GREEN / YELLOW / RED and structured findings.

- [ ] **Step 4: Evaluate success criteria from spec §10**

Walk through each of the 6 iteration-1 success criteria:

1. Orchestrator detects empty `.specs/memory/` → suggests `/colb:init`. **Pass / fail?**
2. `/colb:init` produces `constitution.md` + initial `features.md`. **Pass / fail?**
3. `/colb:roadmap` edits `features.md`. **Pass / fail?**
4. `/colb:plan <feature>` scaffolds + brainstorms + writes plan. **Pass / fail?**
5. `/colb:execute <feature>` dispatches subagents + TDD gates fire. **Pass / fail?**
6. `/colb:review <feature>` dispatches Reviewer subagent + reports. **Pass / fail?**

- [ ] **Step 5: Write the iteration-1 closing report**

Append to `/home/adria/COLBAI/TFG/colbPowers/docs/installation-log-2026-04-19.md` a final section "Iteration 1 Verdict" with:
- Pass/fail per success criterion.
- Known iteration-2 gaps (e.g., superpowers skills still write to default paths; manual relocation required).
- Any unexpected regressions.

- [ ] **Step 6: Commit the closing report**

```bash
cd /home/adria/COLBAI/TFG/colbPowers
git add docs/installation-log-2026-04-19.md
git commit -q -m "docs: iteration 1 smoke-test verdict (Developer + Reviewer flows)"
```

---

## Self-Review

**1. Spec coverage:**

- §3 Identity — Task 1 (seed), Task 2 (prune + git init), Task 3 (manifest), Task 4 (LICENSE/NOTICE/README/CLAUDE.md). Covered.
- §4 Artifact Model — Task 5 (templates: constitution, features, design, tasks, active_context). Covered.
- §5 Orchestration — Task 7 (using-colbpowers bootstrap with state table + dispatch rule). Covered.
- §6 Skill Bundling — Tasks 7–12 (6 native skills). Dropped skills removed in Task 2. Bundled superpowers skills are left as-copied (per §6 iteration-1 scope). Covered.
- §7 Directory Structure — produced by Tasks 1–20 aggregated. Covered.
- §8 Bootstrap — Task 6 (session-start rewrite), Task 7 (using-colbpowers body). Covered.
- §9 Install Path — Task 21. Covered.
- §10 Success Criteria — Task 23 explicitly walks each of the 6 criteria. Covered.

**2. Placeholder scan:**

- No "TBD" or "fill in details" in task content.
- All file contents shown in full.
- Known-deferred items (e.g., superpowers skill artifact-path adaptation) are explicitly called out as iteration-2 work and handled in iteration 1 via the orchestrator relocating files — no hidden "TODO" debt.

**3. Type consistency:**

- Feature slug rule defined once in Task 11 (create-feature) and referenced consistently elsewhere (`<slug>` in paths).
- `active_context.json` shape defined in Task 5 template and updated consistently in Tasks 11, 17, 18, 22, 23.
- Subagent dispatch rule defined once in Task 7 (using-colbpowers) and referenced by Tasks 12, 18, 19.
- Skill invocation style uses `colbPowers:<skill-name>` consistently in commands.

No inconsistencies identified.

---

## Execution Handoff

Plan complete and saved to `/home/adria/COLBAI/TFG/colbPowers/docs/plans/2026-04-19-colbpowers-iteration1.md`. Two execution options:

1. **Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration.
2. **Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints.

**Which approach?**
