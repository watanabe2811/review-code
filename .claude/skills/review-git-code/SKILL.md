---
name: review-git-code
description: |
  Clones a Git repository and produces a structured code review report covering
  security, performance, style/maintainability, test coverage, error handling/
  logging, and architecture/scalability. Targets Python and Java stacks: FastAPI
  APIs, Spring Boot services, PySpark jobs, Flink jobs, ETL sync jobs, SQL/data-
  access code, Docker, and CI/CD pipelines.
  Use when: the user gives a git URL and asks to review/audit it, "review this
  repo", "clone and review", "audit this codebase from git", "review git code".
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
---

# Review Git Code

Clone a Git repository, review its code against `references/checklists/`, and
write one Markdown report to `review/`.

## Inputs

- `<git-url>` — required.
- `[branch-or-tag]` — optional; defaults to the repo's default branch.

## Procedure

### 1. Clone or update

```bash
repo_name=$(basename -s .git "<git-url>")
if [ -d "input/$repo_name/.git" ]; then
  git -C "input/$repo_name" fetch --all
  git -C "input/$repo_name" checkout "<branch-or-tag>"   # if given
  git -C "input/$repo_name" pull
else
  git clone <git-url> "input/$repo_name"
  [ -n "<branch-or-tag>" ] && git -C "input/$repo_name" checkout "<branch-or-tag>"
fi
short_hash=$(git -C "input/$repo_name" rev-parse --short HEAD)
```

Git is assumed already installed and authenticated on this machine — do not
attempt to manage credentials.

### 2. Read project structure

List the repo tree (respecting `.gitignore`) to build a mental map before
reading individual files — e.g. `git -C input/$repo_name ls-files | head -300`
or a targeted `find`/`Glob`. This map drives stack detection (step 3) and the
large-repo guard (step 5).

### 3. Detect stack

Check for these signals (a repo may match several):

| Signal | Evidence |
|---|---|
| Python | `*.py`, `requirements.txt`, `pyproject.toml` |
| Java | `*.java`, `pom.xml`, `build.gradle` |
| FastAPI | `fastapi` import / dependency |
| Spring Boot | `spring-boot` dependency in `pom.xml`/`build.gradle` |
| PySpark | `pyspark` import, `spark-submit` config |
| Flink | Flink deps in `pom.xml`/`build.gradle`, or `pyflink` import |
| ETL sync | batch scripts moving data between systems on a schedule (cron config, Airflow DAGs, standalone sync scripts) |
| SQL / data-access | `.sql` files, a migrations directory, ORM usage (SQLAlchemy, JPA/Hibernate) |
| Docker | `Dockerfile`, `docker-compose.yml` |
| CI/CD | `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile` |

**No-match fallback:** if the repo matches none of these (e.g. Go, plain
JS/TS), don't refuse. Run only the language-agnostic parts of each checklist
(the "General" sections) and say so plainly in the report's Scope &
Limitations section, with reduced-confidence findings.

### 4. Detect and run static-check tooling (best-effort)

Only if the repo already has the relevant config **and** the tool is already
installed on this machine — never install anything:

| Stack | Config to look for | Tool | Check before running |
|---|---|---|---|
| Python | `ruff`/`flake8`/`pylint` config, `mypy` config, `bandit` config | `ruff`/`flake8`/`pylint`/`mypy`/`bandit` | `command -v <tool>` |
| Java | `checkstyle`/`spotbugs`/`pmd` in `pom.xml`/`build.gradle` | `checkstyle`/`spotbugs`/`pmd` | `command -v <tool>` |
| Docker | a `Dockerfile` exists | `hadolint` | `command -v hadolint` |
| CI/CD | GitHub Actions workflows | `actionlint` | `command -v actionlint` |

Run whatever is available via `Bash` and capture the output. If nothing is
available, state plainly in the report that no static analysis ran — never
fabricate tool output.

### 5. Load checklists

Read only the checklist sections relevant to what step 3 detected — the
"General" section of each file plus the matched stack subsections — not all
six files in full, to control token usage:

- `references/checklists/security.md`
- `references/checklists/performance.md`
- `references/checklists/style-maintainability.md`
- `references/checklists/testing.md`
- `references/checklists/error-handling-logging.md`
- `references/checklists/architecture-scalability.md`

### 6. Large-repo guard

If the repo exceeds **300 source files**, don't try to read everything.
Prioritize application/business-logic source directories over vendored,
generated, build-output, and test-fixture directories. Sample the core paths
and say so explicitly in the report's Scope & Limitations section — don't
silently skip files without noting it.

### 7. Deep review by layer

Identify and read closely — not skim — the files in each architectural layer
present in the repo:

- **Entrypoint** — `main.py`, `Application.java`, `manage.py`, job entry scripts
- **API / Controllers** — route handlers, `@RestController` classes
- **Services** — business logic
- **Repositories / Data Access** — DB/ORM access code
- **Config** — settings files, env templates, `application.yaml`
- **Infra** — `Dockerfile`, CI/CD pipeline files
- **Tests**
- **General** — cross-cutting concerns that don't belong to one layer (dependency management, documentation, overall architecture)

Skip layers that aren't present. Evaluate each file you read against the
loaded checklists, noting concrete findings with file:line references as you
go rather than trying to reconstruct them from memory afterward.

### 8. Write the report

Write to `review/<repo-name>_<short-hash>_<YYYYMMDD_HHMMSS>.md`:

```markdown
# Code Review: <repo-name> @ <short-hash> (<branch-or-tag>)

## Summary
<2-4 sentences: what the repo is, overall health, top priorities>

## Stack Detected
<table or list of matched signals from step 3>

## Static Analysis Run
<tool, version, exit status, raw issue count per tool — or "No static
analysis tooling was configured/available in this repo; findings below are
from manual review only.">
<This section is a factual log only — do not restate individual issues here;
every concrete issue appears exactly once, below, in its module section.>

## Findings

### Entrypoint
<findings, or omit section if not present>

### API / Controllers
...

### Services
...

### Repositories / Data Access
...

### Config
...

### Infra
...

### Tests
...

### General
...

<Each finding:>
- **[Severity] Category** — `file:line`
  <description>
  **Recommendation:** <fix>
  **Source:** <tool name, or "manual">

## Scope & Limitations
<sampling applied (if any), stack signals not matched, tools not available>
```

Severity scale: **Critical** (security/data-loss/crash-level) / **High** /
**Medium** / **Low**. Category is one of: Security, Performance,
Style/Maintainability, Testing, Error Handling/Logging, Architecture/
Scalability. A positive or neutral observation worth recording (a pattern
done well, a deliberate tradeoff) is not an issue — tag it **[Note]** instead
of a severity, so it can't be mistaken for something that needs fixing.

## Notes

- Never install tooling on the host.
- Never fabricate static-analysis output.
- Every finding needs a concrete `file:line` — no vague "somewhere in the
  service layer" claims.
- See `references/sources.md` for the 5 external skills this checklist set
  was synthesized from.
