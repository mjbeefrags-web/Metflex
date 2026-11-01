# CI/CD

No application manifests, lockfiles, test commands, container build context, registry, or deployment environment currently exist. Therefore no passing or executable CI pipeline is claimed. GitHub workflow files belong in `.github/workflows/`; `github-actions/` holds this planning note only.

The release pipeline should, once commands exist, run formatting/lint, type checks, unit/integration tests, dependency and secret scanning, build immutable images, scan images, publish with provenance, and deploy only from protected branches/environments. Use short-lived identity (OIDC) where the platform supports it; do not place long-lived secrets in workflow files. Deployment must consume the exact image digest tested by CI.

Builder must publish canonical commands and QA must define required suites; Planner should assign issue IDs before implementation.
