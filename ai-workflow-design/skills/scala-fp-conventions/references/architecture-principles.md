# Scala architecture principles

Use this short reference for architecture-only work; implementation/test/review agents use the main skill.

- Extend existing owners and compose functions before adding layers. Each new type or module needs a concrete responsibility and a reason existing code cannot safely own it.
- Prefer immutable domain state and pure decisions with explicit effect boundaries. Preserve the project's effect and concurrency model.
- Model expected errors explicitly with the established types. ADTs are useful when they clarify real alternatives or invariants; do not create a hierarchy for every value.
- Prefer composition. A new interface with one implementation requires a concrete boundary, replacement need or testing reason that simpler seams do not satisfy.
- Do not add service/manager/helper/validator/factory classes mechanically. Compare the minimal alternative with costs in readability, coupling and change isolation.
- Simplicity includes safety, resource ownership, atomicity and testability. A separate component can be necessary even if it increases file count.
- Keep Scala 2.13/3 and cross-module compatibility explicit. Do not propose stack migration, new effect libraries or speculative extension points unless the accepted task needs them.

Contract checklist: reused owners; justified additions; public signatures; error/resource/concurrency invariants; forbidden deviations; alternatives; implementation sequence. Formatting procedures belong to the writing agents, not this reference.
