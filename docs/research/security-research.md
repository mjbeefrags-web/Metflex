# Security Research

## Threat model baseline

Assets: account credentials/sessions, personal viewing data, admin controls, licensing evidence, source media, playback entitlements, signing keys, and service secrets. Threats include credential stuffing, object-level authorization flaws, account takeover, SSRF through imported URLs, malicious media/parser inputs, path traversal, token leakage, abuse of transcode capacity, and supply-chain compromise.

Use [OWASP API Security Top 10 2023](https://api-security.owasp.org/editions/2023/en/0x00-header/) as a threat checklist, especially broken object-level authorization, broken authentication, resource consumption, SSRF, and unsafe API consumption.

## Passwords and sessions

Prefer a proven password hashing implementation (Argon2id where available) with current OWASP parameters; do not invent crypto or use fast general-purpose hashes. Rate limit login and recovery, use generic responses where appropriate, support secure reset tokens, and revoke sessions after credential/security changes. Cookie sessions need Secure, HttpOnly, appropriate SameSite, CSRF defenses, rotation, and server-side revocation. If JWTs are chosen, validate signature, fixed allowed algorithm, issuer, audience, expiry/not-before, key rotation, and token type; JWT payload is not confidential by default.

References: [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html), [RFC 7519](https://www.rfc-editor.org/rfc/rfc7519.html), [RFC 8725 JWT BCP](https://www.rfc-editor.org/rfc/rfc8725.html), [RFC 9700 OAuth BCP](https://www.rfc-editor.org/rfc/rfc9700.html).

## Authorization

Enforce deny-by-default authorization for each object and action at the service/API boundary, including watchlists, progress, profiles, admin content operations, and playback grants. Tests should cover cross-account ID substitution. Separate administrator roles from regular user permissions and audit changes to rights, source, publication state, and access grants.

## SSRF and uploads/media ingest

Best initial design: no arbitrary user-supplied source URL; admin chooses from configured/verified source integrations or uploads to a quarantine bucket. If remote fetch is required, allowlist schemes and hosts, resolve and reject private/loopback/link-local/reserved IPs, revalidate every redirect and DNS resolution, block metadata endpoints at network layer, cap redirects/bytes/time, and avoid forwarding credentials. Probe/parse media in sandboxed workers with CPU, memory, disk, process and wall-time limits; keep FFmpeg patched. Validate actual file signatures/container, size, duration and hashes rather than trusting extension or client MIME.

## Rate limits, secrets, dependencies

Apply account/IP/device-aware limits to authentication, search, playback grants, and transcode submission; Redis-backed counters require fail-open/closed policy per endpoint. Store secrets in deployment secret manager, rotate signing and storage credentials, scope service identities, and never log secrets/signed URLs. Lock dependencies, generate SBOM, scan and update; prioritize reachable vulnerabilities and FFmpeg parser exposure. Review licenses for all dependencies and codecs.

## Security sources

[OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/), [OWASP API Top 10](https://api-security.owasp.org/editions/2023/en/0x00-header/), [RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html), [JWT BCP RFC 8725](https://www.rfc-editor.org/rfc/rfc8725.html).
