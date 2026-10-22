---
name: change-walkthrough
description: Explain a selected code change on explicit user request with key implementation points, data flow, requirement and test links, risks, and a suggested reading order. Use through a read-only Code Researcher context; do not run automatically after development.
---

# Change Walkthrough

Help the user understand and inspect a change without retelling the entire diff. This is a read-only explanation workflow. It does not authorize fixes, new tests, publishing or a new development cycle.

## Scope and invocation

Run only when the user asks to explain or walk through changes, including equivalent natural-language requests. Use Code Researcher in a fresh bounded context; do not create a new permanent agent role. If the platform supports explicit-only skill policy, enable it; otherwise the dispatcher must enforce this trigger. Natural-language requests count as explicit invocation even when no `$skill-name` token is present.

Determine base and target: commits, commit range, staged/unstaged diff, named change or an explicitly selected snapshot. Resolve them to immutable IDs and record included untracked files. If the completed task already supplies exact base/candidate IDs, reuse them. If multiple unrelated changes are present and scope cannot be inferred, ask one focused question; do not silently choose the entire branch or latest commit.

## Procedure

1. Read the permitted diff and related OpenSpec/design/test references directly. Treat summaries as leads; confirm important claims in code. Identify change size and the main observable behavior.
2. Select 3–7 key points: entry point, main decision, state/data transformation, effect boundary, error path and the tests that matter. For a genuinely tiny change, fewer points are better than filler.
3. Trace the main path in plain language: input → validation/decision → state or effect → result. Explain non-obvious choices using contract/history evidence. If the motivation is only inferred, say so.
4. Link requirements to implementation anchors and tests. Distinguish “test exists” from “test ran and passed on this snapshot.” Read actual check receipts before claiming execution. Include gaps and meaningful manual scenarios.
5. Verify file paths, method names and line numbers against the target revision. Use links supported by the viewer; if revision permalinks are unavailable, include revision + path + symbol + verified line. Never fabricate precise lines for unseen files.
6. Produce the report below as a user-readable artifact. Return only a compact summary/ref to Orchestrator. Match the user's language and technical familiarity. Aim for roughly 600–1,200 words for an ordinary change; shorten for a small change. For a large change, explain the main path and offer named deep-dive areas without pretending full coverage.

## Report format

1. **What changed:** 2–4 sentences on behavior and scope, including base/target IDs.
2. **Affected components:** a compact map of the selected 3–7 points and their responsibilities.
3. **Key methods:** verified links and why the user should inspect them.
4. **Main logic and data flow:** a short causal explanation; a small diagram only if useful.
5. **Requirements and tests:** scenario → implementation → test → actual execution status. Clearly mark unknown/missing evidence.
6. **Risks and manual checks:** concrete scenarios and observable outcomes grounded in requirements; distinguish demonstrated defects from untested risks.
7. **Suggested reading order:** the shortest path to understanding, normally 3–7 anchors. Link existing anchors instead of copying code blocks.

Do not dump the full diff, list every file equally or invent risk findings to fill a section. “No additional risk established within this scope” is valid. Report material defects to the user and responsible workflow role through the caller's output contract; do not fix them yourself.

## Example invocation and outline

Request: “Explain the retry change between the completed task's base and candidate.”

Resolve the task's recorded snapshots. A useful explanation might follow the existing request entry point, retry eligibility decision, persistent state update, and regression tests. If the spec allows retry only from Failed, trace that exact condition; do not invent policies for cancellation, retry limits or backoff.

Illustrative anchor format: `candidate-revision · module/path/Job.scala · retry · verified line N`. Replace it with inspected facts in a real report. If only test source is available, state “test located; run result unavailable,” not “covered and passing.”

## Failure handling

On missing scope, inaccessible files or unavailable revisions, identify the specific missing input. Partial explanation is acceptable only with explicit coverage limits. Never claim full review, security assurance or correctness from a walkthrough. Use the caller's result contract with report reference, evidence, issues and next action.
