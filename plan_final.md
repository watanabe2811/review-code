# Plan (final): Git Code Review Skill

Refined from `plan.md`. Changes from the first draft are called out inline as
**(refined)**. A second round — expanded stack detection, static checks,
layered deep-review, per-module reporting — is marked **(amended)**. A third
round, fixing structural/risk issues found on re-review, is marked **(fix)**.

## Deliverable manifest **(fix: reference-skill copies moved out of the committed tree)**
```
review-code/
├── CLAUDE.md                                   # new
├── README.md                                   # new
├── .gitignore                                  # new
├── input/.gitkeep                              # new (clone target, gitignored contents)
├── review/.gitkeep                             # new (report output, gitignored contents)
├── research/reference-skills/                  # new — gitignored working cache, NOT committed
│   └── <source-1..5>/
│       ├── reference.md                        # raw downloaded SKILL.md content
│       └── knowledge/...                       # supporting files that source's SKILL.md links to (references, checklists, scripts, examples)
└── .claude/skills/review-git-code/
    ├── SKILL.md                                # new — main skill
    └── references/
        ├── sources.md                          # new — the 5 sources: name, repo URL, stars, license, 1-line takeaway (committed — attribution only, no raw copies)
        └── checklists/
            ├── security.md
            ├── performance.md
            ├── style-maintainability.md
            ├── testing.md
            ├── error-handling-logging.md
            └── architecture-scalability.md
```
Each checklist file's per-stack subsections cover: FastAPI, Spring Boot, ETL
sync jobs, PySpark, Flink, SQL/data-access, Docker, CI/CD.

`.claude/skills/` (including `references/sources.md` and `checklists/`) is
committed — it's the actual deliverable. `input/`, `review/`, and
`research/reference-skills/` are gitignored: `research/reference-skills/`
holds raw third-party `SKILL.md` content pulled in only as a synthesis aid
**(fix: previously this raw content was marked "committed" while also being
called "not redistributed" — contradiction; committing verbatim third-party
content is redistribution regardless of intent, and nesting it as literal
`SKILL.md` files under `.claude/skills/` also risked Claude Code's skill
loader picking them up as real, invokable skills. Renaming to `reference.md`
and moving the whole cache outside `.claude/skills/` and out of git fixes
both problems at once).**

## Phase 0 — Project scaffolding
- `CLAUDE.md` + `README.md` (missing, required by global Rule 3).
- `input/`, `review/`, `research/reference-skills/` with `.gitkeep`; `.gitignore` ignoring `input/*`, `review/*`, `research/*` (keep `.gitkeep`s).

## Phase 1 — Research & download 5 reference skills **(refined: concrete method; fix: added code-search channel + non-committed storage)**
- Search via `WebSearch` for terms like `"SKILL.md" code review github`, `claude code skill code review`, `anthropic skill review`.
- **(fix)** Also use GitHub **code search**, not just repo search — `filename:SKILL.md review` via `gh api search/code` or WebSearch — since a relevant `SKILL.md` is often nested inside a larger "awesome-claude-skills"-style collection repo that repo-level search by name/topic won't surface.
- Cross-check candidates and rank by popularity using `gh search repos --sort=stars` where applicable (git is already configured on this machine, `gh` is commonly available alongside it — fall back to WebSearch-only ranking if `gh` isn't installed/authenticated).
- For the top 5, fetch the raw `SKILL.md` (`WebFetch` on the raw GitHub URL) and save it to `research/reference-skills/<source-name>/reference.md` — a **gitignored, non-committed working cache** used only to synthesize Phase 2 checklists **(fix)**.
- **(amended)** Each source's `SKILL.md` typically points to supporting knowledge files (e.g. `references/*.md`, checklists, scripts, examples) via relative links — fetch those too, not just the top-level file, and save them under `research/reference-skills/<source-name>/knowledge/` mirroring the source repo's relative paths. This gives Phase 2 the full body of guidance each source actually uses, not just its entry-point summary.
- Record each source's repo URL, star count, and license in `.claude/skills/review-git-code/references/sources.md` (this file, attribution only, is what actually gets committed).
- Fallback if fewer than 5 genuine Claude Code review-skills exist: fill remaining slots with well-known code-review checklists (Google Engineering Practices, OWASP Code Review Guide) and note the substitution explicitly in `sources.md`.

## Phase 2 — Design review checklists **(refined: detection + synthesis, not copy)**
- One file per dimension in `references/checklists/` (security, performance, style-maintainability, testing, error-handling-logging, architecture-scalability).
- Each file: a general Python/Java section + subsections per matched stack component: FastAPI, Spring Boot, ETL sync jobs, PySpark, Flink, SQL/data-access, Docker, CI/CD.
- Content is **synthesized** (paraphrased/organized) from the Phase 1 research cache plus known framework practices — not verbatim copied, to respect source licenses and keep things concise:
  - FastAPI: blocking I/O in async handlers, dependency-injection misuse, input validation via Pydantic, N+1 query patterns, auth/authz on routes.
  - Spring Boot: field injection vs constructor injection, `@Transactional` boundary correctness, exception-handling via `@ControllerAdvice`, blocking calls on request threads, actuator endpoint exposure.
  - PySpark: shuffle-heavy operations, partitioning, caching/persist misuse, broadcast joins, driver-side collects on large data.
  - Flink: checkpointing config, state backend choice, watermark/late-data handling, serialization cost, backpressure.
  - ETL sync: idempotency, retry/backoff, transactional boundaries, schema-drift handling, data validation at boundaries.
  - SQL/data-access: parameterized queries vs string-built SQL, missing indexes on hot query paths, N+1 ORM patterns, migration safety (backward-compatible schema changes).
  - Docker: running as non-root, multi-stage builds, pinned base image versions, secrets baked into image layers, `.dockerignore` hygiene.
  - CI/CD: secrets handling in pipeline config, whether lint/test/security-scan gates exist and are required (not just present), dependency-pinning in pipeline steps.

## Phase 3 — Author SKILL.md **(refined: explicit detection logic + scale guard; amended: stack breadth, static checks, layered review, per-module report; fix: folded former Phase 2.5 in, added no-match fallback, de-duplicated report structure, pinned threshold)**
- Frontmatter: `name: review-git-code`, `description` written to trigger on "review this repo/git url" style requests.
- Instructions (all of the following are runtime logic written into this one file — **(fix)** static-check detection is no longer a separate build phase, it's steps 6–7 below):
  1. Parse `<git-url>` and optional `[branch-or-tag]` from the invocation.
  2. Clone/update into `input/<repo-name>/` (fetch+checkout+pull if it already exists).
  3. Resolve short commit hash for the report filename.
  4. **Read project structure** — list the repo tree (respecting `.gitignore`) to establish a map before diving into files.
  5. **Detect stack** from repo contents, matching one or more of: Python, Java, FastAPI (`fastapi` import), Spring Boot (`spring-boot` dep in `pom.xml`/`build.gradle`), PySpark (`pyspark` import/`spark-submit` config), Flink (Flink deps or `pyflink` import), ETL sync (batch scripts moving data between systems on a schedule), SQL/data-access (`.sql` files, migrations dir, ORM usage), Docker (`Dockerfile`/`docker-compose.yml`), CI/CD (`.github/workflows`, `.gitlab-ci.yml`, `Jenkinsfile`).
     **(fix) No-match fallback:** if the repo matches none of these signals (e.g. Go, plain JS/TS), don't refuse — run only the general Python/Java-agnostic parts of each checklist (security, testing, error-handling, style basics that aren't language-specific) and state plainly in the report's scope section that stack-specific checklists didn't apply and confidence is lower.
  6. **Detect static-check tooling** already configured in the repo: Python (`ruff`/`flake8`/`pylint` config, `mypy` config, `bandit` config), Java (`checkstyle`/`spotbugs`/`pmd` in `pom.xml`/`build.gradle`), Docker (`hadolint` if a `Dockerfile` exists), CI/CD (`actionlint` for GitHub Actions). Only proceed if config exists **and** the tool is already installed on this machine (check with `command -v <tool>`) — never install new tooling.
  7. **Run detected tools** via `Bash` and capture their output. If nothing is available, state plainly in the report that no static analysis ran — never fabricate tool output.
  8. Load only the relevant checklist sections (general + matched stack signals) rather than all files blindly, to control token usage.
  9. **Large-repo guard:** prioritize source directories over vendored/generated/build/test-fixture directories; if the repo exceeds **300 source files** (fixed threshold), sample the core application/business-logic paths and note the sampling explicitly in the report's scope section rather than silently skipping files.
  10. **Deep review by layer** — identify and read closely, not just skim, the files belonging to each architectural layer present in the repo: entrypoints (`main.py`, `Application.java`, `manage.py`, Airflow/Spark job entry scripts), API/controllers, services/business logic, repositories/data-access, config (settings files, env templates, `application.yaml`), infra (`Dockerfile`, CI/CD pipeline files), and tests, plus a catch-all **General/Repo-level** bucket for cross-cutting concerns that don't belong to one layer (dependency management, documentation, overall architecture) **(fix: added catch-all, was previously undefined for repo-level findings)**. Not every repo has every layer — skip absent ones.
  11. Write `review/<repo-name>_<short-hash>_<YYYYMMDD_HHMMSS>.md` organized as:
      - **Summary**
      - **Stack Detected**
      - **Static Analysis Run** — a short factual log only: which tools ran, versions, exit status, raw issue counts (or "none run, no configured tooling found"). **(fix)** This section does **not** restate individual issues — every concrete issue (whether it came from a tool or manual reading) appears exactly once, inside its module section below, tagged with its source (tool name or "manual").
      - **Findings per module** (Entrypoint, API/Controllers, Services, Repositories/Data Access, Config, Infra, Tests, General — each present layer as its own section); each finding: dimension tag, severity (Critical/High/Medium/Low), file:line, description, recommendation, source (tool/manual).
      - **Scope & limitations** appendix (sampling applied, stack signals not matched, tools not available).

## Phase 4 — End-to-end validation **(refined: mandatory concrete run; fix: coverage beyond FastAPI + terminology)**
- Invoke the finished skill against one small, real public **FastAPI** repo end-to-end and confirm: clone succeeds, stack is correctly detected, static checks run if configured, the report lands in `review/` with the correct filename pattern and per-module structure, and findings are substantively grounded in the actual code (not generic filler).
- **(fix)** Also run it against one small real **Java/Spring Boot** repo, since Java is equally in-scope per the original requirements and was otherwise left completely unvalidated. If time-constrained, this second pass may be lighter (confirm detection + report structure) rather than as exhaustive as the FastAPI pass — but it must actually run, not be assumed to work by analogy.
- This is documentation/instructions, not application code — validation is these dry runs plus a self-read of `SKILL.md`/checklists for clarity and internal consistency, per `skill_flow.txt`'s "if it's documentation, re-read and evaluate" rule. The 95%-coverage gate only applies if a helper script gets added later.

## Phase 5 — Finalize docs
- `README.md`: usage (`/review-git-code <git-url> [branch]`), directory layout, attribution list (from `references/sources.md`).

## Risks / notes
- GitHub may not have 5 clean "Claude Code Skill" examples (recent feature) — documented fallback in Phase 1, mitigated further by the code-search channel.
- Large target repos risk blowing the review budget — mitigated by the fixed 300-file sampling guard in Phase 3, step 9.
- Downloaded reference material is internal guidance only, attributed in `sources.md`; raw copies live in a gitignored cache and are never committed **(fix)**.
- This directory (`review-code`) is not a git repo yet — scaffolding doesn't require it; will ask separately if the user wants `git init` before anything gets committed.
- Static-check tools are best-effort only — the skill never installs tooling on the host, it only runs what's already configured/available. Reports must say plainly when no static analysis ran, never fabricate tool output.
- Non-Python/Java repos get a reduced-confidence generic review rather than being refused **(fix)**.
- Phase 4 validation covers FastAPI thoroughly and Spring Boot lightly; PySpark/Flink/Docker/CI-CD paths remain unvalidated by an actual run in this iteration — acceptable for v1 but worth calling out explicitly rather than silently, and a good candidate for a future follow-up pass **(fix: documented instead of hidden)**.
