---
name: buddy-agent
description: "Explicit-only two-agent delivery workflow. Use when the user invokes $buddy-agent and wants each bounded task completed by one fresh worker and independently checked by one fresh reviewer through a correction loop before completion is reported."
---

# Buddy Agent

Run one worker-reviewer pair for every bounded task in the request. The main agent remains the coordinator and user-facing owner; it does not count as one of the two agents.

## Requirements

This skill requires an agent environment that can create multiple subagents and send follow-up messages to them. If those capabilities are unavailable, report the limitation instead of imitating independent review.

## Boundaries

- Treat explicit `$buddy-agent` invocation as authorization to delegate the in-scope task to exactly two fresh subagents: one worker and one reviewer.
- Do not invoke this skill implicitly. Do not reuse a worker or reviewer from another task.
- If the request contains multiple distinct tasks, define their boundaries and assign a separate fresh pair to each. Queue pairs when concurrency is limited; never drop the review to save time.
- Neither subagent may spawn more agents unless the user separately asks for additional delegation.
- Delegation does not expand authority. The pair may only perform actions already authorized by the request and governing instructions. Do not infer permission to commit, push, deploy, publish, release, change production, send messages, spend money, alter permissions, or perform destructive actions.

## Set up the pair

1. Resolve the task scope, acceptance criteria, relevant source of truth, allowed changes, required validation, and known limits before delegation.
2. Spawn the worker with the complete task contract. The worker owns implementation, proportionate validation, and an evidence-based handoff listing changed artifacts, checks run, results, and remaining limits.
3. Spawn the reviewer separately with the same task contract. The reviewer must remain independent: it may inspect artifacts and run safe, non-destructive checks, but it must not edit the deliverable, implement fixes, or treat the worker's claims as proof.
4. Have the reviewer first form an independent acceptance checklist. After the worker's handoff is available, send the handoff and exact artifact scope to the reviewer for inspection of the actual post-work state.

Use the available collaboration tools to preserve the same pair across the correction loop. Route handoffs, findings, and rebuttal evidence through the main agent so the decision trail remains legible.

## Review loop

1. The worker reports `READY_FOR_REVIEW` with its evidence and known limitations.
2. The reviewer returns exactly one verdict:
   - `APPROVED` — every acceptance criterion is satisfied within explicitly stated evidence boundaries.
   - `CHANGES_REQUESTED` — one or more concrete findings remain, each tied to an acceptance criterion and supported by file, diff, command, UI, or other direct evidence.
3. For `CHANGES_REQUESTED`, send the findings to the same worker. The worker either fixes and revalidates each finding or responds with specific contradictory evidence.
4. Send the revised state and evidence to the same reviewer. The reviewer must inspect the new state rather than approving the worker's description alone.
5. Repeat until the worker reports ready and the reviewer returns `APPROVED`.

Agreement must be evidence-based. The worker should not accept an incorrect finding merely to end the loop, and the reviewer should not waive an unmet criterion merely because the worker disagrees. If the dispute exposes material ambiguity, missing authority, destructive validation, an unavailable environment, or another real external blocker, stop and report `BLOCKED` with the exact unresolved issue instead of fabricating consensus.

## Completion gate

Do not report the task as complete in the main window until both conditions are true:

- the worker states that the reviewed state is ready and has no known unaddressed findings; and
- the reviewer explicitly returns `APPROVED` after inspecting that state.

Then return one concise consolidated status containing:

- outcome and exact scope;
- changed artifacts or actions taken;
- verification performed and strongest evidence;
- reviewer verdict;
- untested boundaries, remaining risks, or follow-up gates.

Reviewer approval establishes worker-reviewer agreement only. It is not automatically the user's acceptance, proof of production behavior, or permission for the next delivery step.

If either agent becomes unavailable or independent review cannot be completed, report the task as `BLOCKED`; never substitute main-agent self-review and call it buddy-approved.
