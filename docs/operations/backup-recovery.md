# Backup and recovery

## Local PostgreSQL example

With the development compose service running, create a custom-format dump from the repository root:

```powershell
New-Item -ItemType Directory -Force .local-backups | Out-Null
$container = docker compose -f infrastructure/compose.yaml ps -q postgres
docker compose -f infrastructure/compose.yaml exec -T postgres pg_dump -U cineflow -d cineflow -Fc -f /tmp/cineflow.dump
docker cp "${container}:/tmp/cineflow.dump" .local-backups/cineflow.dump
```

Restore into an empty local database (destructive to that database):

```powershell
$container = docker compose -f infrastructure/compose.yaml ps -q postgres
docker cp .local-backups/cineflow.dump "${container}:/tmp/cineflow.dump"
docker compose -f infrastructure/compose.yaml exec -T postgres dropdb -U cineflow --if-exists cineflow
docker compose -f infrastructure/compose.yaml exec -T postgres createdb -U cineflow cineflow
docker compose -f infrastructure/compose.yaml exec -T postgres pg_restore -U cineflow -d cineflow --clean --if-exists /tmp/cineflow.dump
```

Do not commit `.local-backups`. These commands demonstrate a local logical dump only; they are not encrypted or offsite, and have not been restore-tested in this repository.

## Production requirements

Select backup frequency/retention from approved RPO/RTO. Encrypt in transit and at rest, store copies in a separate failure domain/account, restrict and audit access, monitor freshness, and periodically restore into an isolated environment. Record restore duration and data integrity evidence. Include object/media storage and encryption-key recovery if CINEFLOW stores media. QA should own independent failure verification; production restore execution requires the designated operator and approved runbook.
