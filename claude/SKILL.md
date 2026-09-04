---
name: buddy-agent
description: "Explicit-only two-agent delivery workflow. Use when the user explicitly invokes Buddy Agent and wants each bounded task completed by one fresh worker and independently checked by one fresh reviewer through a correction loop before completion is reported."
disable-model-invocation: true
---

# Buddy Agent

Run one worker-reviewer pair for every bounded task in the request. The main agent remains the coordinator and user-facing owner; it does not count as one of the two agents.

## Requirements

This skill requires an agent environment that can create multiple subagents and send follow-up messages to them. If those capabilities are unavailable, report the limitation instead of imitating independent review.

## Boundaries

- Treat explicit user invocation of this skill as authorization to delegate the in-scope task to exactly two fresh subagents: one worker and one reviewer.
- Do not invoke this skill implicitly. Do not reuse a worker or reviewer from another task.
- If the request contains multiple distinct tasks, define their boundaries and assign a separate fresh pair to each. Queue pairs when concurrency is limited; never drop the review to save time.
- Neither subagent may spawn more agents unless the user separately asks for additional delegation.
- Delegation does not expand authority. The pair may only perform actions already authorized by the request and governing instructions. Do not infer permission to commit, push, deploy, publish, release, change production, send messages, spend money, alter permissions, or perform destructive actions.

## Set up the pair

1. Resolve the task scope, required acceptance criteria, relevant source of truth, allowed changes, required validation, and known limits before delegation. Separate optional improvements from required outcomes; a reviewer may refine how to check a criterion but may not silently add new requirements.
2. Spawn the worker with the complete task contract. The worker owns implementation, proportionate validation, and an evidence-based handoff listing changed artifacts, checks run, results, and remaining limits.
3. Spawn the reviewer separately with the same task contract. The reviewer must remain independent: it may inspect artifacts and run safe, non-destructive checks, but it must not edit the deliverable, implement fixes, or treat the worker's claims as proof.
4. Have the reviewer first form an independent acceptance checklist. After the worker's handoff is available, send the handoff and exact artifact scope to the reviewer for inspection of the actual post-work state.

Use the available collaboration tools to preserve the same pair across the correction loop. Route handoffs, findings, and rebuttal evidence through the main agent so the decision trail remains legible.

## Review loop

1. The worker reports `READY_FOR_REVIEW` with its evidence, known limitations, and an identifiable artifact state, such as a version, document revision, or bounded file snapshot. A commit identifier alone is insufficient when relevant uncommitted changes exist.
2. The reviewer returns exactly one verdict:
   - `APPROVED` — every required acceptance criterion is satisfied within explicitly stated evidence boundaries, with the approved artifact state identified.
   - `CHANGES_REQUESTED` — one or more required corrections remain, each tied to an agreed criterion or a concrete correctness or authority violation, and supported by direct evidence. List optional suggestions separately; they do not block readiness or approval and are not implemented without authorization.
3. For `CHANGES_REQUESTED`, send the findings to the same worker. The worker either fixes and revalidates each finding or responds with specific contradictory evidence.
4. Send the revised state and evidence to the same reviewer. The reviewer must inspect the new state rather than approving the worker's description alone. Repeat checks affected by the changes; broaden review when new evidence indicates a regression or a previously unchecked dependency.
5. Repeat until the worker reports ready and the reviewer returns `APPROVED`.

The reviewer must directly inspect the deliverable and independently verify its important claims with proportionate checks. Distinguish checks personally performed, worker-reported results inspected, and behavior not verified. Do not present a worker's successful check as independently reproduced. Unavailable evidence required for acceptance prevents approval; optional checks may remain explicitly untested.

Track required findings and how each was resolved. Reopen a resolved finding only when relevant artifacts change or new evidence warrants it. If an objection repeats without new evidence, the coordinator selects the smallest safe, authorized check that can resolve the disagreement. Do not repeat the same arguments or invent a round limit that waives an unmet criterion. Disagreement alone is not an external blocker; report `BLOCKED` only when a specific missing input, authority, capability, or environment prevents resolution.

Agreement must be evidence-based. The worker should not accept an incorrect finding merely to end the loop, and the reviewer should not waive an unmet criterion merely because the worker disagrees. If the dispute exposes material ambiguity, missing authority, destructive validation, an unavailable environment, or another real external blocker, stop and report `BLOCKED` with the exact unresolved issue instead of fabricating consensus.

## Completion gate

Do not report the task as complete in the main window until all conditions are true:

- the worker states that the reviewed state is ready and has no known unaddressed required corrections;
- the reviewer explicitly returns `APPROVED` after inspecting that state; and
- the coordinator confirms that the deliverable still matches the approved state. Any subsequent change to a reviewed artifact requires the same reviewer to inspect the affected changes and renew approval before completion. An unchanged copy may retain approval when its equivalence is verified.

Then return one concise consolidated status containing:

- outcome and exact scope;
- changed artifacts or actions taken;
- verification performed and strongest evidence;
- reviewer verdict;
- untested boundaries, remaining risks, or follow-up gates.

Reviewer approval establishes worker-reviewer agreement only. It is not automatically the user's acceptance, proof of production behavior, or permission for the next delivery step.

If either agent becomes unavailable or independent review cannot be completed, report the task as `BLOCKED`; never substitute main-agent self-review and call it buddy-approved.
