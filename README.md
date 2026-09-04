# Buddy Agent

Buddy Agent is an explicit-only agent skill that gives every bounded task a dedicated worker and an independent reviewer.

The worker implements and validates the task. The reviewer inspects the actual result and either approves it or sends concrete findings back to the same worker. The loop continues until the work is approved or a real blocker is reported.

For requests containing multiple distinct tasks, Buddy Agent creates a fresh worker-reviewer pair for each task.

## Requirements

- An agent environment that supports skills
- The ability to create multiple subagents and send follow-up messages to them

If those capabilities are unavailable, the skill reports the limitation instead of presenting self-review as independent review.

## Install in Codex

Clone the repository into your Codex skills directory:

```bash
git clone https://github.com/tommep/buddy-agent.git ~/.codex/skills/buddy-agent
```

Start a new Codex task so the skill catalog refreshes.

## Use

Invoke the skill explicitly:

```text
$buddy-agent build this feature and verify it works
```

Buddy Agent will coordinate one worker and one independent reviewer, keep the same pair through any correction loop, and report completion only after the reviewed state is approved.

## What approval means

Reviewer approval confirms that the worker and reviewer agree the stated acceptance criteria are satisfied within the available evidence. It does not grant permission to deploy, publish, spend money, change production, send messages, or perform destructive actions.
