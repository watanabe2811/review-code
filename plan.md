# Plan: Git Code Review Skill

Source: `requirements.md`. This plan covers building `.claude/skills/review-git-code/`
end-to-end: project scaffolding, reference-skill research, checklist design,
the skill itself, and validation.

## Phase 0 — Project scaffolding
- Create `CLAUDE.md` and `README.md` for this project (required by global Rule 3 — currently missing).
- Create `input/` and `review/` directories (with `.gitkeep` so they exist even empty) and a `.gitignore` that ignores their contents but keeps the directories tracked.
- Note: this directory is not yet a git repo. Scaffolding does not require git; will ask separately if the user wants `git init`.

## Phase 1 — Research & download 5 reference skills
- Search GitHub for existing Claude Code Skills (`SKILL.md`) focused on code review, using terms like "claude code skill code review", "SKILL.md code review", "anthropic skill review".
- Pick the 5 most popular by stars/relevance.
- Fetch each one's `SKILL.md` (and any directly-referenced supporting files) and save under `.claude/skills/review-git-code/references/<source-name>/`, with a short note on where it came from (repo URL, stars, license).
- If genuine Claude Code review-skills are scarce, fall back to the next-best widely-used code-review checklists (e.g. Google Engineering Practices, OWASP) as a documented substitution — this is a fallback, not the primary path.

## Phase 2 — Design review checklists
- Author one checklist file per review dimension under `.claude/skills/review-git-code/references/checklists/`:
  - `security.md`
  - `performance.md`
  - `style-maintainability.md`
  - `testing.md`
  - `error-handling-logging.md`
  - `architecture-scalability.md`
- Each checklist has a general Python/Java section plus project-type-specific subsections: FastAPI, ETL sync jobs, PySpark, Flink.
- Content synthesized from Phase 1 references plus known best practices for each framework (e.g. FastAPI: blocking calls in async handlers, dependency injection misuse; PySpark: shuffles/partitioning/caching/broadcast joins; Flink: checkpointing/state backend/watermarks/serialization; ETL: idempotency, retry/backoff, schema drift).

## Phase 3 — Author SKILL.md
- Write `.claude/skills/review-git-code/SKILL.md` with:
  - YAML frontmatter (`name`, `description` tuned for auto-invocation on "review this git repo" style requests).
  - Instructions: parse git URL + optional branch/tag arg → clone/update into `input/<repo-name>/` → detect project type(s) present → load relevant checklists from `references/checklists/` → review code → write `review/<repo-name>_<version>_<timestamp>.md`.
  - Report template embedded or referenced (summary, findings by dimension, severity, file:line, recommendation).
  - Guidance for large repos: prioritize `src`/core application directories, skip vendored/generated/build directories, cap scope if the repo is very large.

## Phase 4 — End-to-end validation
- Run the skill against one small real public repo per major project type if feasible (at minimum one FastAPI-style repo) to confirm: clone works, project-type detection works, report is generated with the correct filename pattern and structure.
- Since this deliverable is primarily documentation/instructions (not tested application code), validation is a manual dry-run + self-review of `SKILL.md` and checklists for clarity/completeness, per `skill_flow.txt` Step 5 ("nếu là tài liệu yêu cầu đọc lại và đánh giá"). The 95%-coverage gate applies only if this phase produces actual helper scripts.

## Phase 5 — Finalize docs
- Update `README.md` with usage instructions (`/review-git-code <git-url> [branch]`), directory layout, and attribution for the 5 reference sources.

## Risks / notes
- GitHub search may not surface 5 clean "Claude Code Skill" examples since the feature is recent — Phase 1 has a documented fallback.
- Large target repos risk exceeding review scope/budget — mitigated via prioritization guidance in Phase 3.
- Downloaded reference material is for internal guidance only; attribution kept in `references/`, not redistributed as this project's own claimed work.
- `input/` and `review/` will contain third-party code and generated reports — never committed (`.gitignore`).
