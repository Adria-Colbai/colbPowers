# Design: Memory Specs Integration for Custom Plugin

## Purpose
To adapt the base superpowers plugin to strictly enforce the reading, writing, and initialization of project-specific documentation stored in `.specs/memory/`. Agents must ensure the project constitution and feature list exist, consult them before planning or implementing, and synchronize changes back to the specs upon review or completion.

## Architecture

The integration spans four core phases of the agent workflow:

### 0. Initialization & Verification (Startup Phase)
Upon starting a conversation or project, the agent must verify the foundational memory documents exist and are populated.
*   **Target Skills**: Create new skills `defining-constitution` and `defining-features`. Update `using-superpowers` to trigger these checks.
*   **Modification**:
    *   Check for the existence and non-emptiness of `.specs/memory/constitution.md` and `.specs/memory/features.md`.
    *   If missing or incomplete, guide the user through defining them using the new skills before proceeding with any other tasks.

### 1. Context Loading (Planning Phase)
Before any design or technical plan is created, the main agent must load global rules and feature definitions.
*   **Target Skills**: `brainstorming`, `writing-plans`
*   **Modification**: Add a mandatory Step 1 to use the Read tool on:
    *   `.specs/memory/constitution.md`
    *   `.specs/memory/features.md`
*   **Constraint**: The agent cannot proceed to the next step until these files are read.
*   **Output Path Change**: All generated specs must be saved to `.specs/memory/docs/specs/` and plans to `.specs/memory/docs/plans/` (replacing the default `docs/superpowers/` paths).
*   **Content Requirement**: Every generated spec and plan MUST explicitly mention which feature(s) from `features.md` it is referring to.

### 2. Context Injection (Subagent Phase)
Subagents are initialized with blank context windows. They must be explicitly directed to read the documentation relevant to their specific tasks.
*   **Target Skills**: `subagent-driven-development`, `dispatching-parallel-agents`
*   **Modification**: The prompt template provided to the `Task` tool must be updated to include instructions for the subagent to read the global constitution and the feature spec for its assigned work (from `.specs/memory/docs/specs/`).

### 3. Specification Synchronization (Review Phase)
As implementations evolve, they may diverge from the original specs due to technical constraints. The specs must remain the source of truth.
*   **Target Skills**: `requesting-code-review`, `finishing-a-development-branch`
*   **Modification**:
    *   Add a "Spec Synchronization" check.
    *   The agent must compare the final implementation against `.specs/memory/features.md` and the specific feature plan/spec in `.specs/memory/docs/`.
    *   If there is a divergence, the agent must update the relevant `.specs/memory/` documents to reflect the shipped code.

## Data Flow
1. **Agent Start** -> Checks `constitution.md` and `features.md`. If missing/empty -> Invokes `defining-constitution`/`defining-features`.
2. **Planning** -> Reads memory docs -> Generates Spec/Plan linking to features -> Saves to `.specs/memory/docs/specs/` or `.specs/memory/docs/plans/`.
3. **Subagent Task** -> Prompt commands reading `constitution.md` and specific spec.
4. **Code Review/Merge** -> Reads implementation and specs -> Updates specs if they differ.

## Error Handling
*   If `.specs/memory/constitution.md` or `.specs/memory/features.md` are missing when the agent attempts to read them during planning, it should halt and invoke the definition skills.
*   If a subagent cannot find the specified spec document, it should abort the task and report the error to the main agent.

## Testing Strategy
1.  **Initialization Test**: Start a session with empty/missing `.specs/memory/` files and verify the agent prompts to define them.
2.  **Read Enforcement Test**: Invoke `brainstorming` and verify the agent's first action is a `read` call to the spec files.
3.  **Path & Content Test**: Verify specs/plans are written to `.specs/memory/docs/` and explicitly reference features.
4.  **Subagent Prompt Test**: Dispatch a mock task and verify the `prompt` payload includes the required `read` directives for the subagent.
5.  **Sync Test**: Provide an implementation that deliberately differs from a spec, invoke `requesting-code-review`, and verify the agent initiates a `write` or `edit` to update the spec file.