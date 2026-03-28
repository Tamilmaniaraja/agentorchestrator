# agentorcustrator

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

Four agents coordinate in sequence:

```
requirements.md
      │
      ▼
 ┌─────────────┐
 │    probe    │  Reads the requirements, resolves ambiguities,
 │             │  returns a dense Requirements Summary
 └──────┬──────┘
        │ Requirements Summary (≤600 tokens)
        ▼
 ┌─────────────┐
 │orchestrator │  Generates all Agile + technical spec artifacts
 │             │  in docs/<project-name>/
 └──────┬──────┘
        │
        ▼
 ┌─────────────┐
 │team-builder │  Proposes team composition, creates one
 │             │  .agent.md per team member
 └──────┬──────┘
        │
        ▼
 ┌─────────────┐
 │  reviewer   │  Reads every generated file, checks completeness,
 │             │  cross-consistency, and methodology correctness
 └──────┬──────┘
        │ REQUIRES FIXES → orchestrator fixes → reviewer re-checks
        │ APPROVED ✅
        ▼
  docs/<project-name>/
```

**Token efficiency built in:** agents pass file paths, not content. Only the Requirements Summary travels inline between agents. Each agent reads files itself using its own tools.

---

## Getting started

### Prerequisites

- [Claude Code](https://claude.ai/code) — the Claude Code CLI or desktop app
- A requirements document written in Markdown

### Usage

1. Clone this repo into your project or copy the `.github/agents/` folder.

2. Write your requirements in a `.md` file (see [`ProjectRequirement.md`](ProjectRequirement.md) as an example).

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
├── .github/agents/
│   └── <project-name>-<role>.agent.md   # one per team member
└── team-roster.md
```

---

## The four agents

### `orchestrator`

The conductor. Drives the full workflow from requirements to approved documentation. Coordinates the other three agents, generates all artifacts itself, and iterates with the reviewer until everything is approved.

**Invoke with:** path to your requirements `.md` file.

### `probe`

A relentless interviewer. When invoked directly, it stress-tests your plan or design by asking one focused question at a time, walking every branch of the decision tree.

When called as a sub-agent, it reads the requirements document autonomously and returns a dense Requirements Summary — no interactive interview.

**Invoke directly with:** a plan, design, or requirements file you want stress-tested.

### `team-builder`

Analyzes the project and proposes a team composition (Project Manager always included; all other roles derived from requirements). Confirms with the user before creating any files. Generates one `.agent.md` per team member with project-specific responsibilities, working agreements, and skills.

**Invoke with:** path to requirements doc + project name (or let the orchestrator call it).

### `reviewer`

A strict documentation reviewer with expertise in PMBOK 7th edition, Scrum, and software engineering best practices. Checks every generated file for completeness, accuracy, cross-consistency, and methodology correctness. Reports issues with specific fixes. Runs in a loop until every issue is resolved.

**Invoke with:** project name + path to requirements doc.

---

## Example output

The [`docs/lumina/`](docs/lumina/) folder contains real output generated from [`ProjectRequirement.md`](ProjectRequirement.md) — a native iPadOS app with on-device AI, cryptographic vaults, and compliance requirements. It includes:

- 8 Epics, 48 User Stories, 476 total story points
- 6 delivery sprints + buffer sprint
- 8 team agent files (iOS Architect, iOS Developer, Security Engineer, ML/CoreML Engineer, UX Designer, QA Engineer, Compliance Engineer, Project Manager)
- 5 ADRs covering the most consequential technical decisions
- Full SwiftData schema with 6 entities and 15-case audit action enum
- Threat model, ML pipeline design, GDPR/CCPA privacy impact assessment

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

See [`ProjectRequirement.md`](ProjectRequirement.md) for a worked example — it is intentionally complex (a native iPadOS app with on-device AI, cryptographic vaults, and compliance requirements). Your requirements doc can be much simpler; the orchestrator scales to the complexity of what you provide.

---

## Agents in your project

The generated team agents under `docs/<project-name>/.github/agents/` are real Claude Code agents. After the orchestrator run, any team member agent can be invoked directly:

```
/agent lumina-ios-developer E2-S1
/agent lumina-security-engineer "review Sprint 1 threat model gaps"
/agent lumina-project-manager "Sprint 2 planning blockers"
```

Each agent already knows the project's tech stack, has working agreements pointing to the generated spec files, and has the right tool permissions for its role (coding roles get `runCommands`; non-coding roles do not).

---

## Troubleshooting

**The reviewer keeps finding issues and won't approve**
The reviewer runs up to 5 iterations before stopping to avoid infinite loops. If issues persist beyond 2–3 passes, they are usually point arithmetic errors in the release roadmap or file path references that don't exist on disk. Check the reviewer's reported fixes carefully — each issue report includes a specific recommended correction.

**The orchestrator asks about PMP every time**
That is by design — the choice is per-run, not saved. Answer `No` to skip PMP and generate only Agile + technical spec artifacts.

**The team composition doesn't match my project**
At Step 5 (team confirmation), you can reject the proposed team and describe what you actually need. `team-builder` will revise the composition before creating any files.

**Generated artifacts reference files that don't exist**
This happens if a technical spec stub was skipped because the condition wasn't met (e.g., no ML/AI detected, so `ml/pipeline-design.md` was not created), but the Definition of Done or an agent Working Agreement still references it. Re-run the reviewer — it catches and reports all dangling path references.

**How long does a run take?**
A typical project takes 5–12 minutes end-to-end depending on complexity and how many reviewer iterations are needed. The LUMINA example (8 epics, 48 stories, 7 tech spec stubs, 8 team agents) took approximately 10 minutes including two reviewer passes.

---

## License

MIT

---

## Contributing

Issues and pull requests welcome. The agents are plain Markdown files — improvements to prompts, new workflow steps, or additional quality checks in the reviewer are all fair game.

If you run the orchestrator on a project and find recurring issues the reviewer doesn't catch, opening an issue with the requirements doc and the reviewer's output is the most useful contribution.
