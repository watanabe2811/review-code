# Testing / Testability Review Checklist

## General (Python & Java)

- [ ] New/changed non-trivial logic has tests, not just the happy path
- [ ] Failure paths and boundary conditions are covered — empty collections, null/None, invalid input — not only the success case
- [ ] Tests assert actual behavior/output, not just "a mock was called" — a test that only confirms its own setup proves nothing about the code under test
- [ ] Tests are not flaky/non-deterministic (no unmocked real time, unseeded randomness, or real network calls in unit tests)
- [ ] Mocks of external responses are complete enough to be meaningful — a partial mock that omits fields the code actually reads passes in the test and fails in production
- [ ] Hard-to-test code (impossible to unit test without mocking everything) is treated as a design smell — usually points at missing dependency injection

## FastAPI

- [ ] Suspected bugs (especially authorization bugs) are reproduced with a failing test before being reported as a finding — write the test asserting secure/correct behavior, confirm it fails against the current code, and cite that as evidence
- [ ] Tests use `app.dependency_overrides` to swap auth/DB dependencies with fakes, rather than `unittest.mock.patch`-ing internals
- [ ] `dependency_overrides` are reset between tests (teardown) so state doesn't leak across the suite
- [ ] Async app tests use an async client (`httpx.AsyncClient` + `ASGITransport`) consistently, avoiding event-loop mismatch issues
- [ ] Auth/authorization-sensitive routes have tests for the 401/403/404/422 paths, not only the 200/201 happy path

## Spring Boot

- [ ] Unit tests use Mockito (`@ExtendWith(MockitoExtension.class)`, `@Mock`/`@InjectMocks`) rather than spinning up the full Spring context (`@SpringBootTest`) for logic that doesn't need it
- [ ] Integration tests that need a real datastore use Testcontainers rather than mocking the database away entirely
- [ ] `@Transactional` test behavior (rollback semantics) is understood and intentional, not accidentally masking a bug that only appears outside a test transaction

## PySpark

- [ ] Transformation logic is tested against small in-memory/local DataFrames (not requiring a full cluster) so tests run fast and deterministically
- [ ] Tests cover schema edge cases the job will actually see in production (nulls, unexpected types, empty partitions), not just a clean happy-path sample
- [ ] Where feasible, transformation functions are structured to be testable in isolation from Spark session setup (pure functions over Rows/columns where possible)

## Flink

- [ ] Stateful operators/`ProcessFunction`s are tested with Flink's test harnesses (e.g. `OneInputStreamOperatorTestHarness`) covering watermark/timer behavior, not only plain unit tests of embedded pure-function logic
- [ ] Tests cover late-data and out-of-order event handling, not only in-order happy-path streams

## ETL sync jobs

- [ ] Sync logic is tested for partial-failure and retry scenarios — what happens if the job dies mid-batch and is re-run (idempotency), not just the clean full-success path
- [ ] Schema-drift handling (an unexpected new/missing column from the source) has a test, if the job is expected to tolerate it

## SQL / data-access

- [ ] Query-count assertions exist for critical list/read paths known to be N+1-prone (e.g. `CaptureQueriesContext` in Django, a SQL-count assertion in Java tests), not just "it returns the right data"
- [ ] Migrations are tested for reversibility/backward compatibility where the project requires zero-downtime deploys

## Docker

- [ ] If the project has a build/CI step that builds the image, it's actually exercised (not a Dockerfile that's never built in CI and silently rots)

## CI/CD

- [ ] Tests actually run in the pipeline and are a required (not optional/allowed-to-fail) gate before merge
- [ ] Coverage thresholds, if configured, are enforced (`--cov-fail-under`, JaCoCo minimum, etc.), not just reported
