# Product Requirements and User Journeys

**Status:** Draft for product/Director approval. These are planning requirements, not implemented capabilities. The product vision and kickoff are the source for scope; unresolved details are explicitly marked.

## Product outcome

CINEFLOW is a movie discovery and streaming platform. Authorized video playback and adaptive streaming are eventual product commitments. A title is playable only when documented authority covers the title and associated media in the user's territory and at playback time. The MVP boundary, exact hosting/processing/delivery model, and which titles/sources are available remain to be confirmed. The platform supports discovery, library state, authorized playback, content operations, and auditable rights controls, and starts as a cost-aware modular monolith.

## Personas

- **Viewer/user:** discovers catalog, searches/filters, and streams titles authorized for the user's territory.
- **Registered user:** manages account/profile(s), watchlists, favorites, continue-watching, progress, viewing history, and subtitles/preferences as policy allows.
- **Content operator:** submits metadata/media and records rights evidence; cannot self-approve unless explicitly authorized.
- **Rights reviewer/admin:** approves, rejects, expires, or revokes rights and publication, operates content workflows, and reviews audit/analytics; sensitive actions are audited.
- **Operations engineer:** observes and recovers jobs/services without bypassing content authorization.

## Functional requirements

1. Accounts: registration, authentication, logout, recovery, and account security are eventual capabilities; exact identity flow, verification and policy await decision.
2. Profiles: users manage isolated profiles; profile ownership and limits are eventual capabilities, with detailed rules open.
3. Catalog: title details, genres/categories, cast/crew and authorized artwork/metadata as supported by source rights.
4. Discovery: search and filters with stable pagination and empty/error states.
5. Rights: every distributable media asset has verifiable grant/license evidence including rights holder/source, territories, effective dates, permitted activities, restrictions, attribution, and status. Rights availability varies by title and territory; this does not make streaming itself optional.
6. Publication: draft title is not publicly visible or playable until metadata and rights review requirements pass.
7. Playback: backend authorizes account/profile/title/asset/territory/time and availability; issues narrowly scoped short-lived delivery authorization; client supports adaptive HLS where supported. The delivery mechanism may be first-party hosted or an explicitly authorized provider, subject to architecture and product decisions.
8. User library: watchlists, favorites, playback sessions/progress, continue-watching, and viewing history are eventual committed capabilities; detailed conflict, retention and deletion rules await approval. Ratings are not specified in the kickoff feature set and are outside this baseline pending separate product direction.
9. Subtitles: language/label/format metadata and authorized delivery; captions remain selectable and accessible.
10. Content processing: bounded asynchronous validation/probe/transcode/packaging workflow with durable status, retries, idempotency, audit events, and explicit failed/cancelled outcomes.
11. Admin: least-privilege operation for catalog, source, rights review, publication and takedown; audit sensitive actions.
12. Recommendations: recommendations are an eventual committed capability; algorithm, explanation, evaluation, personalization and retention approach remain open.
13. Analytics: analytics is an eventual capability; keep operational telemetry distinct from product analytics and define data, consent, minimization, retention, and deletion before collection.

## User journeys and acceptance criteria

### A. Discovery

**Journey:** open homepage → browse curated collections and genres → search/filter → view details and availability.

**Acceptance:** catalog responses are paginated and stable; filters and search can be shared/bookmarked where sensible; unavailable/expired content has clear state; no draft, revoked, or out-of-territory title leaks into public results; accessible keyboard interaction and loading/error/empty states exist.

### B. Playback

**Journey:** select Play → server checks user/profile, title, rights territory/time and published asset → server grants scoped access → player loads manifest/renditions/subtitles → progress updates → errors presented.

**Acceptance:** denied/expired/revoked content receives no media credential; child playlist, segment, subtitle, and key delivery are protected by the selected mechanism; credentials are short-lived and absent from logs; client detects native HLS or hls.js support; unsupported/error states are actionable; progress is throttled and bound to the active profile/session; seek/resume rules are defined and tested.

### C. Account and profiles

**Journey:** register → verify if configured → authenticate → create/select profile → manage watchlist and resume title.

**Acceptance:** authorization prevents cross-account/profile access; profile lifecycle rules cover deletion and history; session expiry/revocation works; CSRF protection applies to cookie-authenticated mutations; UI supports keyboard and accessible status/errors.

### D. Content management and rights

**Journey:** authorized operator enters metadata/source → evidence and scope recorded → independent review as required → validation passes → publication → revocation/takedown if needed.

**Acceptance:** unverified, expired, disputed, or insufficiently scoped rights block publication and playback; reviewers can record decision/rationale; sensitive transitions are audited; rights revocation prevents new grants and purges/blocks cached delivery within an agreed bounded time; attribution/restrictions are preserved.

### E. Background processing

**Journey:** submit authorized asset → quarantine and validate → probe → process to staged outputs → validate rendition set → atomically promote asset pointer → record status and notify operator.

**Acceptance:** API request is not held for a transcode; task is idempotent under retry; worker resource limits prevent unbounded processing; partial outputs cannot become public; job status and failure reason are inspectable; only successfully validated and rights-approved output is promoted; retries and cancellation have deterministic behavior.

## Nonfunctional requirements — draft targets pending approval

- **Security:** threat model and OWASP API risks addressed; deny by default; object-level authorization tests; secrets not committed/logged; dependencies and media parsers updated.
- **Reliability:** API, DB, queue, workers, object store, and CDN failure behavior defined; backups/restores demonstrated before production readiness.
- **Performance:** define service-level objectives after expected catalog size, viewer concurrency, devices, and budget are known. No numeric SLA is approved.
- **Accessibility:** target WCAG 2.2 AA as a proposed acceptance target; confirm with product owner and test with keyboard and assistive technology.
- **Privacy:** collect minimum viewing/identity data; document purpose, retention, deletion, consent, and export for applicable regions.
- **Maintainability:** modular monolith with migration-managed schema, typed contracts, structured logs, repeatable local environment, and automated tests.
- **Cost:** optimize media egress and rendition footprint; cloud/provider and unit-cost ceilings remain unresolved.

## Decisions required before scope commitment

Launch territories/legal owner; viewer age policy; guest playback; target browsers/TVs/devices; account verification/MFA/identity provider; profile count and family sharing; ratings/comments policy; accessibility target; metadata/artwork licensing source; upload vs configured provider ingestion; DRM requirements; deletion/retention policy; concurrency/catalog/bitrate assumptions; provider and budget; analytics consent and retention.
