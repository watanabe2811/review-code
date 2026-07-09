# Architecture & Scalability Review Checklist

## General (Python & Java)

- [ ] SOLID red flags: a class/module with more than one reason to change, a "Manager"/"Handler"/"Processor" grab-bag class, an `if/else`/`switch` chain that grows with every new type instead of using polymorphism/strategy
- [ ] Dependency direction is correct: domain/business logic doesn't import infrastructure (DB drivers, HTTP clients, framework classes) directly — infrastructure depends on domain-defined interfaces, not the other way around
- [ ] High-level modules depend on abstractions (injected interfaces), not concrete implementations constructed with `new`/direct instantiation buried in business logic
- [ ] No obvious circular dependencies between modules
- [ ] Config is externalized (env/config file), not hardcoded, so behavior can change per environment without a code change
- [ ] Horizontal scalability: the service holds no in-process state (session affinity, local cache as source of truth) that would break running multiple instances behind a load balancer
- [ ] Database connections/HTTP clients are pooled, not created per call
- [ ] Design patterns are used to solve an actual demonstrated problem, not applied for their own sake (watch for a single-implementation interface, or a simple `if/else` replaced by a Strategy+Factory+Registry with no real variability)

## FastAPI

- [ ] Business logic lives behind `Depends`/services, not inline in route functions — keeps routes thin and testable
- [ ] Singletons (HTTP clients, connection pools) are created once in the `lifespan` context, not re-created per request
- [ ] Existence/permission checks are expressed as small composable dependencies (cached per request) rather than duplicated inline in every route that needs them

## Spring Boot

- [ ] Layering is respected: Controller → Service → Repository, with no controller reaching directly into repository/DB access, and no repository containing business rules
- [ ] Package structure organizes by feature/domain rather than purely by technical layer, if the codebase is large enough for that to matter
- [ ] Configuration/properties classes are used instead of scattering `@Value` across unrelated classes

## PySpark

- [ ] Job is structured as composable, testable transformation functions rather than one monolithic script mixing I/O, config, and transformation logic
- [ ] Partitioning strategy (both read-time partition pruning and in-job `repartition`/`coalesce`) is deliberate for the data volume, not left to accidental defaults
- [ ] The job degrades reasonably as data volume grows — no assumption baked in that a DataFrame will always fit comfortably (e.g. no unconditional `.collect()`/`.toPandas()` on data whose size scales with input)
- [ ] Checkpointing/intermediate persistence is used for long transformation chains where recomputation cost or lineage depth would otherwise be a problem

## Flink

- [ ] Job parallelism is configured deliberately relative to expected throughput and available task slots, not left at a default that under- or over-provisions
- [ ] State size growth is bounded (TTL on keyed state where appropriate) rather than growing unboundedly over the job's lifetime
- [ ] The job's scaling story (rescaling via savepoints, parallelism changes) is considered if throughput requirements are expected to grow

## ETL sync jobs

- [ ] **Idempotency**: re-running the sync after a partial failure does not duplicate or corrupt data — upserts/natural keys/dedup logic, not blind appends
- [ ] Sync is incremental (watermark/cursor/CDC-based) where the data volume makes full re-sync impractical, with a documented fallback to full sync when needed
- [ ] Transactional boundaries around multi-step writes to the destination are clear — a failure partway through doesn't leave the destination in an inconsistent intermediate state
- [ ] Schema drift from the source (new/removed/renamed columns) has a defined handling strategy, not an assumption that the source schema never changes
- [ ] The sync's scaling story is considered as source/destination data volume grows — batch size, parallelism, and destination write capacity are not hardcoded assumptions that only hold at today's volume

## SQL / data-access

- [ ] Schema/migration changes are backward-compatible with the currently-deployed application version where zero-downtime deploys are required (e.g. don't drop a column the old version still reads)
- [ ] Indexing strategy is deliberate for the table's actual query patterns, not just the primary key

## Docker

- [ ] Image is structured for the deployment target (appropriately sized, doesn't bundle unrelated services in one container)
- [ ] Configuration that varies by environment is injected at runtime (env vars, mounted config), not baked into the image per-environment

## CI/CD

- [ ] Pipeline scales reasonably with repo growth (e.g. it doesn't rebuild everything from scratch when only isolated changes were made, for monorepos where that matters)
- [ ] Deployment strategy (rolling/blue-green/canary) is appropriate for the service's availability requirements, not a single default assumed to always be safe
