# Networking

## Local baseline

The development compose project gives PostgreSQL and Redis a private internal network and publishes their ports only on `127.0.0.1`. It contains no public ingress.

## Production decisions still needed

Architect and Research must establish cloud/provider, region and residency, DNS ownership, edge/TLS termination, private subnet layout, egress policy, service discovery, and whether media is hosted or delivered through an authorized CDN. Catalog metadata does not establish streaming rights.

When chosen, expose only the edge publicly; keep database, cache, and worker traffic private; allow only necessary service-to-service flows; configure explicit CORS origins; automate certificate renewal; and prevent authenticated/private media from entering shared caches. Add tested Nginx or managed gateway configuration only after actual app routes and upstream health paths are known.
