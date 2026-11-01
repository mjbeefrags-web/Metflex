# Reliability, Monitoring, Backup, and Recovery Plan

**Status:** Planning baseline only; not an SLO or proof of readiness.  
**Owner:** DevOps with Architect and Critical Engineer review. **Issue IDs:** None supplied.

## SLOs to define before production

Agree objectives from product impact and budget before sizing. Candidate indicators:

| Area | Candidate SLI | Questions requiring decision |
|---|---|---|
| API | Availability and latency for catalog, login, playback grant, progress endpoints | Target monthly availability? Which routes are critical? What latency percentiles and regions? |
| Playback | Successful starts, startup time, rebuffer ratio, CDN/origin errors | Measure by device/region? What excludes user network? How much telemetry is permitted? |
| Ingestion | Queue age, job success rate, time to validated asset | Which job classes/priority? What processing deadline and retry ceiling? |
| Data | DB availability, restore point/freshness, backup success | RPO/RTO, PITR window, retention, geographic copy/legal constraints? |
| Rights enforcement | Denied grant after rights expiry/revocation; time to invalidate active CDN grant | Acceptable residual access window based on short grant TTL/cache policy? |

Do not declare an SLO until workload, targets, measurement definitions, and alert owner are agreed. Define error budgets and maintenance behavior only after product and budget inputs exist.

## Metrics, logs, traces, and alerts

Instrument API, worker, database client and edge using OpenTelemetry conventions where appropriate. Export to a selected backend; OpenTelemetry is instrumentation/collection, not a storage/visualization backend. Prometheus is appropriate for numeric operational time-series, but not complete billing or a canonical per-user analytics record. Keep user/title identifiers out of metric labels to control cardinality and protect privacy.

Minimum operational signals before production:

- **API:** request rate, latency distribution, 4xx/5xx by safe route template, saturation, DB pool wait, dependency failures.
- **Database:** connections, query latency/slow queries, locks, storage, backup/PITR status, replication/failover health if used.
- **Queue/workers:** pending and oldest message age, active/reserved jobs, retries, terminal failures, processing duration/resource use, worker heartbeat.
- **Media:** ingest validation failures, staged bytes, transcoding time per media minute, output validation failures, active asset promotion failures.
- **Delivery:** CDN hit ratio, origin egress/error rate, manifest/segment 4xx/5xx, cache authorization errors, request/egress volume, playback start/rebuffer metrics only with approved privacy policy.
- **Security/rights:** admin changes, rights approval/revocation events, repeated auth failures, rate-limit activity, signing/access-control error spikes.

Alert on symptoms with actionable runbooks and owners. Page only when user impact or imminent data/security risk warrants it; use ticket/dashboard warnings for lower urgency. Alerts should cover API availability, DB connection/storage saturation, backup failure, oldest queue age, worker crash loops, CDN/origin error spikes, certificate expiry, and security-critical secret/signing events once thresholds are agreed.

Structured logs must include timestamp, severity, service/version, trace/request ID and safe error code. Exclude access tokens, passwords, signed CDN URLs, raw source URLs with credentials, subtitles/media content, and raw viewing histories. Restrict access and define retention by market/privacy requirements.

## Backup and restore

Proposed minimum before production:

- Automated encrypted PostgreSQL backups and PITR with separate account/region policy approved against data residency requirements.
- Object storage versioning/retention for original authorized masters and validated rendition assets where rights permit; document deletion/expiry obligations and legal holds.
- Export/version infrastructure configuration and secrets metadata (never secret values); maintain recovery access outside the primary deployment.
- Define RPO/RTO, backup retention, restoration priority, and who can initiate restore.
- Rehearse restore to an isolated environment; verify DB consistency, migrations, required object references, rights records, CDN origin access, and playback smoke test using authorized fixtures.
- Record exact date, backup point, commands/procedure, duration, data loss, gaps, reviewer, and follow-up actions. A configured backup without a restore drill is not recovery evidence.

Queue messages should be reconstructible from durable `ContentJob`/outbox state, not the only record of work. Document replay/idempotency before reprocessing. Monitor orphaned staged objects and reconcile them against database asset versions, while respecting rights and retention rules.

## Incident response and recovery

Create runbooks for database outage/failover, Redis/broker failure, worker backlog, malformed/malicious media input, CDN/origin outage, suspected account/session key exposure, leaked CDN signing key, rights revocation/takedown, and restore. Each runbook should name severity, decision owner, containment steps, safe user-facing behavior, evidence to preserve, recovery validation, and post-incident review. Do not invent failover guarantees without a provider and tested configuration.

## Operational readiness gates

1. SLOs, RPO/RTO and on-call owner are documented.
2. Security review approves network, IAM, secret, proxy and CDN signing boundaries.
3. Backup and restore drill is successful and recorded.
4. Deployment and rollback are rehearsed with compatible migrations.
5. Alerts, dashboards and incident runbooks are exercised.
6. Media publishing/playback authorization demonstrates rights state and revocation handling in a controlled environment.
7. Load tests meet agreed thresholds with CDN and representative media fixtures; cost estimate reviewed against budget.

No reliability, security, load, backup, restore, deployment, or playback test has been executed as part of this plan.
