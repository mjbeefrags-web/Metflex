# ADR-0002: Gate Media Publication and Deliver from Private Origin

- **Status:** Proposed; Architect and legal/product approval required
- **Date:** 2026-10-31
- **Context:** Product limits content to public-domain, openly licensed, or explicitly authorized media. HLS has multiple child resources, while media delivery volume should not consume API workers.
- **Decision proposal:** Persist rights scope/evidence separately from technical media readiness, with independently evaluated permissions for acquisition, temporary processing storage, permanent storage, derivative/transcode, CDN redistribution, and authorized third-party provider playback. Require relevant rights scope plus validated active asset before hosted publication/playback. Keep object-storage origin private; issue short-lived, narrowly scoped CDN access for hosted media. For provider mode, return only provider-authorized handoff/playback bootstrap and do not infer downloading, storing, transcoding, or rehosting rights.
- **Alternatives:** Public origin URLs; API proxying media; treating ingest success as proof of rights; treating third-party playback authorization as permission to rehost.
- **Consequences:** Adds rights-review workflow and provider-specific edge configuration; enables revocation and auditability while preserving CDN delivery efficiency. Provider playback paths may require vendor SDK/player and have different capabilities from HLS.js. Provider and exact token mechanism remain open.
- **Security:** CDN tokens/URLs are bearer credentials; short TTL, scoped paths, log/referrer redaction, and origin access controls are required. Provider handoff tokens are similarly sensitive and subject to provider terms. Revocation delay is bounded by grant TTL/cache policy and must be documented.
- **References:** [Streaming research](../../research/streaming-research.md), [research decision R-003/R-004](../../research/technical-decisions.md).
- **Confidence:** High for conceptual separation; medium for vendor-specific delivery design.
- **Issue:** None supplied.
