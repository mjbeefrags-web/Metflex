# CINEFLOW Engineering Protocol

## Source of truth

The repository is the shared memory for decisions, requirements, research, assignments, and evidence. Every contributor reads `AGENTS.md`, `README.md`, `docs/PROJECT_STATUS.md`, and the documentation for their area before work. Record discoveries and important decisions in the owned repository docs. Reference GitHub issue IDs when available; before that, use clearly provisional planning IDs.

## Roles and ownership

| Role | Owns |
|---|---|
| Engineering Director | Project coherence, milestone gates, assignment coordination, cross-domain review and approval |
| Research Engineer | `docs/research/`, source validation, compatibility and technical limitations |
| Software Architect | `docs/architecture/`, domain boundaries, contracts, ERD, ADRs |
| Technical Planner | `docs/planning/`, requirements, milestones, issue backlog and acceptance criteria |
| Full-Stack Builder | Application feature implementation in `backend/` and `frontend/` after gates |
| Critical Engineer / QA | Independent strategy, threat/failure review, test evidence in `docs/quality/` |
| DevOps Engineer | Infrastructure/deployment implementation and `docs/operations/` |

## Change control

- Domain owners make changes in their owned areas. A change affecting another owner's responsibility requires discussion and a documented approval/decision.
- Do not silently alter an approved ADR, API contract, migration, rights rule, or security boundary. Propose a change with evidence and impact, then record the outcome.
- Assign ownership before parallel work on shared files or contracts.
- Preserve existing work; inspect the tree and working state first. Once Git is initialized, use feature branches and pull requests.
- No external messages or GitHub writes without explicit authorization and an available integration. Record issue definitions locally if GitHub is unavailable.

## Work cycle and gates

1. Confirm milestone, completed evidence, ready/blocked issues, unresolved ADRs, critical path, parallel-safe tasks, and review requirements.
2. Planner selects an issue with dependencies and acceptance criteria.
3. Builder reads relevant code and docs; Research answers unresolved technical details.
4. Architect approves contracts and boundaries for cross-domain work.
5. Builder implements; runs the issue's required tests and records exact commands/results.
6. Critical Engineer independently reviews and verifies; security-sensitive changes require automated evidence.
7. DevOps reviews operational impact; Builder resolves confirmed defects.
8. Critical Engineer verifies fixes; Director approves completion; Planner updates status.

No major feature implementation before the research/architecture gate: stack compatibility, context/container/component diagrams, domain boundaries, ERD, API contracts, media workflow, security, failure analysis, deployment shape, and roadmap.

## Evidence standard

“Done” requires deliverable existence, acceptance criteria met, and appropriate verification evidence. Distinguish planned, implemented, tested, independently verified, and deployed. Never fabricate results. A failed or skipped test is reported as such. Security-sensitive code is not approved on review alone.

## Status reporting

`docs/PROJECT_STATUS.md` records current milestone, work/assignments, completed issue evidence, active/blocked items, architecture/test status, defects, debt, security risks, changed files, dependencies, and next priorities. End each cycle with a concise report of work, files, actual test results, decisions, blockers, and next action.
