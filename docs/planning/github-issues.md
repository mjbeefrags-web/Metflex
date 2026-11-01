# GitHub Issue Backlog

The initial repository inspection preceded the current documentation baseline. A root README, research baseline, architecture drafts/ADRs, and quality plans now exist. No application code or tests exist, and no issue has been published to GitHub. `CINE-001`–`CINE-006` are proposed local backlog identifiers pending owner assignment and repository/issue access.

## M0 discovery and decision issues

### CINE-001 — Approve MVP boundary, outcomes and measures against the kickoff brief
- **Description / business purpose:** Use the existing CINEFLOW kickoff brief as the product-vision source. Approve the first-release boundary, ordering, measurable MVP outcomes and success measures so delivery is focused without reopening committed eventual capabilities.
- **Technical context:** Discovery, accounts/profiles, authorized playback/HLS/subtitles, user library, admin/content-rights operations, recommendations and analytics are committed eventual capabilities. MVP order and detailed behavior are open; architecture and research drafts exist and need Product inputs/approval.
- **Dependencies / owner:** None; Product.
- **Implementation requirements:** Treat the kickoff brief as the source for eventual product capabilities; record which capabilities enter the MVP and their sequence; distinguish approved decisions from hypotheses; define outcomes/measures and link evidence; obtain Product/Director approval; update the CINE-001 decision input in `product-requirements.md` and keep detailed behavior aligned in `requirements.md`.
- **Acceptance criteria:** MVP boundary names included/deferred capabilities and rationale without making eventual core capabilities optional; measurable success outcomes are defined or explicitly assigned; approver/date/evidence recorded; planning docs match approved boundary.
- **Testing:** Product checklist verifies every proposed capability maps to an outcome and scope disposition.
- **Security:** No unnecessary personal data or unapproved tracking assumptions.
- **Documentation:** Link the kickoff brief and update `product-requirements.md`, `requirements.md`, and affected stories/roadmap.
- **Complexity:** M.
- **DoD:** Approved deliverable and linked evidence exist; unresolved questions have owners.

### CINE-002 — Confirm launch users, regions, platforms and journeys
- **Description / business purpose:** Validate roles, audience, markets/platforms, localization/accessibility expectations and primary journeys.
- **Technical context:** These choices affect identity, catalog, rights, UI and deployment.
- **Dependencies / owner:** CINE-001; Product, with Research.
- **Implementation requirements:** Document roles, journeys, region/platform matrix, evidence/confidence, exclusions and legal unknowns.
- **Acceptance criteria:** Approved role list; journey per in-scope role; launch matrix; unknowns assigned; story candidates updated.
- **Testing:** Product walkthrough; QA confirms approved journeys have observable outcomes.
- **Security:** Flag age/child-user requirements for evidence-backed legal review.
- **Documentation:** Update requirements and stories.
- **Complexity:** M.
- **DoD:** Approval and source references recorded; hypotheses are not presented as facts.

### CINE-003 — Verify catalog sources and content/rights operating model
- **Description / business purpose:** Authorized movie playback/adaptive streaming is part of the product vision. Determine the delivery model (first-party hosting/processing or authorized third-party source), verify metadata/media sources, licenses, provenance, rights workflows and revocation behavior, and establish which titles/territories can be offered. This issue decides delivery and rights specifics, not the product-level streaming commitment.
- **Technical context:** Metadata availability is not streaming rights evidence.
- **Dependencies / owner:** CINE-001, CINE-002; Research with Product, Legal and content operations.
- **Implementation requirements:** For each offered title/media asset, record source, date checked, permitted use, attribution, territorial limits, confidence, refresh/correction flow; obtain explicit rights policy and define denial behavior for unsupported titles.
- **Acceptance criteria:** Approved content/delivery model and source/rights matrix; evidence/confidence recorded; deny/escalation outcome for unknown, expired and revoked rights; legal questions assigned; authorized playback remains in product scope while individual title availability is evidence-based.
- **Testing:** Research review; QA drafts boundary cases for any distribution path.
- **Security:** Keep credentials and confidential contracts out of public docs; limit rights evidence access by policy.
- **Documentation:** Update requirements and provide Architect a linked research brief.
- **Complexity:** L.
- **DoD:** Product/Legal approval recorded; no rights assumption remains implicit.

### CINE-004 — Set quality targets, capacity assumptions and constraints
- **Description / business purpose:** Establish measurable SLO/SLI, latency, load/growth, recovery, accessibility, client, budget, team and delivery constraints.
- **Technical context:** These drive deployment, caching, storage and reliability choices.
- **Dependencies / owner:** CINE-001, CINE-002; Product and DevOps by domain.
- **Implementation requirements:** Define metric, threshold, measurement window, launch/growth assumptions, budget/team constraints, evidence and approver.
- **Acceptance criteria:** Each target is numeric or explicitly deferred with decision owner; measurement method is named; DevOps records feasibility/risks.
- **Testing:** QA confirms targets have verification methods; no performance test is required for discovery.
- **Security:** Include security, recovery and incident response requirements.
- **Documentation:** Update NFRs and handoff to Architect/DevOps.
- **Complexity:** M.
- **DoD:** Approved target sheet and owned open items exist.

### CINE-005 — Define legal, privacy, data, identity, payment, analytics and integration requirements
- **Description / business purpose:** Confirm jurisdictions, data collection/retention/deletion, identity/payment need, analytics, integrations and residency.
- **Technical context:** These affect data model, providers, security contracts and deployment.
- **Dependencies / owner:** CINE-001, CINE-002; Product with Research, Legal, Security and DevOps.
- **Implementation requirements:** Maintain requirement/source/owner/confidence matrix; identify data categories, purposes, retention/deletion, residency, consent and provider constraints; do not choose vendors without approval.
- **Acceptance criteria:** Approved integration/data inventory; identity/payment/analytics explicitly in/out; retention/deletion/residency documented or assigned; review owner named.
- **Testing:** QA checks future verification evidence; Security reviews threat/privacy inputs.
- **Security:** Minimize collected data and protect sensitive sources.
- **Documentation:** Update NFR, product and architecture input docs.
- **Complexity:** L.
- **DoD:** Decision owners, sources and blockers recorded; no vendor/policy invented.

### CINE-006 — Publish and approve architecture baseline from verified inputs
- **Description / business purpose:** Architect translates approved requirements/research into architecture artifacts and contracts so Builders have a stable baseline.
- **Technical context:** Architecture drafts, data model, system design and ADR proposals exist under `docs/architecture/`; approval and reconciliation with product/legal/operational inputs remain pending.
- **Dependencies / owner:** CINE-001–005; Architect.
- **Implementation requirements:** Complete applicable system/context/container/component, domain/data, backend/frontend, API, media/event/cache, security, scale/failure and deployment docs; mark non-applicable artifacts with rationale; create accepted ADRs for consequential decisions; record open questions and validation evidence.
- **Acceptance criteria:** README status updated; MVP capabilities map to module/data owners/contracts; security, rights, failures and deployment addressed; traceability to approved inputs; Product and relevant domain owners review; Architect approves.
- **Testing:** Architecture/contract consistency review; QA and DevOps confirm verification and operations needs are represented.
- **Security:** Document trust boundaries, authorization, secrets, privacy, threat and failure handling.
- **Documentation:** Architecture source of truth and ADRs updated in repository.
- **Complexity:** XL.
- **DoD:** Accepted baseline exists and no unresolved decision blocks proposed MVP issues. This issue creates a follow-up implementation backlog; it does not implement features.

## Gated tracks

Do not create implementation issues for M2–M13 until CINE-006 and the M1 architecture gate are approved. Preserve the kickoff milestone IDs/names in `milestones.md`. Each child issue must identify one outcome, approved contracts/modules, dependencies, measurable acceptance, test evidence, security and documentation. Avoid broad or duplicate placeholders such as “build backend” or “add streaming.”
