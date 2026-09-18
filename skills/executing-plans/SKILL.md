---
name: executing-plans
description: "Execute an approved implementation plan task by task, complete each task, resolve its ticket, then pre-commit review, user approval, and batch commit; use when a plan has been approved or when implement hands off its reviewed plan."
---

# Executing Plans

Build the work an approved plan describes, one task at a time, keep task todos synchronized, resolve the source ticket after approval, run a pre-commit code review on the complete batch, then let the user review and approve it before committing.

An approved plan is the precondition. If approval is missing, return to `writing-plans` and wait. Record the current `HEAD` as `base-sha` before editing; it anchors the batch diff and the final review.

## Process

### 1. Read and challenge the plan

Read the whole plan and the ticket/spec it argues from. Capture the source ticket path from the plan's `Ticket:` field and read the applicable tracker instructions. Check that every task can start, every interface lines up, every referenced file exists or is explicitly created, and every verification command is runnable. Raise concerns and wait for the user's answer or a plan amendment before editing.

### 2. Create task todos

Create one todo per plan task, in plan order, using the task names from the plan. Keep exactly one active task at a time, mark a task complete immediately after its declared checks pass, and do not leave a completed task in progress while moving to the next one. The plan owns the order until the user approves an amendment.

### 3. Execute all tasks sequentially

For each task:

1. Mark the task in progress.
2. Follow its steps in order and apply the code, test code, and commands shown by the plan.
3. Run each step's `Expected:` check and compare the actual result with the expected result.
4. Drive `/tdd` at the seams agreed by the plan: red test, minimal implementation, green test, then the next vertical slice.
5. Run the task's declared `Checks:` before calling it complete, including typecheck and focused tests.
6. Mark the task todo complete immediately after all of its checks pass.

Continue to the next task after the current task's checks pass. Do not pause for user release, commit, or batch review between tasks.

If a step surprises the implementation, record the information and continue only when the plan and the work still describe the same behavior. Use `/codebase-design` when an interface no longer matches the plan's boundary. If the plan and code disagree about what the work is, stop and ask for an amendment.

### 4. Account for every deviation

Compare the finished task with the plan step by step. Record every difference:

- A plan step that was skipped or could not be used as written.
- A file, signature, dependency, or behavior that differs from the plan.
- An extra change required to make the planned behavior compile or pass verification.
- A correction where the plan was wrong but the implementation is right.

State why each deviation was necessary. A real blocker stops the task and goes back to the user; it is never resolved by silently choosing an interpretation.

### 5. Run code review and present the complete batch for user review

After every task is complete, verify that every task todo is marked complete, run the plan-level `Checks:` and the full test suite once, and invoke `/code-review` with `base-sha` as the fixed point so it reviews the complete uncommitted batch. Present the batch and the review report together, then wait for the user's explicit approval. Include:

- A summary of every completed task.
- The complete deviation report.
- The complete diff: `git diff base-sha`.
- Every verification command and its result.
- The complete `/code-review` report.

Do not resolve the source ticket or commit until the user approves this reviewed batch. If the user requests changes, rework the affected tasks, return those task todos to in progress, rerun their focused checks and the plan-level checks, run `/code-review` again, and present the batch again. If the plan is amended, update the plan before continuing.

### 6. Commit the reviewed batch

After approval and before committing, close the source ticket using the tracker instructions. For the local Markdown tracker, mark every verified ticket acceptance item complete, change its `Status:` to `resolved`, and append a concise `## Resolution` summary; for a wayfinder ticket, use its `## Answer` and map context-pointer rules instead. Keep this ticket update in the same batch as the implementation. Run a final `git diff --check` and confirm no implementation files changed after the review. Then commit the complete batch as one commit using the plan or feature name in the commit message. Do not create task-by-task commits or a separate ticket-status commit.

## Deviation report template

```markdown
## Batch: <plan or feature name>

**Completed tasks:**

- Task <N>: <Task name>

**Ticket:**

- Source ticket: `<path>`
- Status: `resolved`
- Resolution: <short summary>

**Deviations:**

- Task <N> / `path/to/file.ts` - explain the difference and why it was necessary.
- Step N changed or skipped - explain the observed result.

**Verification:**

- `pnpm typecheck` -> clean
- `pnpm vitest run path/to/test.ts` -> passed

**Diff:** one-line summary of the complete batch.
```