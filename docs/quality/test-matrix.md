# CINEFLOW test matrix

**Status:** Planned coverage; no implementation or tests are available in this workspace.
**Issue:** Not supplied. **Last reviewed:** 2026-10-31.

| Area / risk | Minimum scenario | Evidence required | Current status |
|---|---|---|---|
| API contract | Valid, missing, malformed, oversized and boundary inputs; stable errors | Executed contract tests and schema/version | Blocked: no API defined |
| Authentication | Missing, invalid, expired, revoked and replayed credentials | Executed tests with expected status and audit behavior | Blocked: auth design absent |
| Authorization | Cross-user/profile/resource ID substitution; role and ownership matrix | Negative tests prove no data/content disclosure | Blocked: roles/resources absent |
| Business rules | Duplicate request, out-of-order action, state transition boundary | Issue-linked assertions and persisted state | Blocked: requirements absent |
| Persistence | Constraint violations, concurrent writes, transaction rollback | DB engine/version, setup, assertions, rollback evidence | Blocked: schema absent |
| Pagination/querying | Empty, last/first page, invalid cursor, stable ordering | Contract tests and query plan at representative data size | Blocked: API/schema absent |
| Frontend journeys | Loading, empty, error, retry, session/profile change, narrow viewport, keyboard | Browser/version, screenshots or automation report | Blocked: frontend absent |
| Media | Invalid/corrupt file, unsupported codec, malformed HLS, missing segment/subtitle | Safe fixture hashes, player logs, expected recovery | Blocked: media scope absent |
| Playback access | Unauthorized title, forged ID, expired URL/license, profile change | Authorization result and redacted request trace | Blocked: rights/playback design absent |
| Provider integration | Malicious URL, redirect, timeout, slow response, malformed response | Stub scenario, egress evidence and bounded failure | Blocked: providers absent |
| Rate limits | Boundary, parallel clients, alternate identity/token paths | Config, request counts, responses, reset behavior | Blocked: policy absent |
| Duplicate events | Same event repeated and concurrent delivery | Idempotency key/state evidence and side-effect count | Blocked: event design absent |
| Dependencies | PostgreSQL/Redis/storage/CDN unavailable; worker crash | Fault injection, recovery time and consistency checks | Blocked: deployment absent |
| Performance | Cold/warm cache, realistic concurrency, hot title, slow dependencies | Workload profile, p50/p95/p99, errors, CPU/memory/queue | Blocked: targets/runtime absent |
| Accessibility/compatibility | Keyboard, labels, focus, supported browser/device matrix | Tool/browser versions, reproducible defects | Blocked: UI/support matrix absent |
| Migration/release | Upgrade, rollback, mixed-version behavior, backup restore | Before/after data checks and recovery evidence | Blocked: migrations absent |

Do not mark a row verified from code inspection alone. Add issue IDs and exact commands/results when implementation becomes available.
