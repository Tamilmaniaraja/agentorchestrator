# agentorchestrator

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Claude](https://img.shields.io/badge/Claude-Opus%204.6%20%7C%20Sonnet%204.6-orange)

A Claude Code multi-agent system that transforms a requirements document into a full project documentation suite — Agile artifacts, technical specification stubs, a development team of specialized agents, and optional PMP documents — all reviewed and self-corrected before delivery.

---

## What it does

Point it at a requirements `.md` file. It produces:

**Agile artifacts**
- Product Backlog (Epics → User Stories with story points)
- Definition of Done
- Sprint Plan (Sprint 1, with task breakdowns)
- Release Roadmap (multi-sprint)
- Acceptance Criteria (Gherkin Given/When/Then, min. 3 scenarios per story)

**Technical specification stubs** _(generated based on what the project actually needs)_
- Architecture Decision Records
- Test Strategy
- Coding Standards
- Threat Model _(if security/auth is present)_
- Data Schema _(if persistence is present)_
- ML Pipeline Design _(if ML/AI is present)_
- Privacy Impact Assessment _(if regulated data is present)_

**Development team**
- One `.agent.md` file per team member, pre-configured with responsibilities, working agreements, tool access, and skills — ready to use in Claude Code

**Optionally**
- PMP/PMBOK artifacts: Project Charter, WBS, Risk Register, Stakeholder Register, Communication Plan, Project Management Plan

Everything is reviewed by a strict reviewer agent and iterated until approved.

---

## How it works

The system runs in two phases — **Setup** and **Execution** — each driven by a different agent.

### Phase 1 — Setup (`orchestrator`)

The `orchestrator` is the controller. You invoke it once; it coordinates all other agents:

```
You invoke: orchestrator ProjectRequirement.md
                    │
                    ▼
           ┌─────────────────┐
           │   orchestrator  │  Controls the full setup pipeline
           └──┬──────────────┘
              │
              │ 1. invokes probe ──────────────────────────────────┐
              │                                                     │
              │          ┌─────────────┐                           │
              │          │    probe    │  Reads requirements,       │
              │          │             │  returns Requirements      │
              │          │             │  Summary (≤600 tokens)     │
              │          └──────┬──────┘                           │
              │                 │ summary returned to orchestrator  │
              │◄────────────────┘                                  │
              │                                                     │
              │ 2. generates all Agile + tech spec artifacts        │
              │    in docs/<project-name>/                          │
              │                                                     │
              │ 3. invokes team-builder ──────────────────────────┐ │
              │                                                   │ │
              │          ┌─────────────┐                         │ │
              │          │team-builder │  Proposes team, creates  │ │
              │          │             │  one .agent.md per role  │ │
              │          └──────┬──────┘                         │ │
              │                 │ agent paths returned            │ │
              │◄────────────────┘                                │ │
              │                                                   │ │
              │ 4. invokes reviewer ──────────────────────────────┘ │
              │                                                      │
              │          ┌─────────────┐                            │
              │          │  reviewer   │  Checks every file for     │
              │          │             │  completeness, arithmetic,  │
              │          │             │  and cross-consistency      │
              │          └──────┬──────┘                            │
              │                 │                                    │
              │    REQUIRES FIXES → orchestrator fixes → re-review  │
              │    APPROVED ✅                                       │
              │◄────────────────┘                                   │
              │
              ▼
       docs/<project-name>/  ← all output lands here
```

### Phase 2 — Execution (`project-manager`)

Once setup is done, the `project-manager` drives the sprint story by story:

```
project-manager "my-project sprint 1"
        │
        ▼
 Read progress.md ──── created on first run
        │
        ▼
 Next ⏳ story → write sprint contract
 docs/<project-name>/sprint-contracts/<story-id>.md
        │
        ▼
 Invoke team agent ──── ONE story per subagent call
        │
        ▼
 Invoke reviewer ──── story-mode: this story only
        │
   ┌────┴────┐
APPROVED   REQUIRES FIXES → re-invoke team agent (max 3 attempts)
   │
   ✅ Update progress.md → next story
        │
        ▼ all stories done
 Generate sprint-<n>-review.md
```

**Key design principle:** Each story runs as a focused micro-task subagent so context never fills up across a long sprint. The project-manager reads `progress.md` fresh at every step — state lives in files, not in context.

---

## Getting started

### Prerequisites

- [Claude Code](https://claude.ai/code) — the Claude Code CLI or desktop app
- A requirements document written in Markdown

### Claude model versions

Each agent is pinned to a specific Claude model in its frontmatter:

| Agent | Model | Reason |
|---|---|---|
| `orchestrator` | `claude-opus-4-6` | Drives the full pipeline; needs the strongest reasoning |
| `reviewer` | `claude-opus-4-6` | Strict quality checks; must catch subtle cross-file inconsistencies |
| `probe` | `claude-sonnet-4-6` | Structured output; fast and accurate for analysis tasks |
| `team-builder` | `claude-sonnet-4-6` | Template-based file generation |
| `project-manager` | `claude-sonnet-4-6` | Coordination and state management |

To override the model for any agent, change the `model:` field in its frontmatter. See the [Claude model IDs](https://docs.anthropic.com/en/docs/about-claude/models) for available options.

### Usage — Claude Code

1. Copy the `agents/claude/` folder into your project's `.github/agents/` directory.

   ```bash
   cp -r agents/claude/ your-project/.github/agents/
   ```

2. Write your requirements in a `.md` file (see [`ProjectRequirement.md`](ProjectRequirement.md) as a template).

3. In Claude Code, invoke the orchestrator:

   ```
   /agent orchestrator ProjectRequirement.md
   ```

   Or simply describe what you want:

   > "Run the orchestrator agent with ProjectRequirement.md"

4. Answer the two prompts:
   - Confirm or adjust the inferred project name
   - Choose whether to generate PMP documents in addition to Agile artifacts

5. Confirm the proposed team composition.

6. Wait for the reviewer to return **APPROVED**. All output lands in `docs/<project-name>/`.

### Usage — GitHub Copilot

1. Copy the `agents/copilot/` folder into your project's `.github/agents/` directory.

   ```bash
   cp -r agents/copilot/ your-project/.github/agents/
   ```

2. Write your requirements in a `.md` file.

3. In VS Code Copilot Chat (agent mode), invoke the orchestrator:

   ```
   @orchestrator my-requirements.md
   ```

4. Answer the two prompts:
   - Confirm or adjust the inferred project name
   - Choose whether to generate PMP documents

5. Confirm the proposed team composition.

6. Wait for the self-review to complete. All output lands in `docs/<project-name>/`.

---

## Output structure

```
docs/<project-name>/
├── agile/
│   ├── product-backlog.md
│   ├── definition-of-done.md
│   ├── sprint-plan.md
│   ├── release-roadmap.md
│   └── acceptance-criteria.md
├── architecture/
│   └── architecture-decision-records.md
├── engineering/
│   ├── test-strategy.md
│   └── coding-standards.md
├── security/
│   └── threat-model.md          # if security/auth present
├── data/
│   └── schema.md                # if persistence present
├── ml/
│   └── pipeline-design.md       # if ML/AI present
├── compliance/
│   └── pia-template.md          # if regulated domain present
├── pmp/                         # if PMP selected
│   ├── project-charter.md
│   ├── wbs.md
│   ├── risk-register.md
│   ├── stakeholder-register.md
│   ├── communication-plan.md
│   └── project-management-plan.md
├── agents/
│   └── <project-name>-<role>.agent.md   # one per team member (copy to .github/agents/ to use)
└── team-roster.md
```

---

## The five agents

### `orchestrator`

**Phase 1 — Setup.** Transforms a requirements document into all project documentation. Coordinates probe → artifact generation → team-builder → reviewer loop until everything is approved. Run once per project.

**Invoke with:** path to your requirements `.md` file.

### `project-manager`

**Phase 2 — Execution.** The entry point for running the project after setup. Reads the sprint plan and `team-roster.md` to discover available team agents, then drives the sprint story by story:

1. Writes a focused **sprint contract** for the next story (`docs/<project-name>/sprint-contracts/<story-id>.md`) — exact deliverables, acceptance criteria, and relevant spec paths.
2. Delegates to the correct **team agent** as a micro-task subagent (one story, one subagent call — context never accumulates).
3. Independently verifies output with the **reviewer** in story mode (checks only this story's acceptance criteria and DoD).
4. Updates `progress.md` on approval; retries up to 3 times on failure before escalating as blocked.

Tracks everything in `docs/<project-name>/progress.md`. Generates a sprint review document when all stories are done.

**Invoke with:** project name and sprint number — e.g. `"my-project sprint 1"`. Re-invoke at any time to resume from where `progress.md` left off.

### `probe`

A relentless interviewer. When invoked directly, it stress-tests your plan or design by asking one focused question at a time, walking every branch of the decision tree.

When called as a sub-agent, it reads the requirements document autonomously and returns a dense Requirements Summary — no interactive interview.

**Invoke directly with:** a plan, design, or requirements file you want stress-tested.

### `team-builder`

Analyzes the project and proposes a team composition (Project Manager always included; all other roles derived from requirements). Confirms with the user before creating any files. Generates one `.agent.md` per team member with project-specific responsibilities, working agreements, and skills.

**Invoke with:** path to requirements doc + project name (or let the orchestrator call it).

### `reviewer`

A strict documentation reviewer with expertise in PMBOK 7th edition, Scrum, and software engineering best practices. Invoked automatically by both the orchestrator and project-manager — not typically invoked directly by the user. Operates in two modes:

- **Full-project mode** (invoked by orchestrator): checks every generated file for completeness, accuracy, point arithmetic, naming consistency, and cross-consistency. Runs in a loop until all issues are resolved.
- **Story mode** (invoked by project-manager): checks only the deliverables for a single story against its sprint contract, acceptance criteria, and relevant DoD items.

Always returns one of two exact verdicts: `APPROVED` or `REQUIRES FIXES`.

**Invoke with:** project name — e.g. `"my-project"` for full-project mode, or `"my-project E2-S1"` for story mode.

---

## Example output

Run the orchestrator on your own requirements document to generate your project's full documentation suite. Output lands in `docs/<project-name>/` and will include all Agile artifacts, technical spec stubs, and team agent files appropriate for your project's complexity.

---

## Writing good requirements

The better your requirements document, the better the output. The `probe` agent resolves ambiguities autonomously, but clear upfront requirements produce more accurate artifacts.

Effective requirements documents include:

- **Problem statement** — what user problem is being solved and for whom
- **Core features** — what the product must do (not how)
- **Tech stack / platform** — language, framework, target devices or environments
- **Non-functional requirements** — performance targets, security constraints, compliance obligations
- **Explicit constraints** — what is out of scope, what must not happen
- **Scale and timeline** — approximate team size and delivery horizon

Your requirements doc can be as simple or complex as your project demands — the orchestrator scales to match.

---

## Agents in your project

The generated team agents are written to `docs/<project-name>/agents/`. To use them, copy them into your project's `.github/agents/` directory — Claude Code discovers agents from there automatically. They are real Claude Code agents. After the orchestrator run, any team member agent can be invoked directly:

```
/agent your-project-ios-developer E2-S1
/agent your-project-security-engineer "review Sprint 1 threat model gaps"
/agent your-project-project-manager "Sprint 2 planning blockers"
```

Each agent already knows the project's tech stack, has working agreements pointing to the generated spec files, and has the right tool permissions for its role (coding roles get `runCommands`; non-coding roles do not).

---

## Troubleshooting

**The reviewer keeps finding issues and won't approve**
The orchestrator runs up to 5 reviewer iterations before stopping to avoid infinite loops. If issues persist beyond 2–3 passes, they are usually point arithmetic errors in the release roadmap or file path references that don't exist on disk. Check the reviewer's reported fixes carefully — each issue report includes a specific recommended correction. After 5 failed passes the orchestrator stops and escalates all remaining issues to you for manual resolution.

**The orchestrator asks about PMP every time**
That is by design — the choice is per-run, not saved. Answer `No` to skip PMP and generate only Agile + technical spec artifacts.

**The team composition doesn't match my project**
When the orchestrator presents the proposed team for confirmation (before any files are created), you can reject it and describe what you actually need. `team-builder` will revise the composition before creating any files.

**Generated artifacts reference files that don't exist**
This happens if a technical spec stub was skipped because the condition wasn't met (e.g., no ML/AI detected, so `ml/pipeline-design.md` was not created), but the Definition of Done or an agent Working Agreement still references it. Re-run the reviewer — it catches and reports all dangling path references.

**How long does a run take?**
A typical project takes 5–12 minutes end-to-end depending on complexity and how many reviewer iterations are needed. The example project (8 epics, 48 stories, 7 tech spec stubs, 8 team agents) took approximately 10 minutes including two reviewer passes.

---

## License

MIT

---

## Contributing

Issues and pull requests welcome. The agents are plain Markdown files — improvements to prompts, new workflow steps, or additional quality checks in the reviewer are all fair game.

If you run the orchestrator on a project and find recurring issues the reviewer doesn't catch, opening an issue with the requirements doc and the reviewer's output is the most useful contribution.
