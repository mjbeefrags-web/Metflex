# Independent Quality Strategy

## Purpose and current limits

This document defines how CINEFLOW should be independently verified. It is a plan, not execution evidence. At initial repository inspection the workspace had research documents only. The current workspace also has draft coordination, architecture, planning, quality, and operations documentation, but no application source, tests, manifests, Git history, test infrastructure, published GitHub issues, or approved architecture. The strategy therefore specifies gates and test classes, not concrete code-level test commands.

## Quality principles

1. Verify observable user and security outcomes against approved contracts, not implementation shape alone.
2. Test authorization boundaries with a distinct actor/resource matrix; authentication success alone is not proof of authorization.
3. Use real PostgreSQL and the selected broker/object storage contract for integration tests where behavior depends on their semantics; mocks cannot prove migrations, constraints, queue delivery, CDN signing, or HLS requests.
4. Make processing jobs idempotent and test duplicate delivery, interruption, retry exhaustion, and partial outputs.
5. Preserve independent QA: the Builder supplies reproducible evidence; Critical Engineer chooses/runs or reviews independent checks and reports results separately.
6. No completion claim without a named commit/build, exact commands/environment, result, and acceptance criteria mapping.

## Verification layers

| Layer | Scope | Examples | Required evidence |
|---|---|---|---|
| Static checks | Types, lint, dependency/config hygiene | TS/Python type/lint checks, secret scan, dependency scan | Tool/version, command, exit status, report artifact |
| Unit | Pure policy/domain logic | Rights-window decision, profile ownership, progress merge rules | Test names/results, deterministic fixtures |
| API/component | Route and contract behavior | Pydantic validation, error schema, rate limit, auth dependencies | OpenAPI snapshot/diff, request/response tests |
| Integration | Real persistence/queue/storage adapters | Alembic upgrade, constraints, session isolation, task retry | Disposable environment versions, seed data, logs |
| Browser E2E | Critical user journeys | search to detail, authorized playback initiation, captions, progress | Browser/device versions, trace/video/screenshots, result |
| Media conformance | HLS and encoded outputs | playlist parse, rendition alignment, segment fetch, captions | Input/output hashes, FFmpeg build, validator report, sample playback |
| Resilience | Component failure behavior | Redis loss, CDN/origin errors, DB disconnect, worker kill | Injected failure, observed behavior, recovery evidence |
| Load/soak | Explicit SLO and capacity goals | catalog API, login, playback grants, CDN delivery, transcode queue | Workload model, environment, p50/p95/p99, errors, saturation |

## Release gates

### Gate 0 — Contract readiness

- Product acceptance criteria have IDs and named owner.
- Approved API schemas and authorization rules exist.
- Rights publication and playback conditions have been reviewed by the appropriate legal/content owner.
- SLOs, supported browser/device matrix, and launch territories are agreed.

### Gate 1 — Feature merge

- Relevant unit/API/integration checks pass on the proposed commit.
- Migration tested from an empty supported DB and documented upgrade path.
- Security negative cases and ownership checks included for protected resources.
- No placeholders, fake successful integrations, swallowed errors, or untracked manual data setup.

### Gate 2 — Release candidate

- E2E coverage for each accepted user journey; media output has device-path validation.
- Dependency/container scans and secret checks reviewed; known findings triaged.
- Backup restore and operational recovery evidence available for production claims.
- SLO-based capacity and failure tests pass in a representative environment.

## High-value acceptance scenarios

### Discovery

- Catalog supports stable pagination and documented filters/sort.
- Empty, invalid, and boundary filters return documented responses.
- Search does not disclose hidden/unpublished/unlicensed items.
- Search failure is visible and does not return fabricated results.

### Playback authorization

- Authenticated user requests playback for published title with current rights for user's territory: authorized response is scoped and expires as specified.
- Anonymous, revoked, expired, out-of-territory, unpublished, or removed asset: denied without leaking sensitive source details.
- Changing a title/profile/session identifier to another account's object cannot access playback, history, or watchlist.
- Manifest, variant playlist, segment, subtitle, and key resources enforce the intended access policy through CDN/origin.
- Expiry/revocation behavior is tested at API and delivery edge; behavior must match the approved revocation latency.

### Content publication and ingestion

- Missing/ambiguous rights evidence prevents publication and playback.
- License territory and term boundaries are tested at before/on/after points with documented timezone semantics.
- Duplicate task delivery does not create duplicate assets or corrupt the active version.
- Malformed, oversized, misleading MIME/container, and resource-intensive inputs fail safely and are quarantined.
- Partial transcode artifacts never become the active playable asset.

### User library

- Only authorized account/profile can list/change own watchlist and history.
- Watchlist uniqueness and concurrent updates are enforced by DB constraints/transaction behavior.
- Progress updates follow documented conflict/monotonicity rules and are bounded in write frequency.

## Independent verification record format

For each issue/acceptance criterion capture:

```text
Issue / criterion:
Commit or artifact digest:
Environment (OS, runtime, service versions, configuration class):
Command or manual procedure:
Expected result:
Observed result:
Evidence link/path (logs, report, trace, screenshot):
Reviewer and date:
Defects / limitations:
```

Do not paste secrets, bearer URLs, personal watch data, or production credentials into reports. Redact signed playback URLs and tokens while preserving status and request IDs.

## Test data and privacy

Use synthetic users and licensed/public-domain sample media with a recorded source and permitted test use. Never load real user watch histories into development/test. Test titles should include rights territory/term edge cases and explicit revocation. Keep fixtures deterministic and versioned; avoid depending on mutable external metadata APIs during routine CI.

## References

- [OWASP API Security Top 10 2023](https://api-security.owasp.org/editions/2023/en/0x00-header/)
- [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/)
- [RFC 8216 HLS](https://www.rfc-editor.org/rfc/rfc8216)
- [Apple HLS authoring specification](https://developer.apple.com/documentation/http-live-streaming/hls-authoring-specification-for-apple-devices/)
- [Research compatibility matrix](../research/compatibility-matrix.md)
- [Research security guidance](../research/security-research.md)
