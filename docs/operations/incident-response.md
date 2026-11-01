# Incident response

Until an on-call roster and deployment environment exist, this is a response outline, not a staffed escalation policy.

1. Declare incident severity, incident lead, communications owner, and operational scribe.
2. Establish customer impact, start time, affected services/data, and current evidence from health, logs, metrics, and recent changes.
3. Stabilize safely: stop rollout, disable a faulty feature if available, or roll back to a known-good immutable release. Preserve evidence and avoid destructive data changes.
4. Escalate security/privacy incidents through the approved response and notification process; do not disclose secrets or sensitive data in tickets/chats.
5. Confirm recovery with user-facing indicators and monitor for recurrence.
6. Write a blameless review with timeline, contributing factors, data impact, and owned corrective actions.

Assign contacts, paging thresholds, external notification rules, and service-specific runbooks before production launch.
