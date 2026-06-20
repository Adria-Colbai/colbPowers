# colbPowers

> **colbPowers is an extension of the open plugin [superpowers](https://github.com/obra/superpowers).** It builds on superpowers' complete software development workflow and adds project-level memory skills (`defining-constitution` and `defining-features`) so your agent understands *what* you're building before it starts writing code.

colbPowers is a complete software development workflow for your coding agents, built on top of a set of composable "skills" and some initial instructions that make sure your agent uses them.

## How it works

It starts from the moment you fire up your coding agent. As soon as it sees that you're building something, it *doesn't* just jump into trying to write code. Instead, it steps back and asks you what you're really trying to do.

Once it's teased a spec out of the conversation, it shows it to you in chunks short enough to actually read and digest.

After you've signed off on the design, your agent puts together an implementation plan that's clear enough for an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing to follow. It emphasizes true red/green TDD, YAGNI (You Aren't Gonna Need It), and DRY.

Next up, once you say "go", it launches a *subagent-driven-development* process, having agents work through each engineering task, inspecting and reviewing their work, and continuing forward. It's not uncommon for Claude to be able to work autonomously for a couple hours at a time without deviating from the plan you put together.

There's a bunch more to it, but that's the core of the system. And because the skills trigger automatically, you don't need to do anything special. Your coding agent just has colbPowers.


## Installation

### Claude Code

Install directly from the GitHub repository:

```bash
/plugin install git@github.com:Adria-Colbai/colbPowers
```

### OpenCode

OpenCode integration is a bit more manual than other platforms and requires some tinkering to get right.

**1. Install the plugin**

Add the `plugin` key to your `~/.config/opencode/opencode.json`:

```json
"plugin": [
  "colbPowers@git+https://github.com/Adria-Colbai/colbPowers"
]
```

**2. Add code-review-graph (optional)**

Follow the [official code-review-graph installation instructions](https://github.com/colb-ai/code-review-graph) to install the tool, then wire it up to OpenCode by adding an MCP entry to your `opencode.json` (see the full example below).

To make sure the graph actually updates as your agent edits files, you need to add a hook manually. Create a JavaScript plugin file at `~/.config/opencode/plugins/code-review-graph.js` with the following content:

```js
export const CodeReviewGraphPlugin = async ({ $ }) => {
  return {
    "session.created": async () => {
      await $`code-review-graph status`.quiet()
    },

    "tool.execute.after": async (input) => {
      if (["edit", "write", "bash"].includes(input.tool)) {
        await $`code-review-graph update --skip-flows`.quiet()
      }
    },
  }
}
```

This ensures the graph is refreshed every time the agent edits or writes files, keeping structural context accurate throughout a session.

**3. Full example `opencode.json`**

The `permission` block below is just an example — configure it however tightly or loosely you prefer:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "colbPowers@git+https://github.com/Adria-Colbai/colbPowers"
  ],
  "permission": {
    "*": "ask",
    "bash": {
      "*": "ask",
      "grep": "allow",
      "ls": "allow",
      "cat": "allow"
    },
    "read": "allow",
    "write": "ask",
    "todowrite": "allow",
    "Read": "allow"
  },
  "mcp": {
    "code-review-graph": {
      "type": "local",
      "command": [
        "uvx",
        "--python",
        "3.12",
        "code-review-graph",
        "serve"
      ],
      "enabled": true,
      "environment": {}
    }
  }
}
```

### Verify Installation

Start a new session in your chosen platform and ask for something that should trigger a skill (for example, "help me plan this feature" or "let's debug this issue"). The agent should automatically invoke the relevant colbPowers skill.

## The Basic Workflow

1. **defining-constitution** - Activates on new projects. Captures the project's core values, principles, and non-negotiable constraints. Saves to `.specs/memory/constitution.md`.

2. **defining-features** - Activates on new projects. Enumerates the project's key features and scope boundaries. Saves to `.specs/memory/features.md`.

3. **brainstorming** - Activates before writing code. Refines rough ideas through questions, explores alternatives, presents design in sections for validation. Saves design document.

4. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

5. **writing-plans** - Activates with approved design. Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps.

6. **subagent-driven-development** or **executing-plans** - Activates with plan. Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality), or executes in batches with human checkpoints.

7. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR: write failing test, watch it fail, write minimal code, watch it pass, commit. Deletes code written before tests.

8. **requesting-code-review** - Activates between tasks. Reviews against plan, reports issues by severity. Critical issues block progress.

9. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, presents options (merge/PR/keep/discard), cleans up worktree.

**The agent checks for relevant skills before any task.** Mandatory workflows, not suggestions.

## What's Inside

### Skills Library

**Project Memory** *(colbPowers additions)*
- **defining-constitution** - Captures project values, principles, and hard constraints before any implementation
- **defining-features** - Enumerates the project's key features and scope before brainstorming

**Testing**
- **test-driven-development** - RED-GREEN-REFACTOR cycle (includes testing anti-patterns reference)

**Debugging**
- **systematic-debugging** - 4-phase root cause process (includes root-cause-tracing, defense-in-depth, condition-based-waiting techniques)
- **verification-before-completion** - Ensure it's actually fixed

**Collaboration**
- **brainstorming** - Socratic design refinement
- **writing-plans** - Detailed implementation plans
- **executing-plans** - Batch execution with checkpoints
- **dispatching-parallel-agents** - Concurrent subagent workflows
- **requesting-code-review** - Pre-review checklist
- **receiving-code-review** - Responding to feedback
- **using-git-worktrees** - Parallel development branches
- **finishing-a-development-branch** - Merge/PR decision workflow
- **subagent-driven-development** - Fast iteration with two-stage review (spec compliance, then code quality)

**Meta**
- **writing-skills** - Create new skills following best practices (includes testing methodology)
- **using-colbPowers** - Introduction to the skills system

## code-review-graph (Optional but Intended)

colbPowers is designed to work alongside the [code-review-graph](https://github.com/colb-ai/code-review-graph) MCP server. **It is entirely optional — colbPowers works without it.** But it is considered a first-class companion: when present, the agent will prefer graph tools over raw file scanning for codebase exploration, impact analysis, and code review.

### What it adds

Without code-review-graph, the agent reads files and greps for symbols as usual. With it:

- **Codebase exploration** uses the graph instead of Grep/Glob/Read — faster and cheaper in tokens
- **Impact analysis** (`get_impact_radius`) replaces manual import tracing before risky changes
- **Code review** (`detect_changes` + `get_review_context`) gives risk-scored, structurally-aware feedback
- **Relationship queries** (`query_graph`) surface callers, dependents, and test coverage at a glance

The graph auto-updates on file changes via hooks, so it stays in sync as the agent works through a plan.

### Why it's separate

code-review-graph requires its own setup and infrastructure (a running MCP server). Bundling it into colbPowers would add a hard dependency that breaks the zero-dependency spirit of the skills system. Instead, colbPowers detects it at runtime and upgrades its behavior transparently when available.

## Philosophy

- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success

## License

MIT License - see LICENSE file for details
