# Contributing to agentorchestrator

Thank you for your interest in contributing. This project is a set of Claude Code agent definition files — plain Markdown — so the bar to contribute is low.

---

## Ways to contribute

- **Bug reports** — something in the workflow produces wrong or incomplete output
- **Prompt improvements** — better phrasing, clearer instructions, tighter quality checks in any agent
- **New quality checks** — additional things the reviewer should catch
- **Workflow enhancements** — new steps or modes that improve the output
- **Documentation** — clearer explanations in the README or agent files

---

## Before you open a PR

1. **Run the orchestrator on a real requirements doc** to verify your changes produce correct output end-to-end. The agents are prompt files — the only way to validate them is to run them.

2. **Check for regressions** — if you change the reviewer, make sure it still approves a clean run. If you change the orchestrator workflow, make sure all 10 steps still execute correctly.

3. **Keep changes focused** — one concern per PR. A prompt fix and a new workflow step should be separate PRs.

---

## Agent file conventions

All agents live in `agents/` and follow the Claude Code `.agent.md` format:

```
---
name: <lowercase-hyphenated>
description: <one sentence — used by Claude Code to match the agent to user intent>
argument-hint: <concrete example invocation with real argument format>
agents: [...]       # sub-agents this agent may invoke
tools: [...]        # tools this agent may use
---

<system prompt>
```

When editing an agent file:

- **`description`** — must be specific enough that Claude Code selects this agent and not another for the right task. Generic descriptions cause routing failures.
- **`argument-hint`** — use `my-project` and generic story IDs (e.g. `E1-S1`) as placeholders. Never use project-specific names.
- **Placeholders in templates** — any `<placeholder>` text inside generated agent file templates (inside backtick blocks) must remain as-is. Only replace literal example values like `"lumina"` with `"my-project"`.
- **Token discipline** — agents pass file paths, not file content. Do not add steps that paste document content inline between agents.

---

## Submitting a pull request

1. Fork the repository and create a branch from `main`.
2. Make your changes.
3. Open a pull request against `main`. Fill in the PR template.
4. If you ran the orchestrator to validate, mention the type of requirements doc you tested with (complexity level, domain) — you don't need to share the doc itself.

---

## Reporting bugs

Use the [Bug Report](.github/ISSUE_TEMPLATE/bug_report.yml) issue template. The most useful bug reports include:

- Which agent produced the wrong output
- The reviewer's reported issues (if applicable)
- Whether the problem is consistent or intermittent

You do not need to share your requirements document if it is confidential — a description of its complexity and domain is enough.

---

## Questions

Open a [Discussion](../../discussions) or a plain issue if something is unclear. There are no dumb questions about prompt engineering.
