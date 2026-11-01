# GitHub Organization Proposal

No `.github` configuration or established GitHub convention was present. Proposal only; repository owner approval is needed before adoption.

## Labels

- Type: `type:product`, `type:research`, `type:architecture`, `type:feature`, `type:bug`, `type:qa`, `type:devops`, `type:docs`
- Priority: `priority:P0`–`priority:P3` (Product defines rubric)
- State: `state:blocked`, `state:needs-decision`, `state:ready`
- Domain labels only after approved scope.

Use one type and one priority. Labels do not replace owner, milestone or dependency.

## Milestones and issue template

Create M0 when owners/dates are agreed. Add M1–M12 after scope/architecture review; remove out-of-scope tracks. Do not assign unapproved dates. Issue template fields: ID/title, user/business purpose, context/source links, dependencies, owner, requirements, measurable acceptance, QA evidence, security/privacy/rights, documentation, complexity and DoD. Research issues request source/confidence/approver instead of implementation details.

## PR template

- Linked issue and requirement/story
- Summary and scope
- Design/ADR/API/data changes
- Verification commands/results (or reason not run)
- Security/privacy/rights/accessibility considerations
- Migration/deployment/rollback/monitoring impact
- Documentation and reviewer/QA evidence

## Branch, commit and release strategy

Proposed short-lived branches: `feat/CINE-123-description`, `fix/...`, `docs/...`, `research/...`; PR to protected default branch; no direct release-branch pushes. Proposed Conventional Commit with issue, e.g. `feat(catalog): add provenance field (CINE-123)`. Confirm protections with repo owner.

Proposed tags `vMAJOR.MINOR.PATCH`, release notes linked to issues, and promotion through environments with rollback evidence. Versioning/cadence/approval depend on Product/DevOps decisions in CINE-004/005/006. No schedule is inferred.
