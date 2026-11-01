# Security hardening

The local compose credentials are intentionally development-only and must not be used in a shared or production environment. No secret files are committed by this baseline.

Production readiness requires a managed secret store, least-privilege workload/database identities, private data services, non-root application containers, read-only filesystems where practical, dropped Linux capabilities, resource limits, pinned/scanned images, dependency and secret scanning, TLS, secure headers, audit logging, and an access/retention policy. Encrypt backups with managed keys and test key recovery.

Research must identify privacy, age, retention, residency, and contractual requirements; Architect must document trust boundaries and identity assumptions. QA must validate abuse, authorization, and failure cases. No compliance claim is made.
