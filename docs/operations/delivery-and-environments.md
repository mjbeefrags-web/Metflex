# Delivery, Environments, and Secrets

**Status:** Proposed operating policy; no Git/CI/deployment exists. No GitHub issue IDs are available.

## Repository and CI readiness

The workspace currently has no `.git` metadata or verified GitHub remote. The project owner/DevOps must establish repository ownership, default branch, protected branch rules, CODEOWNERS, issue/PR permissions, and required reviewers before any branch or PR workflow can be used. Do not imply the current files are in a remote repository.

After GitHub hosting is established, proposed CI sequence:

1. Validate formatting, static analysis, frontend/backend type checks, and unit tests using lockfile-pinned dependencies.
2. Run integration tests against ephemeral PostgreSQL/Redis services and migration upgrade from empty database; never connect CI to production.
3. Build frontend/API/worker images once, scan dependencies and images, generate SBOM/provenance, and store immutable artifact digests.
4. Run end-to-end smoke flows against a disposable environment. Media tests use authorized fixtures only; no unlicensed production media in CI.
5. Gate deployment on review, required checks, migration review, security-sensitive approvals, and protected environment authorization.
6. Promote the same immutable artifact to staging then production. Do not rebuild separately per environment.

Use minimal workflow permissions, pin third-party actions to full commit SHAs, avoid `pull_request_target` for untrusted code with secrets, restrict deployment credentials to protected environments, and use OIDC federation instead of long-lived cloud keys when the chosen provider supports it. These are design requirements, not configured controls.

## Environment separation

| Environment | Purpose | Data/credentials |
|---|---|---|
| Developer local | Fast iteration via Compose | Synthetic data, non-production secrets, no production bucket or credentials |
| CI | Reproducible checks | Ephemeral isolated data and short-lived CI credentials; authorized media fixtures only |
| Staging | Production-like release verification | Separate isolated resources, masked/synthetic user data, non-production signing keys and authorized fixture assets |
| Production | User service | Least-privilege secrets, real rights-approved assets only, restricted operator access and audit |

Promote immutable app artifacts; environment configuration may vary but must be validated at startup and reviewed. Database migrations must be backward-compatible with rolling deploys where possible: expand, deploy compatible code, backfill, then contract in a later release. Provide explicit migration timeout/lock review and restore path; never run destructive DDL blindly during deploy.

## Configuration and secret policy

- Keep a documented `.env.example` with names, types, defaults only where safe, and descriptions; do not commit `.env` files containing credentials.
- Validate required settings on startup. Fail closed for missing security-critical values, especially signing keys, DB credentials and trusted proxy configuration.
- Store production secrets in the selected secret manager; inject at runtime. Scope each identity to one service and environment, rotate keys, and log access/audit events.
- Separate session signing, CDN signing, storage access, database, broker and observability credentials. Never reuse local/staging keys in production.
- Do not put secrets in images, build arguments, source maps, CI output, exception details, telemetry attributes, analytics, or command history.
- Scrub passwords, authorization headers, signed playback URLs/tokens, private object keys and personally identifying viewing data from logs.
- Use key rotation with overlap and explicit expiry/revocation behavior. Define incident procedure for suspected exposure before production.

## Release and rollback policy

Use a staged rollout with a release owner, immutable artifact, reviewed change list, migration compatibility note, health/readiness checks, dashboards, and rollback decision threshold. Roll back application artifacts if user-visible error/latency or playback health breaches agreed thresholds; data migrations should be forward-compatible or have a rehearsed restore/recovery procedure. If a schema change is irreversible, block rollout until backup and tested recovery evidence exists. CDN cache purge and signing-key rotation are controlled operational actions with audit trail.

No pipeline, branch protection, secret manager, release, or rollback has been implemented or tested.
