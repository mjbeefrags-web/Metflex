# CINEFLOW Project Decision Register

This is the shared register. A proposal is not an approved decision until the responsible owner and approval are recorded. Research recommendations and draft ADRs must remain labeled as proposals. Link implementation issues when GitHub is available.

| ID | Decision | Status | Owner | Evidence / next action |
|---|---|---|---|---|
| P-001 | Content must be public domain, openly licensed, or explicitly authorized; no unauthorized scraping/redistribution/DRM circumvention | Product constraint | Engineering Director | Encode as rights review and playback acceptance gate |
| P-002 | Begin with a modular monolith; no speculative microservices | Direction set by kickoff; details pending Architect approval | Architect | Draft architecture/ADR and revisit only with measured justification |
| P-003 | Preferred stack is the technology baseline in root README | Proposed baseline | Research Engineer / Architect | Verify compatibility and pin exact versions before implementation |
| P-004 | PostgreSQL system of record, Redis bounded auxiliary use, HLS delivery via object storage/CDN | Research recommendations, not approved | Architect | Review research decisions and complete provider/cost inputs |
| P-005 | Browser auth should favor secure server-side sessions unless client needs require OAuth/OIDC | Research recommendation, not approved | Architect | Decide after account/client scope is established |
| ADR-0001 | Modular monolith with explicit domain modules | Proposed; not approved | Architect | [ADR-0001](architecture/adr/ADR-0001-modular-monolith.md); confirm domain boundaries and module contracts |
| ADR-0002 | Rights approval and technical media readiness are separate; private origin/CDN delivery | Proposed; not approved | Architect / Product / Legal | [ADR-0002](architecture/adr/ADR-0002-media-rights-and-delivery.md); confirm legal workflow, territory and provider signing model |
| ADR-0003 | PostgreSQL authority and transactional outbox for durable asynchronous events | Proposed; not approved | Architect | [ADR-0003](architecture/adr/ADR-0003-postgresql-authority-and-outbox.md); reconcile broker, delivery semantics and operations |

## Open decisions

- Launch markets, territories, and legal review process.
- Deployment/cloud provider, region, availability target, and cost envelope.
- Supported device/browser matrix, accessibility target, codecs, ladder, and DRM requirements.
- First-party vs federated identity, account verification, MFA, profiles, session policy.
- Ingestion model, allowed supplier integrations, upload permissions, rights evidence/approval process.
- Separation of duties for rights evidence: may the evidence submitter also approve the same permission, and under what exception/audit policy?
- Trusted territory source and maximum residual media-access window after rights expiry/revocation; playback remains denied until an authoritative source and bounded enforcement policy exist.
- Catalog scale, viewing volume, data retention, product analytics consent and privacy policy.
- GitHub repository ownership/access, issue publication, branching and PR protections.

Record each approved decision with context, options, consequences, security and operational implications, and an owner. Use `docs/architecture/adr/` for architecture decisions; reference its ADR ID here.
