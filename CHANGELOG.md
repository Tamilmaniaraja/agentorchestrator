# Changelog

All notable changes to this project will be documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).  
This project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] — 2026-04-12

### Added
- `orchestrator` agent — full setup pipeline: requirements → Agile artifacts → tech spec stubs → team agents → reviewer loop
- `probe` agent — autonomous requirements analysis (subagent mode) and interactive stress-testing (direct mode)
- `team-builder` agent — proposes and generates per-role `.agent.md` files with project-specific responsibilities and tool assignments
- `reviewer` agent — full-project mode and story mode; returns `APPROVED` or `REQUIRES FIXES`
- `project-manager` agent — sprint execution driver; one story per subagent call, state in `progress.md`
- `ProjectRequirement.md` — generic requirements document template
- `README.md` — full usage guide, workflow diagrams, output structure, and troubleshooting
- `CONTRIBUTING.md` — contribution guide and agent file conventions
- `CHANGELOG.md` — this file
- Claude model pinning: `orchestrator` and `reviewer` use `claude-opus-4-6`; `probe`, `team-builder`, and `project-manager` use `claude-sonnet-4-6`
