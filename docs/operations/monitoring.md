# Monitoring

Do not deploy empty dashboards and call the system monitored. Define service-level indicators and objectives with product/architecture first, then instrument them.

At minimum, production telemetry should cover request rate, errors and latency; dependency health and saturation; database connection/query/replication health; Redis memory/evictions; worker queue depth and task age/failures; disk, CPU and memory; backup age and restore evidence. Use structured logs with request/trace IDs while excluding secrets and sensitive personal data. Define alert owners, severity, paging hours, and runbooks before enabling paging.

Prometheus/Grafana/OpenTelemetry choices and retention/export destinations remain open pending topology and privacy review.
