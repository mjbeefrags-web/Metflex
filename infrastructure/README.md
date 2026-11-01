# Infrastructure baseline

**Status: bootstrap only.** The repository now has a project README and draft architecture/research documents, but no application source, package manifests, GitHub issue references, or approved deployment architecture. This baseline provides local PostgreSQL and Redis for future integration; it is not a deployable CINEFLOW release.

## Current services

`compose.yaml` specifies PostgreSQL and Redis image versions, keeps published ports on loopback, adds health checks and persistent development volumes, and places dependencies on an internal network. PostgreSQL requires `POSTGRES_PASSWORD` to be supplied by the developer at runtime; no password fallback is committed. Redis has no password/ACL, so the scaffold is for disposable single-developer local use only and must not be exposed or reused outside that scope.

Backend, frontend, Celery, Nginx, production TLS, monitoring, CI image builds, and deployment automation are intentionally not wired to invented application interfaces. Add them after the Architect and Builder publish verified stack, ports, health endpoints, migration commands, asset/media policy, and deployment topology.

## Start local dependencies

From the repository root, set a local password in PowerShell and start the dependencies:

```powershell
$securePassword = Read-Host 'Enter a local-only PostgreSQL password' -AsSecureString
$env:POSTGRES_PASSWORD = [System.Net.NetworkCredential]::new('', $securePassword).Password
$securePassword.Dispose()
docker compose -f infrastructure/compose.yaml up -d
docker compose -f infrastructure/compose.yaml ps
```

Keep the environment variable set in the same shell for subsequent Compose commands, including `docker compose -f infrastructure/compose.yaml down`; unset it afterward with `Remove-Item Env:POSTGRES_PASSWORD`. Do not commit a `.env` file containing credentials. `docker compose -f infrastructure/compose.yaml down -v` deletes local database/cache data.

## Status

- Owner: DevOps
- Completed: repository inventory; local dependency compose; operations documentation baseline.
- Active: none.
- Blockers: no application code or issue IDs supplied; architecture and deployment choices remain draft pending product inputs and Architect approval.
- Decisions: none. The compose file is a reversible development scaffold, not a production topology decision.
- Dependencies: Architect/Research findings and Builder's app integration contracts.
- Next: agree on deployment topology and app health/migration contracts, then add production runtime, CI, observability and recovery validation.
