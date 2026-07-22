# Agent team

To build Mona's Project Pulse dashboard, I am using a custom multi-agent team orchestrated through GitHub Copilot CLI in a GitHub Codespace.

## Custom agents

| Agent | Target model | Responsibility | Agent definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 4.7 (copilot) | Coordinates end-to-end execution, delegates scoped tasks to specialist agents, sequences parallel vs. sequential phases, and verifies the integrated result. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the repository and relevant docs, identifies risks and dependencies, and produces ordered implementation plans with file ownership and validation expectations. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code changes, fixes bugs, keeps behavior deterministic and testable, and validates runnable outcomes for assigned file scopes. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Owns UI/UX quality: accessibility, information hierarchy, responsive behavior, and polished visual styling for the dashboard experience. | `.github/agents/designer.agent.md` |

## Workflow note

The Orchestrator drives the process in Copilot CLI, typically requesting a plan first, then assigning non-overlapping file scopes to Coder and Designer, and finally integrating and reporting outcomes from the Codespace environment.
