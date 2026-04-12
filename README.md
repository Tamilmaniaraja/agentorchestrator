# agentorchestrator

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Claude](https://img.shields.io/badge/Claude-Opus%204.6%20%7C%20Sonnet%204.6-orange)
![Copilot](https://img.shields.io/badge/GitHub%20Copilot-supported-blue)

A multi-agent system for **Claude Code** and **GitHub Copilot** that transforms a requirements document into a full project documentation suite — Agile artifacts, technical specification stubs, a development team of specialized agents, and optional PMP documents — all reviewed and self-corrected before delivery.

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
- One agent file per team member, pre-configured with responsibilities, tool access, and skills — ready to use in Claude Code or GitHub Copilot

**Optionally**
- PMP/PMBOK artifacts: Project Charter, WBS, Risk Register, Stakeholder Register, Communication Plan, Project Management Plan

Everything is reviewed by a strict reviewer agent and iterated until approved.

---

## How it works

The system runs in two phases — **Setup** and **Execution** — each driven by a different agent.

### Phase 1 — Setup (`orchestrator`)

The `orchestrator` is the controller. You invoke it once; it coordinates all other agents:

```
You invoke: orchestrator requirements.md
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
              │          │             │  one agent file per role │ │
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

## Repository structure

```
agentorchestrator/
├── agents/
│   ├── claude/               ← Claude Code agents (.md)
│   │   ├── orchestrator.md
│   │   ├── probe.md
│   │   ├── team-builder.md
│   │   ├── reviewer.md
│   │   └── project-manager.md
│   └── copilot/              ← GitHub Copilot agents (.agent.md)
│       ├── orchestrator.agent.md
│       ├── probe.agent.md
│       ├── team-builder.agent.md
│       ├── reviewer.agent.md
│       └── project-manager.agent.md
├── ProjectRequirement.md     ← requirements document template
├── CHANGELOG.md
├── CONTRIBUTING.md
└── README.md
```

Both sets of agents implement the same workflow and sub-agent orchestration pattern. The differences are in file format and tool naming — see [Agent platform differences](#agent-platform-differences) below.

---

## Getting started

### Prerequisites

**Claude Code**
- [Claude Code](https://claude.ai/code) — CLI or desktop app

**GitHub Copilot**
- [VS Code](https://code.visualstudio.com/) with the [GitHub Copilot extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
- GitHub Copilot subscription with agent mode enabled

Both require a requirements document written in Markdown.

---

### Usage — Claude Code

1. Copy the `agents/claude/` folder into your project's `.github/agents/` directory:

   ```bash
   cp -r agents/claude/ your-project/.github/agents/
   ```

2. Write your requirements in a `.md` file (use [`ProjectRequirement.md`](ProjectRequirement.md) as a template).

3. In Claude Code, invoke the orchestrator:

   ```
   /agent orchestrator requirements.md
   ```

   Or describe what you want:

   > "Run the orchestrator agent with requirements.md"

4. Answer the two prompts:
   - Confirm or adjust the inferred project name
   - Choose whether to generate PMP documents in addition to Agile artifacts

5. Confirm the proposed team composition.

6. Wait for the reviewer to return **APPROVED**. All output lands in `docs/<project-name>/`.

7. To begin sprint execution:

   ```
   /agent project-manager my-project sprint 1
   ```

---

### Usage — GitHub Copilot

1. Copy the `agents/copilot/` folder into your project's `.github/agents/` directory:

   ```bash
   cp -r agents/copilot/ your-project/.github/agents/
   ```

2. Write your requirements in a `.md` file.

3. In VS Code Copilot Chat (agent mode), invoke the orchestrator:

   ```
   @orchestrator requirements.md
   ```

4. Answer the two prompts:
   - Confirm or adjust the inferred project name
   - Choose whether to generate PMP documents

5. Confirm the proposed team composition.

6. Wait for the reviewer to return **APPROVED**. All output lands in `docs/<project-name>/`.

7. To begin sprint execution:

   ```
   @project-manager my-project sprint 1
   ```

---

## Model versions

Each agent is pinned to a specific Claude model. The same model assignments apply to both the Claude Code and GitHub Copilot agent sets:

| Agent | Model | Reason |
|---|---|---|
| `orchestrator` | `claude-opus-4-6` | Drives the full pipeline; needs the strongest reasoning |
| `reviewer` | `claude-opus-4-6` | Strict quality checks; must catch subtle cross-file inconsistencies |
| `probe` | `claude-sonnet-4-6` | Structured output; fast and accurate for requirements analysis |
| `team-builder` | `claude-sonnet-4-6` | Template-based agent file generation |
| `project-manager` | `claude-sonnet-4-6` | Coordination and sprint state management |

To override the model for any agent, change the `model:` field in its frontmatter.

---

## Output structure

```
docs/<project-name>/
├── agile/
│   ├── product-backlog.md
│   ├── definition-of-done.md
│   ├── sprint-plan.md
│   ├── release-roadmap.md
│   ├── acceptance-criteria.md
│   └── sprint-<n>-review.md    # generated after each sprint
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
│   └── <project-name>-<role>.agent.md   # one per team member
├── sprint-contracts/
│   └── <story-id>.md            # one per story, written by project-manager
├── progress.md                  # live sprint tracking
└── team-roster.md
```

---

## The five agents

### `orchestrator`

**Phase 1 — Setup.** Transforms a requirements document into all project documentation. Coordinates probe → artifact generation → team-builder → reviewer loop until everything is approved. Run once per project.

**Invoke with:** path to your requirements `.md` file.

### `project-manager`

**Phase 2 — Execution.** Drives the sprint story by story:

1. Writes a focused **sprint contract** for the next story — exact deliverables, acceptance criteria, and relevant spec paths.
2. Delegates to the correct **team agent** as a micro-task subagent (one story, one subagent call).
3. Independently verifies output with the **reviewer** in story mode.
4. Updates `progress.md` on approval; retries up to 3 times on failure before escalating as blocked.

**Invoke with:** project name and sprint number — e.g. `"my-project sprint 1"`. Re-invoke at any time to resume from where `progress.md` left off.

### `probe`

A relentless interviewer. When invoked directly, stress-tests your plan or design by asking one focused question at a time, walking every branch of the decision tree.

When called as a sub-agent by the orchestrator or team-builder, it reads the requirements document autonomously and returns a dense Requirements Summary — no interactive interview.

**Invoke directly with:** a plan, design, or requirements file you want stress-tested.

### `team-builder`

Analyzes the project and proposes a team composition (Project Manager always included; all other roles derived from requirements). Confirms with the user before creating any files. Generates one agent file per team member with project-specific responsibilities and tool access.

**Invoke with:** path to requirements doc + project name (or let the orchestrator call it).

### `reviewer`

A strict documentation reviewer with expertise in PMBOK 7th edition, Scrum, and software engineering best practices. Invoked automatically by both the orchestrator and project-manager. Operates in two modes:

- **Full-project mode** (invoked by orchestrator): checks every generated file for completeness, accuracy, point arithmetic, naming consistency, and cross-consistency. Runs in a loop until all issues are resolved.
- **Story mode** (invoked by project-manager): checks only the deliverables for a single story against its sprint contract, acceptance criteria, and relevant DoD items.

Always returns one of two exact verdicts: `APPROVED` or `REQUIRES FIXES`.

**Invoke with:** project name — e.g. `"my-project"` for full-project mode, or `"my-project E2-S1"` for story mode.

---

## Agent platform differences

Both agent sets implement the same workflow. The differences are tooling syntax and file format only:

| | Claude Code (`agents/claude/`) | GitHub Copilot (`agents/copilot/`) |
|---|---|---|
| **File extension** | `.md` | `.agent.md` |
| **Install path** | `.github/agents/` | `.github/agents/` |
| **Sub-agent field** | `agents: [...]` | `agents: [...]` |
| **Sub-agent tool** | built-in | requires `"agent"` in tools list |
| **Codebase search** | `"search"`, `"codebase"` | `"search/codebase"` |
| **File editing** | `"editFiles"` | `"edit"` |
| **Run commands** | `"runCommands"` | `"runCommand"` |
| **Fetch URLs** | `"fetch"` | `"web/fetch"` |
| **Workflow handoffs** | — | `handoffs:` field supported |

---

## Using generated team agents

The generated team agents are written to `docs/<project-name>/agents/`. Copy them into your project's `.github/agents/` directory to use them directly:

```bash
cp -r docs/my-project/agents/ .github/agents/
```

**Claude Code:**
```
/agent my-project-backend-developer E2-S1
/agent my-project-security-engineer "review Sprint 1 threat model gaps"
/agent my-project-project-manager "Sprint 2 planning blockers"
```

**GitHub Copilot:**
```
@my-project-backend-developer E2-S1
@my-project-security-engineer review Sprint 1 threat model gaps
```

Each agent knows the project's tech stack, references the generated spec files, and has the right tool permissions for its role (coding roles get `runCommand`; non-coding roles do not).

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

Use [`ProjectRequirement.md`](ProjectRequirement.md) as a starting template.

---

## Troubleshooting

**The reviewer keeps finding issues and won't approve**
The orchestrator runs up to 5 reviewer iterations before stopping. If issues persist beyond 2–3 passes, they are usually point arithmetic errors in the release roadmap or file path references that don't exist on disk. Each issue report includes a specific recommended correction. After 5 failed passes the orchestrator stops and escalates all remaining issues to you for manual resolution.

**The orchestrator asks about PMP every time**
That is by design — the choice is per-run, not saved. Answer `No` to skip PMP and generate only Agile + technical spec artifacts.

**The team composition doesn't match my project**
When the orchestrator presents the proposed team for confirmation (before any files are created), you can reject it and describe what you actually need. `team-builder` will revise the composition before creating any files.

**Generated artifacts reference files that don't exist**
This happens if a technical spec stub was skipped because the condition wasn't met (e.g., no ML/AI detected, so `ml/pipeline-design.md` was not created), but the Definition of Done or an agent Working Agreement still references it. Re-run the reviewer — it catches and reports all dangling path references.

---

## Versioning

This project follows [Semantic Versioning](https://semver.org/). See [CHANGELOG.md](CHANGELOG.md) for the full release history.

| Change type | Version bump |
|---|---|
| Breaking change to workflow or output format | `MAJOR` |
| New agent, new step, new capability | `MINOR` |
| Bug fix, prompt improvement, typo | `PATCH` |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on submitting issues and pull requests.

The agents are plain Markdown files — improvements to prompts, new workflow steps, or additional quality checks in the reviewer are all fair game. If you run the orchestrator on a project and find recurring issues the reviewer doesn't catch, opening an issue with the requirements doc and the reviewer's output is the most useful contribution.

---

## License

MIT
