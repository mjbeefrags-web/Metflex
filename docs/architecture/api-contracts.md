# CINEFLOW API Contract Proposal

**Status:** Draft for Architect/Director approval; not an approved implementation contract.  
**Contract style:** JSON over HTTPS, `/api/v1`, OpenAPI generated/reviewed with every API change.  
**Scope:** Explicit first-release contract proposals for product journeys A–E. No server/client code or OpenAPI file exists yet. Field choices and limits below are conservative proposals and must be confirmed against product/legal decisions before implementation.

## Contract conventions

- `application/json; charset=utf-8` for request and response bodies, except `204 No Content` and any future upload protocol.
- IDs are opaque UUID strings; timestamps are RFC 3339 UTC strings. Decimal/floating values are prohibited for money-like fields (none currently planned); playback positions are integer milliseconds.
- Unknown request fields should be rejected for security-sensitive/admin schemas; ordinary responses may add fields compatibly. Required means required on create unless explicitly marked optional.
- OpenAPI schemas are the source of truth. Response DTOs are explicit and must not serialize ORM objects, password material, internal object keys, raw licensing evidence, worker paths, or service secrets.
- All authenticated endpoints require an active account. Object-level authorization is applied server-side on every request; client-supplied account/profile IDs never grant access.
- Use a common error envelope; redact internal exception details and provider/source credentials.
- **Unresolved:** session vs OAuth/OIDC choice, public vs account-only discovery, account verification/MFA, login policy, supported device clients, launch territories, rights evidence policy, retention, and rate-limit infrastructure.

## Authentication model proposal

Default for browser-only client: opaque, server-side session delivered in a `Secure; HttpOnly; SameSite=Lax` cookie, with CSRF token/header on state-changing requests. Cookie name, domain, SameSite mode, session lifetime and rotation policy must be selected for the deployed domains. Do not return session bearer tokens in JSON. API may support a future OAuth/OIDC bearer mode if native/delegated clients are required; that is outside this v1 proposal.

| Operation | Method/path | Authentication / authorization | Success |
|---|---|---|---|
| Register | `POST /api/v1/auth/registrations` | Anonymous; rate limited | `201` `UserSummary`, verification required by proposal |
| Request email verification | `POST /api/v1/auth/email-verification-requests` | Anonymous; rate limited; same generic response for unknown/already verified address | `202` generic `ActionRequested` |
| Confirm email verification | `POST /api/v1/auth/email-verifications` | One-time opaque email token | `204`; account email becomes verified |
| Login | `POST /api/v1/auth/sessions` | Anonymous; rate limited | `200` `SessionView`, session cookie set |
| Current user | `GET /api/v1/me` | Authenticated | `200` `MeView` |
| Request password recovery | `POST /api/v1/auth/password-recovery-requests` | Anonymous; rate limited; generic response | `202` generic `ActionRequested` |
| Complete password reset | `POST /api/v1/auth/password-resets` | One-time opaque reset token | `204`; password changed and all existing sessions revoked |
| Logout current session | `DELETE /api/v1/auth/sessions/current` | Authenticated + CSRF | `204`, cookie cleared and session revoked |
| List current account sessions | `GET /api/v1/auth/sessions` | Authenticated | `200 SessionList`, no secret material |
| Revoke one session | `DELETE /api/v1/auth/sessions/{session_id}` | Authenticated + CSRF; owner or elevated recovery action | `204` |
| Log out all sessions | `DELETE /api/v1/auth/sessions` | Authenticated + CSRF; recent re-authentication proposal | `204` |

These lifecycle endpoints are proposed so registration, verification, recovery, expiry, and revocation are explicit. They require a transactional email provider and approved account policy; they are not implemented or approved. Recommended registration schema proposal:

```json
// POST /api/v1/auth/registrations
{
  "email": "viewer@example.test",
  "password": "user-supplied secret",
  "display_name": "Viewer"
}
// 201 response
{
  "id": "uuid",
  "email": "viewer@example.test",
  "display_name": "Viewer",
  "status": "pending_verification",
  "created_at": "2026-10-31T10:00:00Z"
}
```

Password must be accepted over TLS, not logged, and stored only using a vetted password hash. Proposed password bounds: 12–128 Unicode code points, reject NUL/control characters, allow passphrases and paste; final policy should follow selected auth library and OWASP guidance. Normalize email for uniqueness; do not silently alter password. Login body: `{ "email": string, "password": string }`. Return generic `401 AUTHENTICATION_FAILED` regardless of whether account exists, is disabled, or secret is wrong (verification UX may require `403 ACCOUNT_ACTION_REQUIRED` after successful authentication). Do not disclose email enumeration via registration conflicts; decide response semantics with the product owner.

Verification request body `{ "email": string }`; verification confirm body `{ "token": string }`. Recovery request body `{ "email": string }`; reset body `{ "token": string, "new_password": string }`. Action-request endpoints always return `202 { "message": "If the account can use this action, instructions will be sent." }` for syntactically valid requests, whether account exists or not. Tokens are high-entropy, single-use, hashed at rest, purpose-bound, invalidated after use/reset, and never logged. Proposed token expiry: verification 24 hours, password recovery 30 minutes; email deliverability, user support, and threat review must confirm these values. Verification token replacement invalidates prior tokens. Reset revokes all account sessions atomically with password update.

Session policy proposal: server stores only a hash/identifier for opaque session secrets. Every request checks account status and session revocation. Rotate the session identifier after login, privilege changes, and re-authentication. Proposed idle expiry 7 days and absolute expiry 30 days; login session lists show creation, last-seen and current-device flag, not IP addresses by default. Session expiry returns `401 SESSION_EXPIRED`; revocation takes effect on the next request. Refresh/extension behavior, concurrent playback limits and remember-device UX are product decisions. Exact cookie domain, `SameSite`, CSRF mechanism, verification requirement before login, re-auth window and session durations remain pending approval.

Verification completion is idempotent only for a consumed token within its operation window; expired, invalid, or reused tokens return the same safe `400 INVALID_OR_EXPIRED_ACTION_TOKEN`. Email change is not included in v1 and must require current-password/MFA re-authentication plus verification of the new address before switching. Password reset invalidates all sessions and outstanding reset tokens, returns no session, and requires a fresh login. Session expiry/revocation is checked on every protected request; server revocation is authoritative immediately. If cookie-only browser auth is selected, CSRF protection is mandatory for state-changing requests. Exact auth and verification policy remains open.

## Profiles (journeys B/C)

| Operation | Method/path | Authorization | Success |
|---|---|---|---|
| List profiles | `GET /api/v1/profiles` | Current account owns all results | `200 ProfileList` |
| Create profile | `POST /api/v1/profiles` | Authenticated | `201 Profile` |
| Get profile | `GET /api/v1/profiles/{profile_id}` | Owner account | `200 Profile` |
| Update profile | `PATCH /api/v1/profiles/{profile_id}` | Owner account + CSRF | `200 Profile` |
| Delete profile | `DELETE /api/v1/profiles/{profile_id}` | Owner account + CSRF; re-auth may be required | `204` |

`ProfileCreate`: `{ "name": string (1–40), "avatar_key": string|null }`. `ProfileUpdate` permits only those mutable fields and requires at least one property. `Profile`: `{ "id": UUID, "name": string, "avatar_key": string|null, "created_at": timestamp, "updated_at": timestamp }`. Proposed cap: 5 profiles per account, requiring confirmation. Avatar values are server-controlled identifiers, not arbitrary URLs. Profile delete behavior for watch history, favorites, and progress must follow approved retention rules; do not cascade silently.

## Catalog, search and detail (journey A)

| Operation | Method/path | Authorization | Success |
|---|---|---|---|
| Search/list | `GET /api/v1/movies` | Public read proposal; may require account in selected launch policy | `200 MoviePage` |
| Movie detail | `GET /api/v1/movies/{movie_id}` | Same policy as catalog | `200 MovieDetail` |
| List genres | `GET /api/v1/genres` | Public read proposal | `200 GenreList` |
| Curated collection | `GET /api/v1/collections/{slug}` | Public read proposal | `200 MoviePage` |

List query fields: `q` optional (1–120 chars), `genre` optional slug, `year_from`/`year_to` optional integers 1888–current year+2, `sort` enum `relevance|title|release_year|recently_added` (default `relevance` when q, otherwise `recently_added`), `limit` integer 1–50 (default 20), `cursor` opaque optional. Reject unknown query fields and invalid combinations (e.g. `year_from > year_to`). A 200 page is:

```json
{
  "items": [
    {
      "id": "uuid",
      "slug": "sample-title",
      "title": "Sample Title",
      "release_year": 2020,
      "poster": {"url": "https://img.example.test/poster.webp", "width": 500, "height": 750, "alt": "Sample Title poster"},
      "genres": [{"id": "uuid", "slug": "drama", "name": "Drama"}],
      "availability": "playable"
    }
  ],
  "next_cursor": null
}
```

`availability` is a public-safe enum `playable|coming_soon|unavailable`; never include private rights proof or unauthorized source detail. `MovieDetail` adds `synopsis` (plain text, bounded to 10,000 chars), `runtime_seconds` nullable positive integer, `original_language` nullable BCP 47 tag, `release_date` nullable ISO date, `subtitles` list of `{ "language": BCP47, "label": string, "kind": "subtitles|captions", "default": boolean }`, and safe availability. Public response policy should avoid revealing territories/rights restrictions more finely than product chooses.

### Cursor pagination proposal

Stable keyset cursor, opaque and integrity-protected by server; bind it to normalized filters and sort, and expire after 24 hours (proposal). Response always includes `items` and `next_cursor` (`null` on final page). Client must treat cursor as opaque. Cursor misuse or filter mismatch returns `400 INVALID_CURSOR`. Ordering must include a unique tie-breaker (`movie_id`) to prevent skipped/duplicated rows. Do not expose raw SQL offsets. Search consistency is snapshot-at-query, not a guarantee across catalog edits.

## Watchlists, favorites, history and progress (journeys B/C)

| Operation | Method/path | Authorization | Success |
|---|---|---|---|
| List watchlist | `GET /api/v1/profiles/{profile_id}/watchlist` | Owner account | `200 MoviePage` |
| Add watchlist entry | `PUT /api/v1/profiles/{profile_id}/watchlist/{movie_id}` | Owner account + CSRF | `200 WatchlistEntry` (idempotent upsert) |
| Remove entry | `DELETE /api/v1/profiles/{profile_id}/watchlist/{movie_id}` | Owner account + CSRF | `204` (idempotent if absent) |
| Read history events | `GET /api/v1/profiles/{profile_id}/history` | Owner account; cursor paginated | `200 ViewingEventPage` |
| Clear one title's visible history/progress | `DELETE /api/v1/profiles/{profile_id}/history/{movie_id}` | Owner account + CSRF | `204`; records a history-clear marker |
| Read resume list | `GET /api/v1/profiles/{profile_id}/continue-watching` | Owner account; cursor paginated | `200 ProgressPage` |
| List favorites | `GET /api/v1/profiles/{profile_id}/favorites` | Owner account | `200 MoviePage` |
| Add favorite | `PUT /api/v1/profiles/{profile_id}/favorites/{movie_id}` | Owner account + CSRF | `200 FavoriteEntry` idempotent upsert |
| Remove favorite | `DELETE /api/v1/profiles/{profile_id}/favorites/{movie_id}` | Owner account + CSRF | `204` |

`WatchlistEntry` and `FavoriteEntry`: `{ "movie_id": UUID, "added_at": timestamp }`, separate resources because “watch later” and “favorite” are different user intents (proposal). `ProgressItem`: `{ "movie_id": UUID, "position_ms": nonnegative integer, "duration_ms": positive integer|null, "completed": boolean, "updated_at": timestamp }`. It is a mutable resume projection, not a viewing-history event. `GET .../history` returns append-only playback activity `ViewingEventItem` values (`event_id`, `movie_id`, `event_type`, `occurred_at`, `position_ms`, optional `duration_ms`); `continue-watching` returns resume projections. `DELETE .../history/{movie_id}` inserts a marker to hide events through the request time and clears resume progress; it does not rewrite immutable events. Physical deletion/anonymization follows approved privacy/legal retention policy. Completion is a server-computed policy (90% duration proposed), never client controlled. Event types, retention, delete semantics and completion threshold remain open.

## Playback session and subtitle access (journey B)

| Operation | Method/path | Authorization | Success |
|---|---|---|---|
| Request playback | `POST /api/v1/movies/{movie_id}/playback-sessions` | Authenticated, owner of profile, account permitted, current territory rights valid | `201 PlaybackSession` |
| Get session state | `GET /api/v1/playback-sessions/{session_id}` | Session user/profile owner | `200 PlaybackSession` |
| End session | `DELETE /api/v1/playback-sessions/{session_id}` | Session owner + CSRF | `204` |
| Record progress | `PUT /api/v1/playback-sessions/{session_id}/progress` | Session owner | `200 ProgressItem` |

Request: `{ "profile_id": UUID, "territory": ISO-3166-1 alpha-2|null }`. Prefer territory derived from trusted server-side deployment/request context; if client-supplied, treat only as a hint and never as authority. Store decision basis, not unverified client claim. The response is a tagged union. For hosted content, `delivery="hosted_cdn"`, `manifest_url` is required, and `provider_handoff=null`; for authorized provider content, `delivery="authorized_provider"`, `manifest_url=null`, and `provider_handoff` contains only provider-approved launch information. A provider playback grant does not permit CINEFLOW to rehost media.

```json
{
  "id": "uuid",
  "movie_id": "uuid",
  "delivery": "hosted_cdn",
  "asset_version": "v3",
  "manifest_url": "https://media.example.test/asset/path/master.m3u8?token=REDACTED",
  "provider_handoff": null,
  "expires_at": "2026-10-31T10:15:00Z",
  "subtitles": [{"language": "en", "label": "English", "kind": "captions", "url": "https://media.example.test/asset/path/en.vtt?token=REDACTED"}]
}
```

For `hosted_cdn`, manifest/subtitle URLs or cookies are bearer grants and must be short-lived (5 minutes proposed, subject to provider requirements), scoped to one asset version, HTTPS-only, excluded from logs/analytics, and protected by restrictive `Referrer-Policy`. If signed cookie/path authorization is chosen, represent the grant via response cookie/header contract instead of putting tokens in URL. Never return object-store origin credentials. CDN must authorize every manifest child request. For `authorized_provider`, provider adapter returns a provider-approved launch URI/descriptor with its own TTL and client handling; do not proxy media bytes through CINEFLOW unless separately authorized rights cover that use. Provider field shape, player support and terms remain unresolved.

Proposed provider union shape: `{ "provider_key": string, "handoff_type": "redirect|sdk_bootstrap", "launch_uri": string, "expires_at": timestamp }`; the provider-specific adapter may define a more constrained DTO before integration approval. Treat `launch_uri` as sensitive, do not log it or expose it to third-party referrers, and only permit HTTPS URLs from the configured provider origin/allowlist. Some providers may prohibit exposing such a URI or require an SDK; in that case the provider's official mechanism takes precedence and this generic field is replaced by an approved provider-specific response contract.

Progress request: `{ "position_ms": integer >= 0, "duration_ms": integer > 0|null, "client_updated_at": timestamp, "sequence": integer >= 0 }`. Response contains normalized `ProgressItem` plus `accepted_sequence`. Server validates that position is not implausibly beyond duration (proposed <= duration + 10 seconds); no content should be marked completed from a client boolean. Repeated identical sequence/body returns same logical result. For out-of-order writes, accept only increasing per-session sequence; stale update returns `409 STALE_PROGRESS` with current state. Client should batch at modest intervals (proposal every 15 seconds and pause/end); this is an operational/product setting to validate.

## Content administration, rights and ingest (journey D/E)

Admin endpoints require the specific role/action from the matrix below; MFA/re-auth for sensitive operations is proposed. Tenant/user supplied ownership fields are forbidden. Every mutation writes audit context. Roles are separate: content editor manages unpublished metadata and submits eligible jobs; rights reviewer reviews scoped evidence; publisher publishes only after rights and processing gates; job operator manages retries; auditor is read-only. Whether one person can hold combined roles is unresolved.

### Proposed administrative role/action matrix

| Action | `content_editor` | `rights_reviewer` | `publisher` | `job_operator` | `auditor` |
|---|---:|---:|---:|---:|---:|
| Create/edit unpublished catalog metadata | Allow | Read | Read | Read | Read |
| Register approved source reference | Allow | Read | Read | Read | Read |
| Create rights grant draft / attach evidence | Allow draft only | Allow | Read | — | Read metadata only |
| Approve/reject/revoke one rights permission scope | — | Allow with reason | — | — | Read |
| Submit/cancel/retry processing job | Submit | Read | Read | Allow operational controls | Read |
| Read safe job state/error | Allow | Allow | Allow | Allow | Read |
| Publish/unpublish after gates | — | — | Allow with reason | — | Read |
| Read restricted evidence and audit detail | No by default | Scoped | No by default | No by default | Allow, read-only |
| Manage roles/service identities | No | No | No | No | No (separate security administrator, out of v1 scope) |

Roles and combinations are proposals only. Prefer separation of duties so the same principal cannot both approve rights and publish the same title; whether this is mandatory depends on staffing and legal risk. **Self-approval policy is unresolved:** the conservative proposal is that the reviewer approving a permission must be a different human from the actor who submitted that permission's evidence; otherwise it remains pending until a second reviewer approves. This may add workflow burden and requires Director/legal approval. High-impact actions should require recent re-authentication/MFA once auth factors are selected. Each route requires the specific permission listed; there is no universal administrator bypass.

**Service principals:** non-human workload identities are provisioned outside the public API by deployment/security administration. A service principal has stable `principal_id`, owner/purpose, narrowly scoped permissions, credential reference in a secret manager, rotation/revocation state, and no user session. Proposed scopes include `outbox:publish`, `media_job:claim`, `media_job:update`, `asset:write_staging`, `telemetry:write`; workers do not get rights-approval or publish permissions. Include service-principal actor IDs in audit records and never impersonate a human reviewer.

**Audit actor:** every administrative mutation stores `actor_type` (`user|service_principal`), `actor_id`, action, subject, timestamp, request/correlation ID, reason for rights/publication decisions, and a safe before/after change record. Audit event is committed in the same transaction as the mutation where possible. Audit retention and restricted evidence visibility remain jurisdiction-dependent.

`AuditEventPage` is `{ "items": [{ "id": UUID, "actor_type": "user|service_principal", "actor_id": UUID, "action": string, "subject_type": string, "subject_id": UUID, "occurred_at": timestamp, "request_id": string, "reason": string|null, "change_summary": safe string }], "next_cursor": string|null }`. It never returns credentials, signed URLs, unrestricted evidence documents, or secret diffs. Service identities are provisioned outside the public API; their credentials are references to secret-manager-managed values. Audit records distinguish human actors from workload principals and cannot treat a service principal as a human rights reviewer.

| Operation | Method/path | Authorization | Success |
|---|---|---|---|
| Create draft title | `POST /api/v1/admin/movies` | `content_editor` | `201 AdminMovie` |
| Update draft metadata | `PATCH /api/v1/admin/movies/{movie_id}` | `content_editor` | `200 AdminMovie` |
| Register media source | `POST /api/v1/admin/movies/{movie_id}/sources` | `content_editor` | `201 MediaSourceSummary` |
| Create rights package and scoped permission drafts | `POST /api/v1/admin/sources/{source_id}/rights-grants` | `rights_reviewer` | `201 RightsGrantDetail`; each permission independently `pending_review` |
| Read rights package and scopes | `GET /api/v1/admin/rights-grants/{grant_id}` | `rights_reviewer` or `auditor`; evidence fields separately restricted | `200 RightsGrantDetail` |
| Approve/reject/revoke one rights permission | `POST /api/v1/admin/rights-permissions/{permission_id}/decisions` | `rights_reviewer` + reason | `200 RightsPermission` |
| Submit processing job | `POST /api/v1/admin/movies/{movie_id}/content-jobs` | `content_editor`; source/evidence reference required | `202 ContentJob` |
| Read job | `GET /api/v1/admin/content-jobs/{job_id}` | `content_editor`, `rights_reviewer`, `publisher`, or `job_operator` | `200 ContentJob` |
| Publish title | `POST /api/v1/admin/movies/{movie_id}/publication` | `publisher` | `200 AdminMovie` or `409 PUBLICATION_GATE_FAILED` |
| Read audit events | `GET /api/v1/admin/audit-events` | `auditor` (read-only; cursor paginated) | `200 AuditEventPage` |

### Admin schemas

`AdminMovieCreate` required: `title` (1–250), `release_year` (1888..current year+2|null), `synopsis` (plain text <=10,000|null), `original_language` (BCP 47|null), `genres` (1..20 unique genre UUIDs), `runtime_seconds` (1..86,400|null), `external_metadata` optional provider reference (provider and terms approved separately). `AdminMoviePatch` allows subset, requires at least one property, and uses JSON Merge Patch semantics; `null` clears nullable fields only. Return `AdminMovie` with `id`, fields, `publication_state`, `active_asset_version|null`, `updated_at`, and safe validation warnings. Do not accept `publication_state` as a mutable field.

`MediaSourceCreate` required: `source_mode` enum `hosted_asset|authorized_provider_playback`, `source_type` enum `authorized_upload|approved_provider`, `source_reference` (provider-specific opaque ID or quarantine upload ID), `source_display_name` (1..200), `source_checksum_sha256` (64 hex chars once acquired bytes exist; null for provider playback). Rights evidence is attached to individual permission records in a separate rights grant workflow, not to the source or whole grant. `hosted_asset` means bytes may be acquired into CINEFLOW storage only with separately approved acquisition, storage, derivative/transcode, and CDN redistribution permissions. `authorized_provider_playback` means the user is directed to/served by the provider's playback path; it grants no implied CINEFLOW download, storage, transcoding, or rehosting right. Free-form remote URLs are disallowed by default. A provider adapter may define a schema only after the integration, rights, terms, and supported client player have been approved.

`RightsGrantCreate` required: `source_id`; `permissions` (non-empty list). Each permission has at least one required evidence item and consists of `{ "category": "acquisition|temporary_storage|permanent_storage|derivative_transcode|cdn_redistribution|third_party_provider_playback", "territories": [ISO-3166-1 alpha-2] or `WORLDWIDE` only when this scope's evidence supports it, "starts_at": timestamp, "ends_at": timestamp|null, "attribution_text": string|null, "restrictions": [string], "evidence": [{"kind": "license_url|document|public_domain_basis|authorization|provider_terms", "reference": string, "sha256": string|null}] }`. Evidence attaches to that `rights_permission_id`, not solely the parent grant. Each permission begins `pending_review` and receives a server-generated permission ID. `RightsGrant` only groups records: no operation approves a whole grant or implicitly approves its child permissions. Decision request `POST /api/v1/admin/rights-permissions/{permission_id}/decisions` is `{ "decision": "approve|reject|revoke", "reason": string (10..2000), "effective_at": timestamp|null, "evidence_ids": [UUID] }` and requires `Idempotency-Key`. Each action creates a server-generated immutable decision ID; approval requires at least one evidence ID already attached to that permission and records the reviewer identity/time. Broad grant approval is disallowed. Hosted HLS publication requires individually approved, currently valid acquisition, storage (temporary processing and permanent origin as applicable), derivative/transcode, and CDN redistribution permissions. Provider playback requires a separately approved `third_party_provider_playback` permission but does not satisfy hosted-media requirements. A temporary storage permission does not imply permanent storage; one territory or term does not imply another. Legal counsel and provider terms must approve categories, evidence sufficiency, public-domain handling, and revocation semantics.

`RightsGrantDetail` response shape: `{ "id": UUID, "source_id": UUID, "permissions": [{ "id": UUID, "category": enum, "territories": [string], "starts_at": timestamp, "ends_at": timestamp|null, "attribution_text": string|null, "restrictions": [string], "status": "pending_review|approved|rejected|revoked|expired", "evidence": [{"id": UUID, "kind": enum, "reference": string, "sha256": string|null, "submitted_by_actor_type": "user|service_principal", "submitted_by_actor_id": UUID}], "decision": {"id": UUID, "actor_type": "user", "actor_id": UUID, "decision": "approve|reject|revoke", "decided_at": timestamp, "reason": string}|null }], "created_at": timestamp }`. Each newly submitted permission gets its own ID, and every evidence reference is persisted against exactly that permission ID. Each decision has its own immutable `decision_id`, permission ID, evidence IDs considered, reviewer actor ID/time, decision and reason. Proposed self-approval rule compares reviewer with evidence submitter; exact separation-of-duty policy requires approval. A grant summary status may be derived for display, but has no authorization effect; every gate evaluates the specific permission records it requires.

`ContentJobCreate`: `{ "source_id": UUID, "target_profile": "vod_hls_v1", "requested_subtitle_ids": [UUID] }`; `Idempotency-Key` header required. Never accept arbitrary FFmpeg arguments or output path. `202 Accepted` means the transaction storing the job and corresponding outbox event has committed; it does not mean a worker has started. A relay publishes outbox events at least once; worker consumers deduplicate using event ID/job idempotency key and make effects idempotent. Proposed relay claims leases, retries transient failures with bounded exponential backoff, quarantines poison events for audited operator resolution, and retains published/dead records according to provisional periods in [data-model.md](data-model.md). No exactly-once delivery is promised. `ContentJob`: `{ "id": UUID, "movie_id": UUID, "source_id": UUID, "state": "queued|validating|processing|validating_output|succeeded|failed|cancelled", "phase": string, "created_at": timestamp, "updated_at": timestamp, "attempt": integer, "error": {"code": string, "message": safe string}|null }`. Raw logs/paths/source credentials are never returned. State transitions are server-owned; retry/cancel operation policy should be added when operational requirements are known.

### Rights and provider playback invariants

For `hosted_asset`, publication gate requires separately approved evidence-backed permissions for acquisition, necessary temporary/permanent storage, derivative/transcode, and CDN redistribution, each covering the asset, intended use, territory, and term. For `authorized_provider_playback`, playback authorization checks a separately approved provider-playback permission and trusted territory/time. The response may contain a provider-approved handoff URI or SDK bootstrap descriptor, but must not fetch/copy/store/transcode/serve provider media through CINEFLOW unless separate explicit permissions authorize each activity. Provider URL/token field shapes and client implementation are deferred until a provider is chosen and its official integration terms are reviewed. Provider playback is not silently converted into HLS.js playback.

**Trusted territory and revocation gate (policy proposal, unapproved):** The authoritative source and confidence policy for viewer territory are not selected. A client-provided country is only a hint. If the system cannot derive territory from an approved trusted source, it must not issue playback. A post-revocation bound is also not approved: proposed maximum residual access is five minutes (grant lifetime/cache TTL), but legal/product/security and CDN/provider capabilities must confirm or tighten it. If a selected CDN/provider cannot enforce an approved bound or promptly revoke access, playback issuance remains blocked for that source. No title may be played by default when territory or revocation guarantees are unknown.

Publication request body: `{ "expected_updated_at": timestamp }` for optimistic concurrency. Server returns `409` if metadata changed or rights/media gates fail, with `details.gates[]` containing safe codes `rights_missing|rights_not_approved|rights_out_of_term|rights_territory_uncovered|asset_missing|asset_not_validated|job_not_succeeded`. It must not publish partially or by a client-provided `published=true` flag.

## Common success and error behavior

### Error envelope

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "One or more request fields are invalid.",
    "request_id": "01J...",
    "fields": [{"path": "year_from", "code": "greater_than_or_equal", "message": "Must be at least 1888."}],
    "retry_after_seconds": null
  }
}
```

`fields` and `retry_after_seconds` are optional and omitted when not applicable. Messages are safe and localized later; clients branch on stable `code`, not message text. Never include stack traces, SQL, file paths, source URLs with credentials, or secrets.

| Status | Meaning | Example codes |
|---|---|---|
| `400` | malformed request, invalid cursor, bad idempotency key | `INVALID_REQUEST`, `INVALID_CURSOR`, `IDEMPOTENCY_KEY_REUSED` |
| `401` | missing/expired/invalid authentication | `AUTHENTICATION_REQUIRED`, `AUTHENTICATION_FAILED`, `SESSION_EXPIRED` |
| `403` | authenticated but action/account policy denies | `FORBIDDEN`, `ACCOUNT_ACTION_REQUIRED`, `RIGHTS_REVIEW_REQUIRED` |
| `404` | resource absent or concealed by object authorization policy | `NOT_FOUND` |
| `409` | state conflict / failed gate / stale optimistic update | `CONFLICT`, `PUBLICATION_GATE_FAILED`, `STALE_PROGRESS` |
| `422` | well-formed JSON fails schema validation | `VALIDATION_FAILED` |
| `429` | rate limit exceeded | `RATE_LIMITED` plus `Retry-After` header and optional body seconds |
| `500` | unexpected internal failure | `INTERNAL_ERROR` with request ID only |
| `502/503/504` | dependency unavailable/timeout, safely classified | `DEPENDENCY_UNAVAILABLE`, `DEPENDENCY_TIMEOUT` |

Use `400` for syntactic/semantic protocol errors, `422` for field/schema validation errors, consistently across the API. Return `404` for unknown resources and, where chosen to prevent enumeration, for resources outside caller scope; do not inconsistently distinguish unauthorized from nonexistent IDs.

### Idempotency and retries

- `PUT`/`DELETE` resource semantics are naturally idempotent. Watchlist add/remove and progress update must meet that behavior.
- Require `Idempotency-Key` for account registration, playback session creation, source/grant creation, content job creation, and publication decisions. Proposed key: 16–128 printable ASCII characters, scoped to authenticated principal + route, with 24-hour retention. Store request hash and response result; same key + same body replays the prior result; same key + different body returns `400 IDEMPOTENCY_KEY_REUSED`.
- Do not retry non-idempotent requests automatically unless idempotency key is retained. Clients may retry idempotent reads/operations on `429/502/503/504` with bounded exponential backoff and jitter, respecting `Retry-After`. Do not retry `401/403/422` without user/state change.
- Job execution itself can happen more than once; HTTP idempotency does not guarantee exactly-once processing. See ADR-0003 proposal.

## Rate limiting proposal

Exact limits require traffic and abuse modeling; suggested initial per-account + per-IP limits below are caps for a first deployment and must be tuned from telemetry. Apply stricter of the two applicable buckets; return `429` and `Retry-After`, and never reveal other accounts' quota state.

| Route class | Initial proposal | Notes |
|---|---:|---|
| Registration/login/verification/recovery/reset | 5 attempts/minute per IP, 10/hour per normalized account key | progressive cooldown; generic auth/action response |
| Catalog/search | 60 requests/minute anonymous IP; 120/min authenticated account | cacheable GET; bounded query complexity and page size |
| Playback grant | 10 creates/minute per account, 3/minute per profile/movie | prevent grant scraping; tune for real seek/device behavior |
| Progress | 1 accepted update per session every 5 seconds; coalesce duplicate/stale messages | product client target ~15 seconds; server may accept end/pause flush |
| Admin mutations/job submission | 30/minute per admin, 5 concurrent active jobs/account (initial proposal) | worker admission quotas; limits depend on hardware/cost |
| Job status reads | 60/minute per admin | polling should back off; consider `Retry-After` while unchanged |

Redis-based limiter is a proposed implementation detail, not a contract dependency; decide fail-open/closed per route. When Redis is unavailable, authentication/playback/admin abuse controls must not silently disappear; conservative local per-process limits can be used only as emergency protection and are not globally consistent. These numeric settings are not approved and require DevOps load/abuse validation before launch.

## Security and validation invariants

- Reject unexpected media types and request bodies above route-specific size limits. JSON API must not accept direct video upload bodies; upload protocol is a separate quarantine flow to be designed.
- Validate route IDs and enums; constrain all text lengths; normalize and validate language tags; use parameterized DB access.
- Search text is plain text, escaped by consumers; do not interpret it as SQL/regex. Bound query length, page size and filter count.
- CORS allowlist is exact per environment; cookie auth requires credentials + CSRF controls. CORS is not authorization.
- Derive trusted client/territory context at a trusted edge; reject spoofed forwarding headers outside trusted proxy.
- Every profile, watchlist, playback session, job, source and grant fetch checks object-level permissions. Admin access and rights mutations produce immutable/auditable record.
- Playback authorization re-evaluates current account status, asset, effective rights, permitted use and territory at grant time.

## Journey coverage

| Journey | Contract coverage |
|---|---|
| A — Discovery | `GET /movies`, `/genres`, `/collections/{slug}`, `/movies/{id}` |
| B — Playback | playback session create/read/end, subtitle descriptors and CDN grant, progress PUT |
| C — Account | registration/login/me/logout; profile CRUD; profile-owned watchlist/history/progress |
| D — Content management | admin movie CRUD, source registration, rights grant/decision, publication gate, audit expectation |
| E — Background processing | idempotent content job create, job state polling, safe failure codes, staged/promotion invariant |

## Open contract decisions blocking approval

1. Auth provider/client choice, verification requirement, password recovery support channel, MFA, session cookie attributes/lifetime/rotation, re-auth windows, and whether external OIDC/native apps are in scope. Endpoints and semantics above are proposals, not approved policy.
2. Public vs authenticated catalog access; visibility policy for unavailable/out-of-territory titles.
3. Launch countries and trustworthy territory resolution method.
4. Rights evidence schema/legal reviewer, public-domain review process, attribution display rules, per-permission evidence/decision policy, self-approval/separation-of-duty rule, and revocation handling for active grants.
5. CDN vendor and whether playback grant uses signed URL, signed cookie, or edge token; exact TTL and client exposure.
6. Upload transport, allowed content source providers, max file size/duration, and job concurrency.
7. Viewing-event semantics/retention/deletion, completion threshold, profile limits, and data retention/deletion. Favorites and watch-later remain separate features; detailed behavior is open.
8. Supported sort/search behavior, language coverage, and collection ownership.
9. Numeric rate limits, session/profile concurrent playback policy, product traffic/SLO, trusted territory source/confidence, and maximum permissible access after revocation.

## References

- [System architecture proposal](system-architecture.md)
- [Data model proposal](data-model.md)
- [Backend research: FastAPI, Pydantic, auth, API versioning](../research/backend-research.md)
- [Security research](../research/security-research.md)
- [Database research](../research/database-research.md)
- [Streaming research](../research/streaming-research.md)
- [OWASP API Security Top 10](https://api-security.owasp.org/editions/2023/en/0x00-header/)
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
