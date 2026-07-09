## Plan: Git Code Review Skill (execution TODO, from `plan_final.md`)

### Steps

**Phase 0 — Scaffolding**
- [ ] Create `CLAUDE.md` (tech stack: Claude Code Skill/Markdown; no app runtime; conventions for this project)
- [ ] Create `README.md` (placeholder — filled in for real in Phase 5)
- [ ] Create `.gitignore` ignoring `input/*`, `review/*`, `research/*` (keep `.gitkeep`s)
- [ ] Create `input/.gitkeep`, `review/.gitkeep`, `research/reference-skills/.gitkeep`

**Phase 1 — Research & download 5 reference skills**
- [ ] `WebSearch` for existing Claude Code code-review skills (repo-name/topic search)
- [ ] GitHub code search (`filename:SKILL.md review`) as a second discovery channel
- [ ] Rank candidates by popularity (`gh search repos --sort=stars` if available, else WebSearch signal)
- [ ] Select top 5 (or apply the Google/OWASP fallback for any unfilled slots)
- [ ] `WebFetch` each source's raw `SKILL.md`, save to `research/reference-skills/<source-name>/reference.md` (gitignored cache)
- [ ] For each source, follow the relative links inside its `SKILL.md` (references/checklists/scripts/examples it points to) and `WebFetch` those too, saving under `research/reference-skills/<source-name>/knowledge/` mirroring the source repo's paths — not just the entry-point file
- [ ] Write `.claude/skills/review-git-code/references/sources.md` — URL, stars, license, 1-line takeaway per source (this file is what gets committed)

**Phase 2 — Checklists**
- [ ] Write `references/checklists/security.md`
- [ ] Write `references/checklists/performance.md`
- [ ] Write `references/checklists/style-maintainability.md`
- [ ] Write `references/checklists/testing.md`
- [ ] Write `references/checklists/error-handling-logging.md`
- [ ] Write `references/checklists/architecture-scalability.md`
  - Each: general Python/Java section + FastAPI, Spring Boot, PySpark, Flink, ETL sync, SQL/data-access, Docker, CI/CD subsections, synthesized from the Phase 1 cache + known practices (not verbatim copied)

**Phase 3 — SKILL.md**
- [ ] Write `.claude/skills/review-git-code/SKILL.md` with frontmatter (`name: review-git-code`, trigger-worthy `description`)
- [ ] Instructions covering, in order: parse input → clone/update `input/<repo-name>/` → resolve commit hash → read project structure → detect stack (9 signals + no-match fallback) → detect + run static-check tooling (best-effort, `command -v` guarded) → load relevant checklist sections → apply 300-file large-repo sampling guard → deep layered review (Entrypoint/API/Services/Repositories/Config/Infra/Tests/General) → write `review/<repo-name>_<short-hash>_<timestamp>.md` per the de-duplicated report structure (Summary → Stack Detected → Static Analysis Run log → per-module findings → Scope & limitations)

**Phase 4 — Validation**
- [ ] Pick one small public FastAPI repo; invoke the skill end-to-end; verify clone, stack detection, static checks (if applicable), report filename pattern, per-module structure, and grounded (non-generic) findings
- [ ] Pick one small public Java/Spring Boot repo; invoke the skill end-to-end (lighter pass); verify detection + report structure at minimum
- [ ] Self-read `SKILL.md` and all 6 checklists for clarity/internal consistency; revise anything unclear
- [ ] If either validation run surfaces a defect (wrong detection, malformed report, crash), fix `SKILL.md`/checklists and re-run until both pass

**Phase 5 — Docs**
- [ ] Fill in real `README.md`: usage (`/review-git-code <git-url> [branch]`), directory layout, attribution list pulled from `sources.md`

### Risks / notes
- Static-check tools are best-effort only — never install tooling, only use what's already on this machine.
- No verbatim third-party content gets committed — raw downloads stay in the gitignored `research/reference-skills/` cache; only `sources.md` (attribution) and the synthesized checklists are committed.
- Non-Python/Java repos get the generic no-match fallback path, not a refusal.
- PySpark/Flink/Docker/CI-CD detection paths are not covered by an actual end-to-end validation run in this pass (only FastAPI + Spring Boot are) — acceptable for v1, flagged as a known gap for a future follow-up.
- This directory isn't a git repo yet — nothing here gets pushed; `git init` is a separate ask if wanted later.
