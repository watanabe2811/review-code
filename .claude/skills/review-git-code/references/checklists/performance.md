# Performance Review Checklist

## General (Python & Java)

- [ ] No N+1 query pattern: a collection is fetched, then a query runs per item in a loop instead of one eager-loaded/batched query
- [ ] List/collection endpoints or queries are paginated or otherwise bounded — no unbounded `SELECT *`/`findAll()` on a table that grows
- [ ] Aggregation/grouping/joining happens in the database, not by pulling all rows into memory and looping in application code
- [ ] No blocking I/O call on a thread or event loop that also serves concurrent requests (see Async subsections below)
- [ ] Expensive/repeated computations or large deserialized objects are cached rather than recomputed per call, where correctness allows it
- [ ] String concatenation in tight loops uses a builder/join, not `+=`
- [ ] No unbounded queues, thread pools, or in-memory collections that grow without an eviction/backpressure mechanism

## FastAPI (async correctness)

- [ ] No blocking call (`requests`, `time.sleep`, sync DB driver, sync `boto3`) inside an `async def` route — this stalls every concurrent request on the shared event loop, not just the caller's
- [ ] Native-async SDKs preferred over sync ones (`httpx.AsyncClient` not `requests`, `asyncpg`/async SQLAlchemy not `psycopg2`, `redis.asyncio` not `redis-py`)
- [ ] No `asyncio.run()`, manual event loop, or manually spawned thread inside a route — signals sync code bolted onto the loop
- [ ] `def` routes / `run_in_threadpool` used only as a bounded last resort for occasional unavoidable blocking calls, not on hot paths (the threadpool has a hard cap; saturating it collapses throughput for everyone)
- [ ] CPU-bound work is offloaded to a separate worker process (Celery/Arq/RQ/`multiprocessing`), not run on the loop or threadpool — the GIL means a threadpool doesn't help CPU-bound work either
- [ ] No unawaited coroutines; short fire-and-forget work uses `BackgroundTasks`, long/retry-critical work uses a real task queue
- [ ] DB session is request-scoped via dependency injection, not a shared module-level session
- [ ] Relationships accessed inside a loop are eager-loaded (`selectinload`/`joinedload`), not lazy-loaded per iteration

## Spring Boot / JPA

- [ ] No `FetchType.EAGER` on collections that are frequently unused, and no lazy-relationship access inside a loop without `JOIN FETCH`/`@EntityGraph`/`@BatchSize`
- [ ] Read-only transactions are marked `@Transactional(readOnly = true)` where applicable (avoids unnecessary dirty-checking overhead)
- [ ] `@Transactional` is on a public method of a Spring-managed bean, called through the proxy — not on a `private` method or via same-class self-invocation, where the annotation silently does nothing
- [ ] I/O-bound work considers virtual threads (Java 21+, `spring.threads.virtual.enabled`) instead of a bounded platform-thread pool, where the runtime supports it
- [ ] Entities avoid Lombok `@Data` (its generated `equals`/`hashCode` can touch every field, including lazy ones, triggering unwanted loads or `StackOverflowError` on bidirectional associations)

## PySpark

- [ ] Wide transformations (`groupBy`, `join`, `distinct`, `repartition`) are minimized/ordered to reduce shuffle volume; filters and projections pushed before joins where possible
- [ ] Skewed joins (one key dominating) are addressed (salting, broadcast join, or `AQE` skew handling) rather than left to stall a single task
- [ ] Small-table joins use broadcast join (`broadcast(df)`/`spark.sql.autoBroadcastJoinThreshold`) instead of a full shuffle join
- [ ] `.cache()`/`.persist()` is used deliberately for DataFrames reused across multiple actions, and unpersisted when no longer needed — not applied blindly, and not missing where it's clearly needed
- [ ] No `.collect()` / `.toPandas()` on a DataFrame that isn't guaranteed small — pulling full distributed data to the driver risks driver OOM
- [ ] Partition count is sized to the cluster (`repartition`/`coalesce`) — not left at a default that creates too many tiny tasks or too few oversized ones
- [ ] UDFs are avoided where a native Spark SQL function/column expression would do the same job (UDFs block Catalyst optimization and are serialization-expensive)

## Flink

- [ ] Checkpointing interval and mode (exactly-once vs at-least-once) are configured deliberately, not left at framework defaults, for the job's latency/throughput tradeoff
- [ ] State backend choice (heap vs RocksDB) matches state size — large keyed state on a heap backend risks GC pressure and OOM
- [ ] Watermark strategy and allowed lateness are set deliberately for event-time processing, not left to accumulate unbounded late-event state
- [ ] Backpressure is monitored/addressed at its source (slow sink, skewed key distribution) rather than papered over with a larger buffer
- [ ] Custom serializers (POJO/Avro/Kryo) are used deliberately — falling back to Kryo for complex generic types is a known serialization-cost trap

## ETL sync jobs

- [ ] Sync runs incrementally (watermark/cursor-based) rather than re-processing the full dataset every run, unless a full sync is genuinely required
- [ ] Batch sizes for reads/writes to source/destination systems are tuned (not row-by-row where a bulk API exists)
- [ ] Retries use backoff (not a tight retry loop hammering a struggling downstream system)

## SQL / data-access

- [ ] Indexes exist on columns used in `WHERE`/`JOIN`/`ORDER BY` for hot query paths
- [ ] Queries select only needed columns, not `SELECT *`, where the difference matters for a wide table
- [ ] Large tables have a `LIMIT`/pagination on any query that could return unbounded rows

## Docker

- [ ] Multi-stage builds are used to keep the final image lean (build tools not shipped in the runtime image)
- [ ] Image layers are ordered so that rarely-changing dependencies are cached (dependency install before application code copy)

## CI/CD

- [ ] Pipeline steps that could run in parallel (independent lint/test/build jobs) are not forced serial
- [ ] Dependency/build caching is configured where the CI platform supports it, to avoid redundant full reinstalls on every run
