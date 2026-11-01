# Container integration points

No backend/frontend source or build manifests exist yet. Add separate production Dockerfiles only after the Builder documents runtime, build output, non-root execution, startup, migration, and health-check contracts. Pin base images by supported version (prefer digest in release builds), use multi-stage builds, and keep build secrets out of image layers. Do not treat this note as a successful application containerization.

The local PostgreSQL and Redis services are defined in [`../infrastructure/compose.yaml`](../infrastructure/compose.yaml).
