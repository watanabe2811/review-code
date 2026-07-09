# review-code

A Claude Code Skill that clones a Git repository and produces a structured
code review report — security, performance, style/maintainability, test
coverage, error handling/logging, and architecture/scalability — organized
per architectural module (entrypoint, API/controllers, services,
repositories, config, infra, tests).

Targets Python and Java stacks: FastAPI APIs, Spring Boot services, PySpark
jobs, Flink jobs, ETL sync jobs, SQL/data-access code, Docker, and CI/CD
pipelines.

## Usage

Inside Claude Code, in this project:

```
/review-git-code <git-url> [branch-or-tag]
```

The skill will:
1. Clone (or update, if already cloned) the repo into `input/<repo-name>/`.
2. Detect the stack in use (language, framework, infra signals).
3. Run any static-analysis tooling already configured in the repo, if it's
   already installed on this machine (never installs anything new).
4. Read the code closely by architectural layer and evaluate it against the
   checklists in `.claude/skills/review-git-code/references/checklists/`.
5. Write one report to `review/<repo-name>_<commit-hash>_<timestamp>.md`.

## Layout

```
.claude/skills/review-git-code/
├── SKILL.md                     # the skill's instructions
└── references/
    ├── sources.md                # attribution for the 5 external skills the checklists were synthesized from
    └── checklists/                # security, performance, style, testing, error-handling, architecture
input/                            # gitignored — target repos get cloned here
review/                           # gitignored — generated reports land here
research/reference-skills/        # gitignored — raw research cache, not committed
```

## Attribution

The review checklists were synthesized from 5 publicly available Claude Code
Skills for code review, found via GitHub search and ranked by stars. Full
attribution (repo URLs, licenses, what was drawn from each) is in
[`.claude/skills/review-git-code/references/sources.md`](.claude/skills/review-git-code/references/sources.md).

## Build history

This skill was scoped and built through a staged plan — see `requirements.md`,
`plan_final.md`, and `TODO.md` for the full process, including two example
end-to-end review runs against [fastapi-realworld-example-app](https://github.com/nsidnev/fastapi-realworld-example-app)
and [spring-petclinic](https://github.com/spring-projects/spring-petclinic) in `review/`.
