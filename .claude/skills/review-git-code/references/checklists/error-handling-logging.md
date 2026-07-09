# Error Handling & Logging Review Checklist

## General (Python & Java)

- [ ] No empty catch/except blocks — a swallowed exception hides a bug silently
- [ ] Exceptions are caught at the narrowest type that's actionable, not the broadest (`except Exception`/`except:` in Python, catching `Exception`/`Throwable` in Java) unless there's a specific reason (e.g. a top-level handler) and it's re-raised/logged, not dropped
- [ ] Original exception context is preserved when wrapping/re-raising (`raise NewError(...) from e` in Python, constructing the new exception with `cause` in Java) — never a bare `raise RuntimeError("failed")` that discards the real cause
- [ ] Exceptions aren't used for ordinary control flow (e.g. signaling "not found") where a return value/`Optional`/sentinel is clearer
- [ ] Logging uses a structured logger (`logging`/SLF4J), never `print()`/`System.out.println()`/`e.printStackTrace()`
- [ ] Log messages include enough context (identifiers, operation) to debug without reproducing locally, but never log secrets, full PII payloads, or credentials
- [ ] Resources (files, sockets, DB connections, HTTP clients) are released on all paths including exceptions — context managers/`try-with-resources`/`try/finally`, not manual close calls that get skipped on error

## FastAPI

- [ ] `HTTPException` details returned to the client don't leak internals (stack traces, SQL fragments, file paths)
- [ ] `yield`-dependencies holding a resource (DB session, file handle, lock) release it via `async with`/`try-finally` so an exception in the route doesn't leak it
- [ ] A global/route-level exception handler exists for unhandled exceptions so the client gets a clean error response rather than a raw traceback

## Spring Boot

- [ ] A `@RestControllerAdvice`/`@ExceptionHandler` centralizes error responses (ideally via `ProblemDetail`) rather than scattering try/catch-and-format logic across controllers
- [ ] Custom exceptions form a sensible hierarchy (not everything extending bare `RuntimeException` with no semantic distinction)
- [ ] `@Transactional` methods that catch an exception internally don't accidentally suppress the rollback the caller expects

## PySpark

- [ ] Job-level failures (bad input schema, missing partition, corrupt file) fail loudly with a clear error rather than silently producing an empty/partial output
- [ ] Per-record error handling (e.g. malformed rows) has an explicit strategy — quarantine/dead-letter, log-and-skip, or fail-fast — rather than an unhandled exception aborting the whole job on one bad row, or a silent `try/except: pass` masking data quality issues
- [ ] Logging from executors is understood to not always reach the driver console — critical diagnostics use accumulators/structured output the driver can actually see, not just `print()` inside a UDF

## Flink

- [ ] Failure/restart strategy is configured deliberately (restart policy, max attempts) rather than left to silently retry forever or fail without recovery
- [ ] Errors in a `ProcessFunction`/operator don't silently drop events — either fail the job (if correctness-critical) or route to a side output for later inspection
- [ ] Checkpoint failures are surfaced/alertable, not silently ignored

## ETL sync jobs

- [ ] Partial-batch failures are handled with a defined strategy (fail whole batch / skip-and-log bad records / dead-letter queue), not an unhandled exception that leaves the sync in an undefined state
- [ ] Retries use backoff and a bounded attempt count, not an infinite tight loop
- [ ] Failures are logged/alerted in a way that's actually monitored, not just written to a log file nobody reads

## SQL / data-access

- [ ] Transaction boundaries wrap related multi-statement operations so a mid-sequence failure rolls back cleanly, not leaving partial writes
- [ ] Connection/query errors are distinguished from "no rows found" — the latter isn't treated as an exception path unless the caller genuinely requires exactly one row

## Docker

- [ ] Container entrypoint fails fast and with a clear exit code/log line on startup misconfiguration (missing required env var), rather than starting in a broken state that fails obscurely later

## CI/CD

- [ ] Pipeline failures produce actionable log output (not just a red X with no context)
- [ ] Failing steps stop the pipeline (no `continue-on-error`/`allow_failure` on steps that should actually block, like tests or security scans)
