---
name: execute-plan
description: Execute a local Markdown plan file task by task in order, updating the plan file itself as progress is made so work can resume after interruptions. Use when the user says things like "/execute-plan @path-to-plan", asks to execute a plan from a file, continue a partially completed plan, or work through a Markdown task list and mark completed and blocked items in place.
---

# Execute Plan

Execute a Markdown plan file one task at a time, in order, and edit the plan file itself to record progress.

This skill is designed to pair well with `prd-to-plan`, but it must also work with looser Markdown task lists.

## Input

Expect a local plan file path, often in a form like:

```text
/execute-plan @plans/my-plan.md
```

If the path is missing or unreadable, ask for a valid local path before continuing.

## Core behavior

1. Read the plan file.
2. Identify tasks in execution order.
3. Start with the first task that is not already marked done.
4. Execute one task at a time.
5. Immediately update the plan file after each completed task.
6. Continue automatically until blocked.

Do not stop after each task just to ask permission. Only stop when blocked or when you are about to hit a practical limit.

## What counts as blocked

Stop when any of these are true:

- A task requires missing information or a human decision
- A task has an unmet dependency
- The task is too ambiguous to execute safely
- The work requires capabilities not currently available
- You are approaching a token, context, runtime, or usage limit

Before stopping, update the plan file so the stopping point is obvious.

## Supported plan shapes

Support common Markdown task formats, including:

- Checkbox items like `- [ ] Add API route`
- Numbered items like `1. Add API route`
- Bullet items like `- Add API route`
- Headings like `## Task 3: Add API route`

Prefer the simplest correct interpretation. If the file is too ambiguous to parse into an ordered task list, stop and say so instead of guessing.

## Ordering rules

- Default to file order
- If the plan explicitly declares dependencies such as `Blocked by: Task 1`, honor them
- If a task is already marked done, skip it
- Resume from the first task that is not done and is not blocked by unfinished earlier work

## Progress marking

Edit the plan file itself. Be conservative and preserve the user's text.

Use these rules:

- Checkbox task:
  - Change `- [ ]` to `- [x]` when complete
- Numbered or bullet task:
  - Add a new indented line directly below it: `Status: done`
- Heading-style task:
  - Add a line directly below the heading: `Status: done`

When blocked, add a short note directly under the current task:

```text
Status: blocked
Blocker: <short reason>
```

If the task later becomes unblocked and is completed, replace the blocked status with `Status: done` and remove or update the blocker note as appropriate.

## Execution loop

For each task:

1. Determine whether it is already complete from the plan markers
2. Confirm it is not blocked by explicit dependencies
3. Execute the task
4. Mark it done in the plan file immediately
5. Re-read the plan as needed and move to the next task

When you stop mid-plan, tell the user:

- which task was completed last
- which task is next
- why execution stopped

## File editing principles

- Keep edits minimal
- Do not rewrite unrelated sections
- Do not reformat the entire file
- Do not invent task structure that is not present
- Prefer adding short status lines over restructuring the document

## Pairing with `prd-to-plan`

When the plan comes from `prd-to-plan`, follow its dependency order and task sections directly.

Typical markers in those plans include:

- `### Task N: ...`
- `- **Blocked by**: ...`

For those files, add status lines inside the relevant task section without disturbing the rest of the template.

## Safety rules

- If a task appears to require destructive or high-risk actions, pause and ask before doing them
- If a task mixes multiple large subtasks, execute only if the intended scope is still clear
- If a task is underspecified, stop and record the blocker instead of improvising

## Output to the user

Keep the user-facing response brief. Report:

- the plan file path
- completed tasks in this run
- the current blocked or next task
- any blocker that needs user input
