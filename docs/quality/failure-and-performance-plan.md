# Failure and Performance Test Plan

This is a test plan only. No application, infrastructure, or performance tests have been executed. Acceptance thresholds must be set by product/architecture owners after target regions, workload, budget, and SLOs are known.

## Failure-mode coverage

| Component/failure | Expected product behavior to define | Test method / evidence |
|---|---|---|
| PostgreSQL unavailable / connection pool exhausted | API fails boundedly with safe error; no false success; readiness reflects loss; recovery reconnects | Integration fault injection; connection metrics/logs; verify transaction rollback |
| Migration fails midway / lock contention | Release stops safely; old version behavior and recovery path documented | Disposable DB migration rehearsal on representative schema/data; capture lock duration and recovery steps |
| Redis unavailable | Endpoint-specific policy: auth/security controls should not silently disappear; cache may bypass to DB under load control; session policy explicit | Network fault injection; assert selected fail-open/closed behavior and rate bounds |
| Broker unavailable / queue backlog | API rejects/degrades job submission explicitly; state not falsely “processing”; backlog observable | Stop broker and accumulate queue; verify admission, idempotency, backlog-age alert and recovery |
| Worker killed during transcode | Job resumes/retries or fails terminally; partial outputs quarantined; no partial asset published | Kill worker at each workflow transition; inspect DB state, staged objects and retry count |
| Duplicate/out-of-order task | Idempotent state transition, no duplicate active asset | Replay same event and deliver older state event; assert uniqueness/version constraints |
| FFmpeg exits nonzero / hangs / fills disk | Bounded termination, cleanup, actionable error, worker remains available | Synthetic corrupt input, timeout and disk quota test in isolated worker |
| Object storage denied/slow | No broken “success” publication; retry/degraded status; no leaked temp source | Inject 403/5xx/latency; ensure promotion occurs only after verified writes |
| CDN/origin failure | Playback communicates retryable/unavailable state; API not overloaded by byte proxying | Simulate CDN 5xx/timeout and origin denial; inspect player UI and edge metrics |
| Rights revoked during cache lifetime | Delivery revocation meets documented maximum latency; no stale authorization beyond that bound | Revoke rights and measure access through API, manifest, segment and cached CDN path |
| Email/provider/search integration unavailable | Core flow has defined degraded behavior; no fabricated success/data | Stub at adapter boundary for deterministic contract tests; provider outage integration test where safe |
| Disk/memory/CPU saturation | Admission control and health state prevent cascading failure | Constrained container resource tests and queue saturation; collect saturation and error metrics |

For each failure define: detection signal, user-visible behavior, retry policy, duplicate safety, rollback/recovery, data integrity check, and alert/runbook. Avoid unbounded retries and retry storms.

## Performance test plan

### Workloads to model

1. **Catalog/search:** anonymous and authenticated browse, common filters, page depth, query shape, cache hit/miss, concurrent search.
2. **Identity:** login/session validation and profile switching; include rate-limit behavior and realistic credential hashing cost.
3. **Playback control plane:** playback grant creation, progress updates, playlist authorization if proxied, token expiry/revocation.
4. **Media data plane:** CDN requests for manifests, variants, segments, and subtitles. Test direct edge delivery and origin/cache-miss behavior; API load testing cannot represent video egress capacity.
5. **Ingestion/transcoding:** mixed durations/resolutions/codecs, queue arrival rate, retries, resource limits, object writes, and storage growth.
6. **Soak and recovery:** steady load plus burst, extended soak, deploy/restart, broker outage, and autoscaling/capacity transition if used.

### Method

- Define a realistic workload model from expected concurrent viewers, sessions, requests per session, bitrate ladder, view duration, rights-region mix, and media ingest rate.
- Fix environment shape, versions, dataset scale, warm/cold cache state, and test client/network profile. Record all of them.
- Begin with baseline at low concurrency; increase load in controlled steps; include realistic think time and playback segment cadence.
- Measure API p50/p95/p99 latency, throughput, error rate, DB query/pool saturation, Redis/broker latency and queue age, worker CPU/memory/disk, transcode time per media minute, CDN cache-hit ratio, origin egress, playback startup/rebuffer ratio, and cost signals.
- Separate steady state, burst, cold cache, dependency failure, and recovery observations. Repeat enough to quantify variance.
- Use synthetic licensed assets; load generation and media egress must be isolated from production user traffic.

### Thresholds pending decisions

No numeric target is approved. Set per-endpoint latency/error SLOs, playback startup/rebuffer targets, transcode completion objective, queue age, availability, and monthly cost envelope before calling a load result pass/fail. A benchmark without an agreed workload and SLO is descriptive evidence only, not capacity certification.

## Risk-based test exit criteria

- Security-sensitive acceptance cases have explicit pass evidence: cross-account isolation, rights state, playback grant scope/expiry, origin denial, upload/ingest containment.
- High-impact failure scenarios include recovery evidence and a runbook, not only an error response.
- Performance reports include workloads, versions, test region, cache state, resource shape, raw summary data, and approved thresholds.
- All critical user journeys have automated or documented reproducible acceptance coverage.
- Open defects are ranked by impact/likelihood and explicitly accepted by the responsible owner; QA does not silently waive risk.

## References

- [Prometheus overview and limits](https://prometheus.io/docs/introduction/overview/)
- [OpenTelemetry overview](https://opentelemetry.io/docs/what-is-opentelemetry/)
- [Celery task behavior](https://docs.celeryq.dev/en/stable/userguide/tasks.html)
- [SQLAlchemy async session concurrency](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)
- [CINEFLOW infrastructure research](../research/infrastructure-research.md)
- [CINEFLOW streaming research](../research/streaming-research.md)
