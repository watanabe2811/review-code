# review-code

## What this is
A Claude Code Skill project. The deliverable is
`.claude/skills/review-git-code/` — a skill that clones a Git repo, reviews
its code, and writes a Markdown report. There is no application runtime to
build or run; this repo's own "code" is the skill's Markdown instructions and
checklists.

## Layout
- `.claude/skills/review-git-code/SKILL.md` — the skill's instructions (entry point).
- `.claude/skills/review-git-code/references/checklists/*.md` — per-dimension review checklists (security, performance, style/maintainability, testing, error-handling/logging, architecture/scalability), each with stack-specific subsections (FastAPI, Spring Boot, PySpark, Flink, ETL sync, SQL/data-access, Docker, CI/CD).
- `.claude/skills/review-git-code/references/sources.md` — attribution for the 5 external reference skills the checklists were synthesized from.
- `input/` — gitignored; target repos get cloned here when the skill runs.
- `review/` — gitignored; generated review reports land here.
- `research/reference-skills/` — gitignored working cache of raw downloaded reference material, used only during checklist authoring, never committed.

## Conventions
- Nothing in `input/`, `review/`, or `research/` is committed — only the skill itself.
- Checklist content is synthesized/paraphrased from external sources, never copied verbatim (license hygiene).
- The skill never installs tooling on the host; static-analysis checks it runs are best-effort against whatever is already configured/installed in the target repo.

## Build history
See `requirements.md`, `plan_final.md`, `TODO.md` for how this skill was scoped and built.
