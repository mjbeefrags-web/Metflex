# Infrastructure Research

## Docker and Compose

Use multi-stage builds, pinned base image digests or controlled tags, non-root runtime, health checks, minimal images, and explicit resource limits. Compose is useful for local development and can run production-like deployments, but Docker documents production-specific overrides; Compose alone is not an HA scheduler or managed database. Keep secrets out of images and source control.

Reference: [Docker Compose production](https://docs.docker.com/compose/how-tos/production/).

## Nginx / edge

Nginx can reverse proxy API traffic and serve static assets. Its proxy module documents buffering and cache behavior; tune separately for API vs media and avoid proxying high-volume media through the application container. Set trusted proxy headers carefully, body/time limits, TLS policy, request IDs, and correct forwarded client address handling. WebSocket requires explicit proxy configuration if later needed.

Reference: [Nginx proxy module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html).

## Cloud and storage

No provider can be recommended responsibly without region, data residency, egress budget, expected viewing hours, bitrate mix, availability target, and operations capacity. Evaluate object storage + CDN based on regional egress, request charges, cache behavior, signed access, logging, lifecycle, origin shielding, and minimum billing. Separate metadata DB from media bytes and avoid serving video through FastAPI.

## CI/CD

GitHub Actions is a candidate if the repository is hosted on GitHub. Use least-privilege `GITHUB_TOKEN`, protected branches/environments, dependency pinning, artifact provenance, secret-scanning, and migration checks. Build once and promote immutable artifacts. No pipeline or repository hosting configuration is present to verify.

Reference: [GitHub Actions security hardening](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions).

## Observability

Prometheus scrapes and stores time-series metrics; good for operational metrics/alerts but explicitly not a complete accurate per-request billing source. OpenTelemetry provides vendor-neutral APIs/SDKs and collection for traces, metrics, logs; it is not itself a telemetry backend. Use structured logs with correlation IDs, RED metrics, queue depth/age, transcode duration/failure, CDN/origin metrics, playback startup/rebuffering where consent/privacy permits. Avoid high-cardinality labels such as user IDs, raw URLs, or title IDs.

References: [Prometheus overview](https://prometheus.io/docs/introduction/overview/), [OpenTelemetry overview](https://opentelemetry.io/docs/what-is-opentelemetry/).

## Reliability and cost

Set explicit SLOs before multi-region or elaborate redundancy. Estimate cost from storage retained, transcoded renditions, CDN egress (viewing hours × average delivered bitrate), request volume, and transcoding compute. Add lifecycle deletion and origin protection. Restore tests, DB backups/PITR, and queue recovery are required operational evidence before production claims.
