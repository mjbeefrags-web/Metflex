# Local development

## Current state

This repository has a project [README](../../README.md), initial research, proposed architecture, and an `infrastructure/compose.yaml` dependency scaffold. It has no application source, backend/frontend manifests, or dependency instructions. The Compose commands below start only PostgreSQL and Redis; they do not start CINEFLOW. The architecture and research are drafts, not approved deployment decisions.

## Start dependencies

Prerequisite: Docker Engine with the Compose plugin.

From the repository root:

```powershell
$securePassword = Read-Host 'Enter a local-only PostgreSQL password' -AsSecureString
$env:POSTGRES_PASSWORD = [System.Net.NetworkCredential]::new('', $securePassword).Password
$securePassword.Dispose()
docker compose -f infrastructure/compose.yaml up -d
docker compose -f infrastructure/compose.yaml ps
```

Compose requires a non-empty `POSTGRES_PASSWORD`; it has no committed fallback. Database/user defaults are `cineflow`/`cineflow`, with host port 5432; Redis uses host port 6379. Both ports bind only to loopback and both services remain on the internal development network. Redis has no password/ACL, so use this scaffold only for disposable single-developer local development; loopback binding does not isolate local processes/users. Do not use it on shared hosts, staging, or production, and do not commit a `.env` containing credentials. Keep `POSTGRES_PASSWORD` in the same PowerShell session for subsequent Compose commands, including `down`; unset it after stopping the services with `Remove-Item Env:POSTGRES_PASSWORD`. The Compose health checks establish only basic service readiness, not application integration.

Stop services in the same PowerShell session with `docker compose -f infrastructure/compose.yaml down`. This preserves named volumes. `down -v` permanently removes local database and Redis data. The Redis AOF and named volumes are not a backup or recovery plan. No Compose command was executed or tested as part of this documentation change.

## Missing application setup

Builder must add app-specific install, migration, seed, run, and test instructions after implementation. Until then, a new developer cannot run the application from this repository.
