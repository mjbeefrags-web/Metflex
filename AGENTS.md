# CINEFLOW Repository Instructions

## Source of truth

- The repository is shared project memory. Read this file, the root README, `docs/ENGINEERING_PROTOCOL.md`, the current `docs/PROJECT_STATUS.md`, and relevant domain documentation before changing work.
- Important technical decisions belong in `docs/PROJECT_DECISIONS.md` or an approved ADR under `docs/architecture/adr/`.
- Do not rely on conversation-only context. Link issue IDs when they exist; until GitHub is connected, use clearly labeled local planning IDs.

## Ownership

- Engineering Director coordinates milestones and approves cross-domain changes.
- Research Engineer owns `docs/research/`.
- Architect owns `docs/architecture/` and ADRs.
- Planner owns `docs/planning/`.
- Critical Engineer/QA owns `docs/quality/` and independent verification.
- DevOps owns `infrastructure/`, `docs/operations/`, and CI/deployment configuration.
- Builder alone implements application features in `backend/` and `frontend/`.
- Avoid concurrent edits to the same files; coordinate shared API contracts before parallel implementation.

## Engineering rules

- Do not implement features before the relevant architecture/contract gate is approved.
- Do not claim tests, integrations, deployments, or issue publication without evidence.
- Do not place secrets in source control. Use centralized configuration and validation.
- Do not add dependencies or infrastructure without a documented need and owner.
- Preserve existing work; inspect before replacing. Use branches/PRs once Git is initialized.
- Keep authorized media sources and rights evidence explicit. Never scrape unauthorized streams, redistribute unlicensed works, or circumvent DRM.
- Documentation changes must identify whether content is verified fact, recommendation, assumption, or approved decision.

## Current repository limitation

At kickoff inspection (2026-10-31, Africa/Cairo), this directory contained the research baseline only; there was no `.git`, code, manifests, or tests. Establish Git/GitHub and CI before implementation work that depends on branch/PR workflow.
