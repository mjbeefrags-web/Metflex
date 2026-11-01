# Disaster recovery

No recovery objectives, hosting provider, data classification, or service dependencies are defined. Product and Architect must approve RPO/RTO and regional recovery requirements before a DR design can be considered complete.

The plan must inventory database, object/media storage, cache, queues, secrets/keys, DNS, container images, and infrastructure state; identify a separate recovery failure domain; define restore order and access; and provide a tested failover and return procedure. Redis cache may be rebuildable, but queue semantics and in-flight tasks must be verified before treating it as disposable. Preserve migration compatibility with the recovery image.

Run tabletop exercises and scheduled restore/failover rehearsals, record measured recovery time and data loss, and track corrective actions. There is no production DR capability yet.
