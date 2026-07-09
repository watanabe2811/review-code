# Requirements: Git Code Review Skill

## 1. Overview
Build a **Claude Code Skill** (`SKILL.md`) that reviews a codebase pulled from a
Git repository. The user invokes the skill with a Git URL; Claude clones the
repo, performs a structured code review, and writes a Markdown report to a
`review/` directory. The skill's review guidance is informed by 5 popular
existing GitHub repos that contain Claude Code Skills for code review.

## 2. Goals
- Repeatable, one-command code review workflow for any Git repo the user points at.
- Review quality grounded in external best-practice references (the 5 downloaded skills), not just the model's own priors.
- Coverage tailored to the user's actual stack: Python/Java APIs, ETL sync jobs, PySpark, Flink.

## 3. Functional Requirements

### 3.1 Skill invocation & input
- Delivered as a Claude Code Skill at `.claude/skills/review-git-code/SKILL.md` (project-scoped, lives in this `review-code` project).
- Invoked as `/review-git-code <git-url> [branch-or-tag]`.
- Skill clones the repo into `input/<repo-name>/` using the host's already-configured `git` (existing credentials/SSH — no auth handling needed in the skill).
  - If `input/<repo-name>` already exists: `git fetch` + `git checkout` the requested ref and `git pull`, rather than re-cloning from scratch.
- Captures the resolved commit short-hash (and branch/tag if given) for use in the report filename.

### 3.2 Reference knowledge base (build-time, not per-run)
- As part of building this skill (one-time setup, done during execution of the final TODO, not repeated on every review run), search GitHub and download the **5 most popular existing Claude Code Skills for code review**.
- Store their raw `SKILL.md` (and any supporting reference files) under `.claude/skills/review-git-code/references/<source-name>/`.
- Synthesize their checklists/structure into this skill's own instructions — cite which ideas came from which source in a short "inspired by" note, but the shipped skill should be self-contained (does not re-fetch these at review time).

### 3.3 Review scope
**Languages:** Python, Java.

**Project types the skill must recognize and tailor guidance for:**
- REST APIs — Python / **FastAPI**
- ETL synchronization flows (batch data-sync jobs)
- **PySpark** jobs
- **Flink** jobs (Java and/or PyFlink)

**Review dimensions (every review must address all of these, skipping only what's genuinely inapplicable):**
1. Security (e.g. injection, secrets handling, authN/authZ, unsafe deserialization, dependency CVEs)
2. Performance (e.g. blocking calls in async FastAPI handlers, N+1 queries, Spark shuffles/partitioning/caching, Flink state/backpressure/serialization)
3. Code style / maintainability (PEP8 / Java conventions, naming, structure)
4. Test coverage / testability (presence of tests, coverage gaps, hard-to-test design)
5. Error handling / logging (exception handling, structured logging vs print/System.out)
6. Architecture / scalability (especially important for ETL/PySpark/Flink pipelines processing large data volumes)

Each finding in the report should include: category, severity (Critical/High/Medium/Low), file:line reference, description, and recommendation.

### 3.4 Output
- One Markdown report per reviewed repo, written to `review/`.
- Filename pattern: `review/<repo-name>_<version>_<timestamp>.md`
  - `<version>` = resolved short commit hash (or tag/branch name if provided)
  - `<timestamp>` = review execution time, format `YYYYMMDD_HHMMSS`
- Report structure: summary/overview → findings grouped by the 6 dimensions above → optional appendix (files scanned, scope/limitations).

## 4. Non-Functional Requirements
- Skill must be reusable across repos without manual reconfiguration between runs.
- Review is performed by Claude itself during the skill run (Bash for clone, Read/Grep for analysis, Write for the report) — no separate external LLM API call.
- Must handle repos where some dimensions don't apply (e.g. no Flink code present) by omitting that section rather than fabricating findings.

## 5. Assumptions & Constraints
- Git is already installed and authenticated on this machine (per `skill_request.txt`) — the skill does not manage credentials.
- `input/` and `review/` directories live at this project's root and are gitignored (cloned third-party code and generated reports should not be committed).
- One repo is reviewed per invocation; multiple repos are handled by invoking the skill multiple times.
- The "coverage ≥ 95%" rule from `skill_flow.txt` Step 5 applies to any helper code (scripts/tests) we write to support the skill — the `SKILL.md` instructions themselves are documentation, not tested code, so that gate applies only if the plan introduces actual scripts.

## 6. Out of Scope
- No CI/CD integration (no auto-trigger on push/PR).
- No UI/dashboard.
- No auto-remediation of findings — report only.
- No support for private repos whose credentials aren't already configured on the host.

## 7. Assumptions requiring explicit confirmation
- Default severity scale: Critical / High / Medium / Low.
- On re-run against an already-cloned repo, the skill updates via `git fetch`/`pull` rather than deleting and re-cloning.
