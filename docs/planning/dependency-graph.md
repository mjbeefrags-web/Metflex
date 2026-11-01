# Dependency Graph

```mermaid
flowchart TD
  C1[CINE-001 Product brief/MVP] --> C6[CINE-006 Architecture baseline]
  C2[CINE-002 Users, regions, platforms] --> C6
  C3[CINE-003 Catalog, media, rights] --> C6
  C4[CINE-004 NFRs, scale, budget] --> C6
  C5[CINE-005 Legal, privacy, integrations] --> C6
  C6 --> M1[M1 Architecture and Engineering Foundation]
  M1 --> M2[M2 Backend and Database Foundation]
  M2 --> M3[M3 Authentication and Profiles]
  M2 --> M4[M4 Movie Catalog]
  M4 --> M5[M5 Content Ingestion and Licensing]
  M4 --> M6[M6 Search and Discovery]
  M5 --> M7[M7 Media Processing, if first-party processing selected]
  M5 --> M8[M8 Video Playback, eventual product capability]
  M7 --> M8
  M3 --> M9[M9 Watchlists and History]
  M6 --> M10[M10 Recommendations]
  M5 --> M11[M11 Admin Dashboard]
  M12[M12 Security and Performance] -. release gate .-> M13[M13 Production Deployment]
  M1 --> M13
```

## Blocking, parallel, critical path

CINE-001–005 are independent discovery deliverables and may run in parallel. CINE-006 blocks implementation and depends on all five. M1–M13 have no implementation-ready child issues. Arrows after CINE-006 show proposed sequencing and remain subject to approved contracts; they do not make any eventual product capability optional.

Critical path: CINE-001–005 → CINE-006 → M1 → approved MVP vertical slice → M12 security/performance gate → M13 production deployment. M0 is initialization and precedes M1. M7 first-party processing is conditional on delivery model; M8 playback is an eventual product capability, with MVP timing/provider pending. M9 and M10 are committed eventual capabilities; their detailed behavior and sequence depend on approved identity, catalog and privacy contracts.

## Risky dependencies

Rights/territory authorization, metadata terms/provenance, identity/retention, and cloud/deployment constraints can change architecture. Resolve before Builder work. Architect owns design; Research owns evidence; Product/Legal own product/policy decisions; QA owns independent verification; DevOps owns infrastructure/operations.
