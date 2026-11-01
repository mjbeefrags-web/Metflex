# Deployment

## Readiness

There is no accepted production topology or application implementation in this repository, so no deployment procedure is currently safe to execute. Before first deployment, the owners must define the target environment, supported regions, availability objectives, data residency, media rights/delivery model, public domains, secrets provider, and operational ownership.

Required release gates: reviewed infrastructure change; pinned image build from a commit; dependency/security checks; QA sign-off; database migration compatibility review; health/readiness checks; backup verified; rollback artifact retained; and a post-deploy smoke/monitoring window. Container startup alone is not success.

## Rollout and rollback contract

Use an immutable release identifier and deploy the same artifact that passed CI. Roll out gradually when the selected platform supports it. Keep schema changes backward-compatible across at least the rollout and rollback window (expand, migrate, contract). Roll back application image/configuration to the last healthy release; do not automatically reverse destructive schema migrations. Restore data only under an approved recovery plan.

Platform-specific commands and approval owners remain open until the Architect selects the deployment topology.
