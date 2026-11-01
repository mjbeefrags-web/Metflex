# ADR-0001: Start with a Modular Monolith

- **Status:** Proposed; Architect approval required
- **Date:** 2026-10-31
- **Context:** CINEFLOW has no existing implementation, known traffic profile, deployment target, or team ownership split. Its catalog, profiles, rights, playback grants and jobs need clear domain boundaries and several transactional invariants.
- **Decision proposal:** Deploy one FastAPI application with explicit domain modules and PostgreSQL as the system of record. Run media workers as separate processes because workload/resource isolation differs. Extract an API domain into a service only when operational measurements or ownership needs justify it.
- **Alternatives:** Separate microservices immediately; single undifferentiated application package.
- **Consequences:** Easier transactions, local development and operations than service-per-domain; module boundaries and dependency rules must be maintained. Independent domain deployment/scaling is not initially available.
- **Security/performance:** Centralized policy points improve reviewability; horizontal API replicas remain possible if stateless. Isolated workers constrain media parser and CPU exposure.
- **References:** [Research decision R-001](../../research/technical-decisions.md#r-001--start-as-a-modular-monolith).
- **Confidence:** High for absent scaling evidence.
- **Issue:** None supplied.
