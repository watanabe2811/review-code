# Reference sources

The checklists in `references/checklists/` were synthesized (paraphrased and
reorganized, not copied verbatim) from the following 5 publicly available
Claude Code Skills for code review, ranked by GitHub stars at the time of
research (2026-07-09). Raw downloads used only as synthesis input live in the
gitignored `research/reference-skills/` cache at the project root — they are
not committed or redistributed.

| # | Source | Repo | Stars | License | Skill path used |
|---|--------|------|-------|---------|------------------|
| 1 | alirezarezvani/claude-skills | https://github.com/alirezarezvani/claude-skills | 21,765 | MIT | `engineering-team/skills/code-reviewer/SKILL.md` (+ `languages/java.md`, `languages/python.md`, `rules/universal.md`) |
| 2 | Jeffallan/claude-skills | https://github.com/Jeffallan/claude-skills | 10,486 | MIT | `skills/code-reviewer/SKILL.md` (+ `references/review-checklist.md`, `references/report-template.md`, `references/common-issues.md`) |
| 3 | awesome-skills/code-review-skill | https://github.com/awesome-skills/code-review-skill | 1,339 | MIT | `SKILL.md` (+ `reference/fastapi.md`, `reference/java.md`, `reference/python.md`, `reference/architecture-review-guide.md`, `reference/code-review-best-practices.md`, `reference/common-bugs-checklist.md`, `reference/cross-cutting/{async-concurrency-patterns,n-plus-one-queries,sql-injection-prevention,xss-prevention}.md`) |
| 4 | mhattingpete/claude-skills-marketplace | https://github.com/mhattingpete/claude-skills-marketplace | 644 | Apache-2.0 | `productivity-skills-plugin/skills/code-auditor/SKILL.md` |
| 5 | SpillwaveSolutions/pr-reviewer-skill | https://github.com/SpillwaveSolutions/pr-reviewer-skill | 20 | none declared (used for research reference only, not redistributed) | `SKILL.md` (+ `references/review_criteria.md`) |

## 1-line takeaway per source
1. **alirezarezvani** — broad multi-language automation angle: complexity/risk scoring on PRs, SOLID-violation and code-smell detection, generates review checklists per language.
2. **Jeffallan** — single-pass broad-scope reviewer (correctness/performance/maintainability/tests) producing a structured, prioritized report; explicit severity/report template worth reusing for our own report format.
3. **awesome-skills/code-review-skill** — the most directly relevant source: has dedicated FastAPI, Java, and Python guides plus cross-cutting guides (N+1 queries, SQL injection, XSS, async/concurrency) that map almost one-to-one onto our review dimensions.
4. **mhattingpete/code-auditor** — codebase-level (not diff-level) audit across architecture/quality/security/performance/testing/maintainability — closest existing analog to "review a whole cloned repo" rather than a PR diff, which is exactly our skill's job.
5. **SpillwaveSolutions/pr-reviewer-skill** — GitHub-PR-centric workflow (gh CLI data collection, inline PR comments) — mostly out of scope for us (we review a cloned repo, not a live PR), but its `review_criteria.md` has a useful generic severity/criteria structure.

## Fallback note
All 5 slots were filled with genuine Claude Code Skills (`SKILL.md` files) found via GitHub code/repo search — the Google Engineering Practices / OWASP fallback documented in `plan_final.md` was not needed.
