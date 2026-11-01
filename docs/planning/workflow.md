# Delivery Workflow and Gates

## Ownership

| Role | Owns | Required review |
|---|---|---|
| Engineering Director | Scope/coherence, milestone acceptance, unresolved cross-domain decisions | Approves milestone exits and risk exceptions |
| Research Engineer | Evidence and technical research | Source/compatibility review when claims affect design |
| Software Architect | Domain boundaries, contracts, architecture and ADRs | Approves architecture/API/schema changes |
| Technical Planner | Backlog, dependencies, acceptance criteria, status | Keeps issue traceability and readiness accurate |
| Full-Stack Builder | Application implementation and author tests | Supplies implementation and actual test evidence |
| Critical Engineer / QA | Independent risk review and verification | Reviews security-sensitive work and independently verifies acceptance |
| DevOps Engineer | Infrastructure and operational reliability | Approves deployment, observability, recovery, resource/cost changes |

## Readiness before work

An issue is Ready only if it has a unique tracker ID, owner, user/business outcome, acceptance criteria, dependencies, chosen API/data contract, relevant research/ADR, testing plan, documentation impact, and defined security/privacy/operational implications. If a decision is unresolved, either split an investigation issue or mark the implementation blocked. The Builder does not silently settle Architect-owned decisions.

## Change process

1. Planner selects a ready issue and records ownership/dependencies.
2. Builder reads root instructions, project status, relevant architecture/research and issue criteria before editing.
3. Builder works on an issue branch and keeps changes scoped. Never overwrite unexplained work.
4. Builder adds tests at the relevant unit/integration/contract/E2E level and reports commands plus exact results; do not state tests passed unless actually run.
5. Architect reviews contracts/data/domain changes; DevOps reviews infrastructure; Critical Engineer independently reviews security-sensitive changes and acceptance evidence.
6. Resolve findings, update docs and status, link evidence/PR, then Director accepts completion.

## Merge and release gates

- **Architecture gate:** no major feature implementation until technology compatibility, context/container/component views, ERD, API contracts, domain boundaries, media pipeline, security, failure modes, deployment plan, quality plan and initial roadmap are reviewed and approved.
- **Rights gate:** no title is published or playable without approved, scoped rights evidence for that asset, territory, term and use. New ingestion remains private pending validation and review.
- **Security gate:** authentication, authorization, file/URL ingest, signing, admin and rights controls require independent review and automated negative tests before acceptance.
- **Migration gate:** migration applies to clean supported DB and upgrade strategy is recorded; production schema is not created ad hoc.
- **Release gate:** automated checks, staging smoke, migration plan, rollback path, alerts, backup/restore evidence, and deployment owner must be available.

## Status reporting

Keep `docs/PROJECT_STATUS.md` as the live project report. For each milestone, report current state and evidence, completed/active/blocked issues, agent owners, accepted decisions/ADRs, test commands and results, known defects, technical debt/security risks, dependencies and next priority. Use “not run” when no test was run. Do not equate activity, merged code, or a passing unit test with end-to-end acceptance.

## Parallel work rules

Parallelize only when file ownership and API/data contracts do not overlap. Planner names the file owner in the issue. Shared contract changes go through Architect first. QA remains independent of Builder implementation. Infrastructure can proceed alongside other work only within an approved topology and with DevOps ownership.
