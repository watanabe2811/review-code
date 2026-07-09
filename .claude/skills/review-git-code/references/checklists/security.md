# Security Review Checklist

## General (Python & Java)

- [ ] All SQL uses parameterized queries / bound parameters — never f-string, `%`-format, `+`-concatenation, or a raw ORM escape hatch (`text(f"...")`, `.extra()`, `createNativeQuery(...+...)`) fed with user input
- [ ] Dynamic identifiers (table/column names, sort direction) are validated against a whitelist, never interpolated directly — placeholders can't bind identifiers
- [ ] No hardcoded credentials, API keys, tokens, or secrets in source — must come from env/config/secrets manager
- [ ] User-controlled input passed to filesystem paths, shell execution, or URL redirects is validated (path traversal, command injection, open-redirect)
- [ ] A declared authentication check is not treated as an authorization check — verify ownership/role/permission explicitly for the specific resource being touched, not just "is this caller logged in"
- [ ] Error responses don't leak internals (stack traces, SQL, file paths, secrets)
- [ ] CORS/CSP is not overly broad; specifically never `allow_origins=["*"]` combined with `allow_credentials=True`
- [ ] Deserialization of untrusted input avoids unsafe mechanisms (`pickle.loads` on external data, Java native `ObjectInputStream` on untrusted streams)
- [ ] Dependencies are pinned and free of known CVEs where checkable (lockfile present, no `*`/unbounded version ranges for security-sensitive packages)

## FastAPI

- [ ] Every mutating/sensitive route has an explicit authorization check beyond `Depends(get_current_user)` — e.g. an `owned_x`/role-check dependency, not just presence of a user
- [ ] Input/output use distinct Pydantic models; the ORM model is never used directly as `response_model` (leaks fields like `hashed_password`)
- [ ] `response_model` is set on every route returning DB-backed data, so unintended fields can't leak
- [ ] Constraints (`Field(gt=...)`, `EmailStr`, etc.) validate at the boundary before any DB write — not caught only by a later DB constraint
- [ ] Secrets/config loaded via `Settings`/env, not hardcoded

## Spring Boot

- [ ] Constructor injection used (not field `@Autowired`) — makes required collaborators explicit and testable, incidental to security but affects whether auth checks are easy to unit test
- [ ] `@PreAuthorize`/method-security or equivalent explicit authorization annotation guards sensitive service/controller methods, not just `@RestController` reachability
- [ ] Actuator endpoints (`/actuator/**`) are not exposed publicly without authentication, especially `/env`, `/heapdump`, `/beans`
- [ ] Native/raw JPQL/SQL queries (`createNativeQuery`, `@Query(nativeQuery = true)`) use `:param` binding, never string concatenation
- [ ] Global exception handler (`@RestControllerAdvice`) doesn't leak stack traces or internal exception messages to the client in production

## PySpark

- [ ] No secrets (S3/cloud credentials, DB passwords, API keys) hardcoded in job code or notebook cells — sourced from a secrets manager or the cluster's credential provider
- [ ] Data read from external/untrusted sources (uploaded files, third-party feeds) is schema-validated before being trusted, not blindly `inferSchema`-ed into downstream logic
- [ ] Output paths/table names built from job parameters are validated, not directly interpolated from an unvalidated upstream config
- [ ] PII/sensitive columns are not logged (`df.show()`, `.collect()` output shouldn't land in job logs for regulated data)

## Flink

- [ ] Secrets for sinks/sources (Kafka credentials, DB connection strings) are not hardcoded in job graph/config committed to the repo — sourced from a secrets store or the deployment's config injection
- [ ] Checkpoint/savepoint storage location isn't world-writable/world-readable if it may contain sensitive state
- [ ] Custom deserializers for untrusted input streams don't use unsafe deserialization (Java native serialization) of attacker-controllable bytes

## ETL sync jobs

- [ ] Credentials for source/destination systems are not hardcoded — sourced from config/secrets manager, and distinct per environment
- [ ] Data in transit between systems uses TLS where the transport supports it
- [ ] Idempotency: re-running a sync after a partial failure doesn't duplicate/corrupt data (see architecture-scalability.md for the broader idempotency discussion)
- [ ] Logs of failed records don't dump full PII/sensitive payloads in plaintext

## SQL / data-access

- [ ] Every hand-written query (raw SQL, ORM raw-SQL escape hatch) uses parameter binding
- [ ] Migrations don't hardcode secrets or environment-specific credentials
- [ ] Database user used by the application has least-privilege grants (no unnecessary `DROP`/`ALTER`/superuser)

## Docker

- [ ] Container does not run as `root` (explicit non-root `USER` in the Dockerfile)
- [ ] No secrets baked into image layers (`ARG`/`ENV` with real credentials, `COPY` of a `.env` file with real values) — secrets injected at runtime instead
- [ ] Base image is pinned to a specific version/digest, not `latest`
- [ ] `.dockerignore` excludes `.env`, credentials, `.git`, and other sensitive local files from the build context

## CI/CD

- [ ] Secrets used in pipeline steps come from the CI platform's secret store, never hardcoded in the pipeline YAML
- [ ] Third-party actions/steps are pinned to a commit SHA or specific version, not a mutable tag like `latest`/`main`
- [ ] Pipeline doesn't `echo`/log secret values (even accidentally via `set -x` with secrets in env)
- [ ] Security-relevant gates (dependency scanning, SAST) exist and are required, not merely present-but-optional
