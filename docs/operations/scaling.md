# Scaling

There is no workload baseline, service implementation, or production telemetry from which to recommend capacity. Do not scale based on assumed future load.

Measure representative request latency/throughput, CPU/memory, database query and connection pressure, cache hit/eviction rate, queue depth/task age, egress, and media bandwidth/storage cost. Set load profiles and acceptance targets with product and QA.

Potential constraints to evaluate: stateless web replicas and session storage; database connection pooling and query/index efficiency; managed Redis availability and queue semantics; independent worker concurrency; CDN cacheability and rights enforcement; media egress/storage lifecycle cost; and network/provider limits. Add horizontal replicas only after verifying shared state, graceful shutdown, idempotent jobs, and dependency headroom. Keep results and thresholds in a dated capacity report.
