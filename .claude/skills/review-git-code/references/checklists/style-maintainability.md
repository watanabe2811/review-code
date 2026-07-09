# Style & Maintainability Review Checklist

## General (Python & Java)

- [ ] Names are meaningful and intention-revealing — no single-letter/`temp`/`data`-style names for anything beyond a trivial loop index
- [ ] Functions/methods are short and single-purpose (rough guide: <50 lines, <4 params); classes stay focused (rough guide: <200-300 lines, not "Manager"/"Handler"/"Processor"-style grab-bags)
- [ ] No deep nesting — early returns/guard clauses used instead of pyramids of `if`
- [ ] No duplicated logic that should be a shared helper (same block appearing 3+ times is a strong signal)
- [ ] No magic numbers/strings — extracted to named constants or enums
- [ ] No dead code, commented-out blocks, or "might need this later" speculative abstractions (YAGNI)
- [ ] Comments explain *why*, not *what* — the code itself should read clearly enough that a what-comment is redundant
- [ ] Public APIs (routes, service methods, exported functions) are documented (docstring/Javadoc) where their contract isn't obvious from the signature

## Python

- [ ] Public functions/methods have type hints on parameters and return values
- [ ] No mutable default arguments (`def f(x=[])`, `def f(x={})`) — use `None` + assign inside, or `field(default_factory=...)` on dataclasses
- [ ] No mutable class-level attributes shared across instances unintentionally
- [ ] `is` used only for `None`/singleton comparisons, `==` for value comparisons
- [ ] Follows PEP 8 (naming, import ordering: stdlib → third-party → local; consistent line length)
- [ ] Modern idioms used where they clarify intent (f-strings, walrus operator, `match` statements) without forcing them where a plain `if`/loop is clearer

## Java

- [ ] Modern Java features used where they reduce boilerplate: `record` for immutable data carriers, switch expressions over legacy fall-through `switch`, text blocks for embedded SQL/JSON
- [ ] `Optional` used only as a return type — never as a field or method parameter, and never `isPresent()` + `get()` where `.map()`/`.orElse()` reads more clearly
- [ ] No deprecated date/time APIs (`Date`, `Calendar`, `SimpleDateFormat`) — `java.time` used instead
- [ ] Stream API used where it clarifies a transformation, not forced onto a simple loop, and not chained so deeply it becomes hard to debug
- [ ] Logging uses SLF4J, never `System.out.println`/`e.printStackTrace()`

## Spring Boot

- [ ] Configuration is type-safe via `@ConfigurationProperties`, not scattered `@Value("${...}")` injections or hardcoded values
- [ ] Controllers stay thin — business logic lives in the service layer, not inline in `@RestController` methods
- [ ] Lombok is used judiciously — `@Builder` on objects with required fields either validates in a compact constructor/`build()` override, or isn't used where it would let a caller skip a mandatory field

## PySpark

- [ ] Transformation chains are broken into named, testable steps rather than one giant chained expression that's hard to debug or unit test
- [ ] Job parameters (paths, dates, thresholds) are passed as arguments/config, not hardcoded inline in the job script
- [ ] Column names and business logic constants are centralized, not repeated as string literals across the job

## Flink

- [ ] Job topology (sources → transformations → sinks) is organized into named, readable stages rather than one monolithic `main`
- [ ] Custom `ProcessFunction`/operator state is encapsulated with clear naming, not anonymous inline lambdas doing non-trivial stateful logic

## ETL sync jobs

- [ ] Source/destination connection details and schedule parameters are externalized to config, not hardcoded in the sync script
- [ ] Transformation/mapping logic between source and destination schemas is centralized and readable, not scattered across multiple files without a clear entry point

## SQL / data-access

- [ ] Migrations are named/ordered clearly and are individually reviewable (not one giant migration bundling unrelated schema changes)
- [ ] Repeated query fragments are extracted into views/named queries rather than copy-pasted across the codebase

## Docker

- [ ] Dockerfile is organized logically (base image → system deps → app deps → app code) and commented only where a step's necessity isn't obvious
- [ ] No leftover debugging cruft (commented-out `RUN` lines, unused build args)

## CI/CD

- [ ] Pipeline definition is organized into clearly named jobs/stages, not one undifferentiated script blob
- [ ] Duplicated steps across jobs are factored into a reusable workflow/template where the CI platform supports it
