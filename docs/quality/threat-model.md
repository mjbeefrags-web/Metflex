# CINEFLOW Initial Threat Model

**Status:** Independent initial analysis; validate against the Architect's approved data-flow and deployment diagrams. This is not a penetration test, audit, or legal review.

## System boundary and assumptions

Expected trust zones: browser/client; public API/edge; PostgreSQL; Redis/broker; media-processing workers; private object storage; CDN; external metadata/content providers; administrators. Exact cloud/network topology is unresolved. Treat all browser input, metadata providers, uploads, imported media, queue messages, and media parsers as untrusted. Content delivery and administrative actions are high-impact trust boundaries.

## Assets and security objectives

| Asset | Security objective |
|---|---|
| Passwords, session IDs, auth keys | Prevent disclosure/replay; enable revocation and rotation |
| Profiles, watch history, preferences | Confidentiality, account/profile isolation, defined retention/deletion |
| Admin privileges and audit trail | Restrict changes, preserve attributable history |
| Rights/license evidence and publication state | Integrity, traceability, no publication beyond grant scope |
| Media source and processed renditions | Confidentiality before publication, integrity and availability |
| Playback grant/signing credentials | Narrow scope, short lifetime, no leakage/replay beyond policy |
| Worker/broker/service credentials | Least privilege and limited blast radius |

## Threat register

| ID | Threat / path | Impact | Required controls | Verification cases |
|---|---|---|---|---|
| T-01 | Broken object-level authorization: change profile/movie/job ID | History disclosure, account takeover impact, unauthorized playback/admin actions | Deny-by-default service authorization; derive principal from session; scope DB reads/writes; separate admin policies | Cross-user and cross-profile ID substitution across every protected endpoint |
| T-02 | Credential stuffing, brute force, reset abuse | Account takeover | Password hashing via vetted library; per-account/IP rate limits, breached-password policy, secure one-time reset, generic errors, MFA policy decision | Threshold/lockout behavior, reset replay/expiry, session revocation |
| T-03 | Session/token theft or leakage through URL/log/referrer/storage | Unauthorized account/playback use | HttpOnly Secure cookie sessions or carefully validated token architecture; CSRF protections; short delivery credentials; redact logs; rotate/revoke | Inspect URLs, browser storage, logs; replay after expiry/revocation |
| T-04 | Rights metadata bypass or stale rights/cache state | Illegal distribution exposure | Rights gate before publish and playback, territory/time evaluation, auditable review, invalidate cache/revoke delivery, takedown workflow | Expired, revoked, unknown territory, future start, unpublished content |
| T-05 | SSRF via remote source, redirects, DNS rebinding | Internal service/metadata endpoint access | Avoid arbitrary URLs; allowlisted egress; block private/reserved IPs and metadata endpoints at network layer; revalidate redirects/resolution; cap response and time | Loopback/private IP, IPv6, redirect chains, mixed DNS, oversized/slow body |
| T-06 | Malformed media exploits FFmpeg/parser or exhausts resources | Worker compromise, service outage, cost spike | Isolated non-root workers; patched pinned builds; resource quotas/timeouts; no ambient secrets; quarantine; input caps; restricted egress | Corrupt/fuzzed/oversized media, timeout, worker termination and cleanup |
| T-07 | Malicious subtitle/metadata rendered unsafely | XSS, phishing, navigation to hostile URLs | Escape text; sanitize only if rich markup is necessary; CSP; validate links and schemes; protect admin preview | Stored/reflected payloads in title, description, subtitle, artwork metadata |
| T-08 | Admin account abuse or compromise | Rights fraud, source replacement, data exposure | MFA/strong admin auth, least privilege, separation of duties for rights approval, immutable audit records, alerting | Privilege escalation and unauthorized publish/rights changes |
| T-09 | Queue/task forgery or duplicate execution | Unauthorized transcode, duplicate assets, unsafe state | Authenticated private broker; message validation; idempotent jobs; durable state transition; service identity | Duplicate/out-of-order/replayed messages and unauthorized producer |
| T-10 | Signed URL/cookie copied or CDN misconfiguration | Media copied outside intended access window; origin bypass | Private origin, scoped short TTL, path/asset restriction, origin access control, no sensitive query logging, clear revocation SLA | Direct origin access, cross-title token use, expired token, cache leakage |
| T-11 | API resource exhaustion / expensive search or job flood | Cost/availability degradation | Per-principal/IP quotas, bounded query/result sizes, request limits, admission control, separate queue capacity | Concurrent expensive search, login and transcode submission floods |
| T-12 | Dependency/container compromise | RCE/data exfiltration | Locked dependencies, SBOM/scanning, minimal non-root containers, provenance, patch process, secret isolation | CI policy checks and image inspection at release |
| T-13 | CSRF/CORS misconfiguration | Unintended state changes or data exposure | CSRF tokens/origin checks for cookie-auth mutations; exact CORS allowlist, credential rules | Cross-site requests, null origins, preflight and credential cases |
| T-14 | Sensitive telemetry/high-cardinality leakage | Privacy disclosure and operational cost | Data minimization, no tokens/PII labels, access-controlled logs, retention limits | Review logs/traces/metric labels using synthetic sensitive markers |

## Security test priorities

Prioritize T-01, T-04, T-05, T-06 and T-10 before any playback/content-management release. Test every API object boundary with two users and roles. Test rights evaluation as a pure policy matrix plus integration at publication and playback entry. Validate CDN and direct object-origin paths against real deployment configuration. Use safe synthetic or deliberately benign malformed media fixtures; destructive parser fuzzing must run in an isolated environment.

## Security decision dependencies

- Auth/session architecture and supported clients.
- CDN vendor, token model, cache key, origin protection, and revocation latency.
- Whether trusted uploads or any remote URL ingestion exists.
- Admin role and rights approval model.
- Launch jurisdictions, privacy/retention obligations, and content-license review.
- Network boundaries, secret manager, and worker isolation in deployment design.

## Sources

- [OWASP API Security Top 10 2023](https://api-security.owasp.org/editions/2023/en/0x00-header/)
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [RFC 9700 OAuth 2.0 Security BCP](https://www.rfc-editor.org/rfc/rfc9700.html)
- [RFC 8725 JWT Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725.html)
- [FFmpeg documentation](https://ffmpeg.org/ffmpeg.html)
- [CINEFLOW security research](../research/security-research.md)
