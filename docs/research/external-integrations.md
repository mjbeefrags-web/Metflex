# External Integrations

No provider or integration is currently configured or verified in this repository. Select integrations only after the Architect and product owner define regions, rights model, scale, privacy requirements, and budget.

## Candidate integration classes

| Need | Candidate boundary | Evaluation requirements |
|---|---|---|
| Metadata enrichment | Explicitly licensed metadata API | Attribution, caching/display terms, territory, quotas, freshness, image rights, deletion/correction process |
| Content source | Public-domain/open-license/authorized supplier | Chain of title or grant, territory, term, permitted streaming/transcoding, sublicensing, attribution, takedown, source authenticity |
| Object storage/CDN | Private object origin + CDN | Region, egress, requests, signed cookies/URLs, CORS, cache invalidation, logs, lifecycle, outage behavior |
| Identity provider | OIDC provider, optional | Data processing, MFA, account linking, redirect security, availability, exit/export path |
| Email delivery | Transactional email provider | SPF/DKIM/DMARC, suppression, privacy, bounce handling, region, cost |
| Telemetry backend | Prometheus/Grafana and/or OTel-compatible backend | Retention, access controls, PII minimization, cardinality, cost |

## Integration contract rules

All external calls need timeouts, bounded retries with jitter, circuit/load shedding where appropriate, schema validation, observability, and an explicit degraded-mode policy. Do not return fabricated successful metadata or playback URLs if an integration fails. Persist provider IDs and source timestamps, not provider responses without a retention/legal reason. Keep provider-specific code behind a small adapter where more than one provider is realistically planned.

## Licensing and distribution

The platform must retain evidence for each asset: rights holder/source, license/grant URL or document, permitted media use, geographic territories, start/end dates, attribution text, restrictions, proof reviewer, and revocation/takedown status. Public domain determination varies by jurisdiction and underlying work; open-source-style licenses may not grant all audiovisual, music, performer, or image rights. Obtain jurisdiction-specific legal review before distribution. No integration has been approved.
