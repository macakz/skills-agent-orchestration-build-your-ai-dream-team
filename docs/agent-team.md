# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent custom team defined under `.github/agents/`, orchestrated with the GitHub Copilot CLI running in a Codespace.

- **Orchestrator** — Model: Claude Opus 4.7 (copilot). Coordinates the Planner, Coder, and Designer, breaking the request into non-overlapping file scopes, sequencing dependent work, and reporting the integrated result. Defined in `.github/agents/orchestrator.agent.md`.
- **Planner** — Model: Claude Opus 4.7 (copilot). Researches the repository and requirements, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, edge cases, and validation expectations, without writing code. Defined in `.github/agents/planner.agent.md`.
- **Coder** — Model: GPT-5.5 (copilot). Implements the dashboard's code within the file scope assigned by the Orchestrator, including runnable app support like `.vscode/launch.json` for Project Pulse. Defined in `.github/agents/coder.agent.md`.
- **Designer** — Model: Gemini 3.1 Pro (copilot). Owns UI/UX, accessibility, information hierarchy, and visual design for the dashboard, ensuring a polished look with project cards, status badges, and responsive layout. Defined in `.github/agents/designer.agent.md`.

All orchestration for this work happens through the GitHub Copilot CLI in a Codespace, with each agent staying within its assigned file scope and the learner retaining control over all git operations.
