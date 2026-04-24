# colbPowers — Plugin Design

**Date:** 2026-04-19
**Status:** approved, pending implementation plan
**Author:** Adrià Cebrián (with Claude)

## 1. Purpose and Scope

`colbPowers` is a general-purpose Claude Code plugin that combines two systems:

- The **spec-first structure** of `colbSpec` (constitution → features → design → tasks, with Architect / Developer / Reviewer phase semantics).
- The **execution discipline** of `superpowers` (brainstorming, TDD, subagent-driven implementation, two-stage review, verification-before-completion, worktrees, systematic debugging).

The motivation: `colbSpec` today is an extremely simple spec-first shell. It needs superpowers' execution loop. Rather than fork superpowers, we bundle a curated subset of its skills into a new plugin with its own identity and a spec-first orchestration layer on top.

**Target harness**: Claude Code for iteration 1. OpenCode support is explicitly deferred; cross-harness infrastructure from superpowers (`.opencode/`, `gemini-extension.json`, `.cursor-plugin/`, `.codex/`) is removed from the seed and may be reintroduced later. When OpenCode support is added, the `skills/` directory and `SKILL.md` files remain portable — only a thin adapter (`.opencode/plugins/colbpowers.js` to register the skills path and inject the `using-colbpowers` bootstrap, plus `package.json` main field) needs to be written.

**Reusability**: general-purpose. Installable on any project. Constitution / features / design / tasks are *mechanisms*; each consuming project fills in its own content.

## 2. Non-Goals

- Not a fork of superpowers — no upstream git relationship.
- No GitHub integration. The `fetch_issues`, `sync-specs`, and `repo-lister` skills from `colbSpec` are dropped; the plugin has zero dependencies beyond Claude Code itself.
- Not backwards-compatible with the existing `colbSpec` repo at `/home/adria/COLBAI/TFG/ProjectAssistant/Part2/`. That repo keeps working as-is; colbPowers is a standalone successor.
- Not a replacement for `superpowers`. A user could install both, but colbPowers is intended to stand alone; the bundled skills are adapted to the spec-first artifacts and will diverge.

## 3. Identity

- **Name**: `colbPowers`
- **Location (during development)**: `/home/adria/COLBAI/TFG/colbPowers/`
- **License**: MIT. Skills adapted from `obra/superpowers` (MIT) — attribution preserved in `NOTICE`.
- **Seeding**: copy the contents of the installed superpowers plugin (`/home/adria/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.7/`), then `rm -rf .git && git init` so there is no upstream relationship.

## 4. Artifact Model

Four layers of project artifacts, all under `.specs/memory/` in the consuming project:

| Artifact | Scope | Written by | Template |
|---|---|---|---|
| `constitution.md` | Project-wide. Vision + Governance + Style (tech stack, coding standards, non-negotiables). | Architect phase | `templates/constitution_template.md` |
| `features.md` | Project-wide roadmap. Feature names, intent, dependencies. | Architect phase (roadmap mode) | `templates/features_template.md` |
| `features/<feature>/design.md` | Per-feature full design doc from brainstorming. | Developer phase (planning) | `templates/design_template.md` (new) |
| `features/<feature>/tasks.md` | Per-feature implementation plan. Granular task checklist produced by writing-plans, tuned to this format. | Developer phase (planning) | `templates/tasks_template.md` |
| `active_context.json` | Current project state: feature in progress, phase, status. Used by orchestrator for state detection. | Developer phase | `templates/active_context_template.json` |

Shift from `colbSpec`: per-feature artifacts move from `src/features/<name>/` into `.specs/memory/features/<name>/`. Source code and spec artifacts stay separated; specs are easier to find and reason about.

## 5. Orchestration and Phases

### 5.1 The orchestrator is the main conversation

The orchestrator is **Claude itself in the user's main chat**, not a separate subagent. Making it a subagent would defeat the in-conversation design goal, since Claude Code subagents start with fresh context and lose chat history.

The orchestrator behavior is installed via a `SessionStart` hook that injects the `using-colbpowers` skill into context at session start (same mechanism superpowers uses for its own `using-superpowers` bootstrap).

### 5.2 Phases

| Phase | Mechanism | Why |
|---|---|---|
| **Architect** | In-conversation skills | Chat nuance is load-bearing for writing the constitution and roadmap; isolation has no benefit. |
| **Developer — planning** | In-conversation skills (brainstorming, writing-plans) | Chat history helps produce coherent design + plan docs. |
| **Developer — execution** | Dispatched subagents (one fresh subagent per task) | Isolation per task; each task gets its own TDD loop and two-stage review. |
| **Reviewer** | One dispatched subagent per run | Fresh context = clean audit, free of implementation bias. |

### 5.3 Proactive suggestions

The `using-colbpowers` bootstrap instructs the orchestrator to read `.specs/memory/` at session start and propose the next phase action based on project state. Examples:

| Detected state | Orchestrator response |
|---|---|
| No `constitution.md` | Suggest `/colb:init`. |
| Constitution exists, empty `features.md` | Suggest `/colb:roadmap`. |
| `active_context.json` shows feature in `planning` status | Suggest continuing brainstorm or `/colb:plan <feature>`. |
| `tasks.md` exists but no matching `src/features/<name>/` | Suggest `/colb:execute <feature>`. |
| Feature implementation looks complete | Suggest `/colb:review <feature>`. |

### 5.4 Slash commands

Slash commands are explicit entry points to the same skills the orchestrator would auto-pick:

| Command | Target skill(s) |
|---|---|
| `/colb:init` | `init-constitution` |
| `/colb:roadmap` | `update-features` |
| `/colb:standards` | `update-constitution` |
| `/colb:plan <feature>` | `create-feature` → `brainstorming` → `writing-plans` |
| `/colb:execute <feature>` | `subagent-driven-development` |
| `/colb:review [<feature>]` | `audit-project` |

Auto-activating skills (via frontmatter `description`) still fire from natural user phrasing; the slash commands are for explicit invocation when the user wants to skip suggestions.

### 5.5 Subagent dispatch rule

The orchestrator **asks before dispatching any subagent**. Subagents are high-cost and their behavior is opaque from the user's view. Skills that run in-conversation can be auto-invoked; subagents require a clear "yes, proceed."

## 6. Skill Bundling

All 14 superpowers skills are copied into `skills/` as the starting point. From there:

### Adapt (reference colbPowers artifacts)

| Skill | Change |
|---|---|
| `brainstorming` | Output path `.specs/memory/features/<feature>/design.md` (not `.specs/memory/docs/specs/…`). Read `constitution.md` as constraint input. Drop the forced "commit design doc" step — we do not assume the consuming project commits specs. |
| `writing-plans` | Output path `.specs/memory/features/<feature>/tasks.md`. Use `tasks_template.md` format. Read constitution + design as inputs. |
| `subagent-driven-development` | Reads `tasks.md` in colbPowers format. Updates `active_context.json` as tasks complete (status, current focus). |
| `requesting-code-review` | Reviewer variant: inputs are constitution + features + design + tasks + `src/`. Emits pass / warn / fail per `colbSpec` Reviewer convention. |
| `executing-plans` | Same artifact adaptations as `subagent-driven-development` (batch variant). |

### Keep as-is

`test-driven-development`, `verification-before-completion`, `systematic-debugging`, `using-git-worktrees`, `finishing-a-development-branch`, `receiving-code-review`, `dispatching-parallel-agents`. No colbPowers-specific references required.

### Drop

- `writing-skills` — meta, not needed for end users.
- `using-superpowers` — replaced by `using-colbpowers` bootstrap.

### New skills (colbPowers-native)

| Skill | Purpose |
|---|---|
| `using-colbpowers` | Bootstrap. Installed by `SessionStart` hook. Contains orchestrator rules from §5. |
| `init-constitution` | Architect Workflow A. Fed by `/colb:init`. |
| `update-features` | Architect Workflow B (roadmap). Fed by `/colb:roadmap`. |
| `update-constitution` | Architect Workflow C (maintenance + conflict detection). Fed by `/colb:standards`. |
| `create-feature` | Scaffolds `.specs/memory/features/<feature>/` and templates. Called by `/colb:plan` before brainstorming and writing-plans. |
| `audit-project` | Reviewer-phase wrapper. Dispatches a fresh subagent that runs `requesting-code-review` over colbPowers inputs. Fed by `/colb:review`. |

### Incremental customization order

1. **Iteration 1**: six new colbPowers-native skills + `using-colbpowers` bootstrap + `hooks/` + `commands/` wiring. Bundled superpowers skills still behave as-copied.
2. **Iteration 2**: adapt `brainstorming` and `writing-plans` (artifact paths).
3. **Iteration 3**: adapt `subagent-driven-development` and `requesting-code-review` (read `.specs/memory/` state, update `active_context.json`).
4. **Iteration 4**: prune any copied skills that turn out unused.

## 7. Directory Structure

```
colbPowers/
├── .claude-plugin/
│   ├── plugin.json            # name, version, description, author, license
│   └── marketplace.json       # optional, for local /plugin install
├── hooks/
│   ├── hooks.json             # SessionStart matcher: startup|clear|compact
│   └── session-start          # bash script adapted from superpowers
├── commands/
│   ├── colb:init.md
│   ├── colb:roadmap.md
│   ├── colb:standards.md
│   ├── colb:plan.md
│   ├── colb:execute.md
│   └── colb:review.md
├── agents/
│   └── colb-reviewer.md       # one subagent persona (fresh-context audit)
├── skills/
│   ├── using-colbpowers/
│   ├── init-constitution/
│   ├── update-features/
│   ├── update-constitution/
│   ├── create-feature/
│   ├── audit-project/
│   ├── brainstorming/
│   ├── writing-plans/
│   ├── subagent-driven-development/
│   ├── requesting-code-review/
│   ├── executing-plans/
│   ├── test-driven-development/
│   ├── verification-before-completion/
│   ├── systematic-debugging/
│   ├── using-git-worktrees/
│   ├── finishing-a-development-branch/
│   ├── receiving-code-review/
│   └── dispatching-parallel-agents/
├── templates/
│   ├── constitution_template.md
│   ├── features_template.md
│   ├── design_template.md     # new
│   ├── tasks_template.md
│   └── active_context_template.json
├── docs/
│   └── specs/                 # design docs for colbPowers itself
├── CLAUDE.md
├── README.md
├── LICENSE
└── NOTICE
```

Removed from the seed: `.opencode/`, `.codex/`, `.cursor-plugin/`, `gemini-extension.json`, `package.json`, `scripts/`, `tests/`, `.specs/memory/docs/` (the superpowers contributor docs — we keep `docs/` only for colbPowers' own design docs), `CHANGELOG.md`, `RELEASE-NOTES.md`, `AGENTS.md`, `GEMINI.md`, `CODE_OF_CONDUCT.md`, `.github/`, `.gitattributes`, `.version-bump.json`.

## 8. Bootstrap Mechanism

`hooks/hooks.json`:

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

`hooks/session-start` is adapted from superpowers' version. It reads `skills/using-colbpowers/SKILL.md`, escapes it for JSON, emits `hookSpecificOutput.additionalContext` wrapped in an `<EXTREMELY_IMPORTANT>` block. Cursor / Copilot / Codex variants are removed for iteration 1.

The `using-colbpowers` skill body contains the orchestrator behavior rules (state detection table, proactive suggestion rules, isolation rules, subagent-dispatch-requires-confirmation rule, skill priority).

## 9. Install Path

Two supported install paths during development:

1. **Local**: `/plugin install /home/adria/COLBAI/TFG/colbPowers` via a local `marketplace.json`. Fast iteration.
2. **From git**: later, push to a GitHub repo and install via marketplace URL.

## 10. Success Criteria

Iteration 1 is complete when, in a fresh consuming project with colbPowers installed, a user can:

1. Open a session → orchestrator detects empty `.specs/memory/` → suggests `/colb:init`.
2. Run `/colb:init` with a short project description → `init-constitution` skill produces `constitution.md` + initial `features.md`.
3. Run `/colb:roadmap` to add a feature → `update-features` edits `features.md`.
4. Run `/colb:plan <feature>` → `create-feature` scaffolds the feature directory, `brainstorming` (still superpowers-flavored) produces a design doc (path adaptation deferred to iteration 2), `writing-plans` (still superpowers-flavored) produces a plan.
5. Run `/colb:execute <feature>` → `subagent-driven-development` dispatches task subagents per the plan. TDD gates fire inside each subagent.
6. Run `/colb:review <feature>` → `audit-project` dispatches a Reviewer subagent that reports pass / warn / fail.

Iterations 2–4 progressively close the path-adaptation gap so artifacts land in `.specs/memory/features/<feature>/`.

## 11. Open Questions

1. Slash command naming: `colb:init` or `colb-init` or `colbpowers:init`? Claude Code supports both `:` and `-` in command filenames; the `:` style groups commands visually under a namespace in some UIs but may have shell escaping issues in a few contexts. Default: `colb:init`. Revisit if user testing shows friction.
2. Should `audit-project` be a skill that dispatches the `colb-reviewer` agent, or should `colb-reviewer` be the only entry point and the skill not exist? Defaulting to the skill-dispatches-agent split, since it matches how other phases split orchestration from execution, but this is a single-source-of-truth question we can revisit.
3. When the orchestrator detects state that doesn't map to any phase (e.g., `tasks.md` partly done, src partly written, but no active_context entry), what does it do? Defer to iteration 2; for iteration 1, orchestrator falls back to plain Claude behavior without proactive suggestions.
