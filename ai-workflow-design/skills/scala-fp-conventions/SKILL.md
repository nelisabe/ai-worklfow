---
name: scala-fp-conventions
description: Write, test, and review Scala 2.13 or Scala 3 changes using the project's functional programming, effect, and Scalafmt conventions. Apply to Scala implementation, tests, and review; do not migrate stacks or reformat unrelated code.
---

# Scala FP Conventions

## Establish project facts

Read the supplied toolchain/research artifact or inspect the permitted configuration. Identify module-specific Scala versions, source/test roots, cross-build targets, test framework, effects/concurrency libraries, build wrapper/plugins, `.scalafmt.conf` and CI checks. Treat missing facts as unknown. Do not assume sbt, Cats, ZIO, ScalaTest, a formatter plugin or Scala 3 syntax.

Use the project's existing style and public contracts unless they conflict with explicit accepted requirements. Do not migrate from Scala 2.13 to 3, replace the test framework, add a dependency, change compiler flags or introduce an effect library for stylistic consistency.

For architecture-only assignments, read [architecture-principles.md](references/architecture-principles.md); the formatter and implementation procedure need not be loaded into the Architect's context.

## Values, functions and errors

- Prefer `val`, immutable collections and pure transformations. A `val` pointing to a mutable object is not immutable state. Avoid shared mutation. Local mutation may be justified by measured performance or an established builder pattern if contained, unobservable and tested; state the reason without introducing a new framework.
- Keep decisions and transformations separate from effects. Prefer straightforward composition and named intermediate values when a long combinator chain obscures behavior.
- Use `Option` for meaningful absence, `Either` for expected typed failures, and `Try` at exception-producing synchronous boundaries when consistent with the existing API. Do not erase errors with `.toOption` or replace invalid input with a plausible default unless specified. `Try` is not a general lazy effect or resource manager.
- Avoid unchecked `.get`, `.head`, `.tail`, indexing, casts and non-exhaustive pattern matches. Use total alternatives such as `headOption`, `lift`, pattern matching or `fold`, preserving domain error semantics. A justified invariant should be explicit and tested, not merely asserted in a comment.
- Use `map`, `flatMap`, `fold`, `collect` and `for` comprehensions when they clarify intent. `collect` deliberately drops unmatched elements; do not use it if dropping invalid input violates requirements. Do not force FP terminology into names or replace a clear match with complicated abstractions.
- Prefer small domain types/ADTs when they make invalid states harder to represent. Scala 2.13 supports sealed traits and case classes/objects; Scala 3 may use enums when appropriate to the project. Do not insert Scala 3-only syntax in shared 2.13 sources.
- Preserve compatibility, null handling and exception behavior at Java/external boundaries according to the contract; do not silently change callers' expectations in the name of purity.

## Effects, resources and concurrency

Use the effect system already present. Keep resource acquisition/release scoped with that stack's existing resource constructs; cancellation and error paths must also release resources. For synchronous standard-library code, use the established safe resource pattern (for example `Using` where compatible), not unmanaged open/close calls.

Do not assume `Future` is lazy. Respect the application's execution contexts and resource lifetimes. Avoid `Await`, `Thread.sleep`, blocking I/O on compute/event-loop pools and uncontrolled fire-and-forget work. If blocking is necessary, use the established dedicated mechanism and test failure/cancellation behavior as appropriate.

Use bounded concurrency when required by resource limits. Preserve ordering, idempotency and atomicity guarantees from the spec. Do not introduce Cats Effect, ZIO, locks, actors or custom schedulers solely to satisfy an FP preference. Read the installed library/version's local conventions before using stack-specific APIs; this skill does not prescribe a universal effect API.

## Tests

Use the existing test framework, fixtures and naming style. Derive expected values from requirements; use base code only to understand integration points. Cover accepted errors, boundaries and invariants, not just happy paths. Use deterministic clocks, seeds and test-controlled scheduling where supported; avoid flaky time assumptions and oversized mocks.

Property-based tests are useful for algebraic/domain invariants if the framework is already available or explicitly approved. Do not add a dependency for a single routine assertion. Tests should observe behavior, not mechanically mirror the implementation. Test Engineer owns test edits; Developer reports suspected defects; Reviewer reports findings without repairs.

## Scalafmt Compliance

1. Locate the applicable `.scalafmt.conf`, including module configs and referenced includes. Record its hash, pinned formatter version and effective dialect for changed files. Preserve it. A missing or incompatible config is a blocker for mandatory format compliance until resolved by the authorized owner; do not create one implicitly through an IDE.
2. Discover the actual formatter integration from build files, wrapper, CI and approved tool registry. Do not assume a plugin is installed. Maven and Gradle integrations differ; do not invent goals or tasks. Project-wide check scope may be acceptable, but write scope must remain restricted.
3. Obtain the changed Scala file list for the exact snapshot and filter it by your role's writable grant. Use a verified file-targeting formatter command or the existing narrow module/source-set task. Format whole changed files, not arbitrary lines. Never expand to whole-repository formatting just because a broad command is easier.
4. Request the initial formatting check through the isolated runner. If violations are in your permitted files, request formatting, inspect the resulting diff for unintended semantic rewrites, publish only permitted changes and repeat the check. A changed snapshot requires fresh affected compile/test evidence.
5. If the integration only formats broadly, run it on a disposable snapshot, extract only authorized file changes through the publisher, then check the resulting candidate. Never run a broad writer against the shared live workspace. If scoped safe extraction cannot be guaranteed, report blocked.
6. Report command/operation ID, selected files/module, config hash, tool version, snapshot, exit code and receipt reference. `not_run`, download failure, unsupported syntax, timeout and nonzero exit are not passed. Reviewer uses check-only operations and requests fixes from the owner.
7. Distinguish changed-file compliance from full-project compliance. If a required project-wide check fails on pre-existing files, report the baseline failure; do not claim full compliance or silently exempt it. An existing approved baseline exemption must be explicit in the acceptance policy.

Command examples are conditional, not permission to execute:

| Confirmed integration | Candidate operation to verify in that project |
|---|---|
| sbt-scalafmt | Module main/test `scalafmtCheck`; `scalafmtOnly` with an explicit verified file list |
| Installed Scalafmt CLI | Configured check mode and explicit files according to that version's help |
| Maven plugin | The configured check/format goals and module selection from the POM and CI |
| Gradle plugin | The configured check/apply tasks and source targeting from the build and CI |
| Other build tool | The project's actual configured formatter integration |

The runtime must map these to exact approved executable/argv, cwd, environment and side effects. An unrestricted shell command is not required by this skill.

## Small examples

Suppose the accepted contract explicitly distinguishes absence from an empty value. This total transformation is compatible with Scala 2.13 and Scala 3 syntax styles that allow braces:

```scala
def firstNonEmpty(values: List[String]): Option[String] =
  values.find(_.nonEmpty)
```

Use only if null elements are excluded by the contract. Do not silently add trimming or discard whitespace-only strings unless required.

For a contract that requires an existing value or a typed missing error:

```scala
sealed trait LookupError
case object Missing extends LookupError

def lookup(key: String, values: Map[String, Int]): Either[LookupError, Int] =
  values.get(key).toRight(Missing)
```

An existing error ADT should be reused instead of introducing this new one. These examples are illustrative and do not establish business requirements or prove compilation/formatting in an unseen project.

## Completion evidence

Provide the relevant FP/design deviations with justification, changed file references, compile/test receipts and formatting outcome using the caller's output contract. Missing tools prevent a compliance claim; they do not authorize installing or modifying the build automatically.
