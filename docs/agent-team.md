# Agent team

To build Mona's Project Pulse dashboard, I am using a custom team of four agents, each defined under `.github/agents/` in this repository, orchestrated with GitHub Copilot CLI in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases, assigns non-overlapping file scopes, runs independent work in parallel, and reports progress. Does not implement or commit changes itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the codebase and documentation to produce an ordered implementation plan, including file assignments, dependencies, parallelizable vs. sequential work, edge cases, and open questions. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX for Project Pulse, delivering a polished dashboard with project cards, status badges, clear priority treatment, and responsive layout, using deterministic CSS hooks such as `.dashboard` and `.project-card`.
- **Definition:** `.github/agents/designer.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements application logic within the file scope assigned by the Orchestrator, and creates supporting runnable-app files such as `.vscode/launch.json` for Project Pulse. Validates changes before reporting completion.
- **Definition:** `.github/agents/coder.agent.md`

All work is orchestrated using GitHub Copilot CLI running in a Codespace; none of the agents stage, commit, or push changes themselves.

## How the team builds Project Pulse together

1. I ask the **Orchestrator** to build the Project Pulse dashboard.
2. The Orchestrator asks the **Planner** to research the repo and produce an implementation plan with file assignments, dependencies, and what can run in parallel vs. sequentially.
3. The Orchestrator parses that plan into phases and delegates each piece to a specialist with an explicit file scope:
   - **Designer** builds the dashboard UI/UX (cards, badges, layout, CSS hooks) in design-owned files.
   - **Coder** implements the app logic and, once markup/data needs are clear, adds supporting files like `.vscode/launch.json` so the app is easy to run.
4. Phases with non-overlapping file scopes run in parallel; phases that depend on each other's output (e.g., Coder needing Designer's markup) run sequentially.
5. The Orchestrator verifies the integrated result hangs together and reports progress and the final outcome back to me after each phase.
6. I remain in control of all git operations (stage/commit/push) via Copilot CLI prompts — no agent commits on its own.
