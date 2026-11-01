# CINEFLOW Research Index

**Status:** Initial research baseline, 2026-10-31  
**Owner:** Research Engineer  
**Project state:** Repository was empty at research start; no project README, issues, code, dependency lockfiles, or approved architecture records were present. Therefore issue IDs and compatibility against pinned versions are unavailable.

This directory records vendor and standards facts separately from recommendations. It is not evidence that any feature is implemented or tested. Version statements are based on the linked documentation as observed on the date above; revalidate current releases and support windows when dependency versions are selected.

## Documents

- [Technology stack](technology-stack.md)
- [Frontend research](frontend-research.md)
- [Backend research](backend-research.md)
- [Database research](database-research.md)
- [Streaming research](streaming-research.md)
- [Infrastructure research](infrastructure-research.md)
- [Security research](security-research.md)
- [External integrations](external-integrations.md)
- [Compatibility matrix](compatibility-matrix.md)
- [Technical decisions](technical-decisions.md)
- [References](references.md)

## Research rules and confidence

Official product documentation, primary project repositories, IETF RFCs, and standards are preferred. “Verified” means the linked primary source documents the stated behavior; “recommendation” is an engineering judgment for this product. Confidence is high when a normative or current official source directly supports a conclusion, medium when deployment-specific validation remains, and low when licensing, regional distribution, cloud, or workload inputs are unknown.

## Current project status

- Completed: initial research structure and cross-domain baseline.
- Active: detailed implementation questions should be added to the relevant document with source/date and decision status.
- Blockers: no GitHub issue tracker context is available, so no GitHub issue IDs exist; target markets, cloud provider, launch scale, budget, and pinned software versions remain unknown.
- Files changed: this directory only; no application code modified.
- Dependencies: Architect decisions on auth model, deployment target, content rights workflow, and launch regions.
- Next action: Architect reviews [technical decisions](technical-decisions.md), assigns issue IDs, and records any approved deviations.
