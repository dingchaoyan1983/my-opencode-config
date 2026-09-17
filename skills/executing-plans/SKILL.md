---
name: executing-plans
description: "Execute an approved implementation plan one task at a time with review checkpoints; use when a plan has been approved or when implement hands off its reviewed plan."
---

# Executing Plans

Build the work an approved plan describes, one task at a time, with the user reviewing each task before the next one starts.

An approved plan is the precondition. If approval is missing, return to `writing-plans` and wait. The checkpoint is part of the workflow: implementation, reviewable diff, deviation report, verification, and user release are one task boundary.

## Process

### 1. Read and challenge the plan

Read the whole plan and the ticket/spec it argues from. Check that every task can start, every interface lines up, every referenced file exists or is explicitly created, and every verification command is runnable. Raise concerns and wait for the user's answer or a plan amendment before editing.

### 2. Create task todos

Create one todo per plan task, in plan order, using the task names from the plan. The plan owns the order until the user approves an amendment.

### 3. Execute one task step by step

For each task:

1. Mark the task in progress.
2. Follow its steps in order and apply the code, test code, and commands shown by the plan.
3. Run each step's `Expected:` check and compare the actual result with the expected result.
4. Drive `/tdd` at the seams agreed by the plan: red test, minimal implementation, green test, then the next vertical slice.
5. Run the task's declared `Checks:` before calling it complete, including typecheck and focused tests.

If a step surprises the implementation, record the information and continue only when the plan and the work still describe the same behavior. Use `/codebase-design` when an interface no longer matches the plan's boundary. If the plan and code disagree about what the work is, stop and ask for an amendment.

### 4. Account for every deviation

Compare the finished task with the plan step by step. Record every difference:

- A plan step that was skipped or could not be used as written.
- A file, signature, dependency, or behavior that differs from the plan.
- An extra change required to make the planned behavior compile or pass verification.
- A correction where the plan was wrong but the implementation is right.

State why each deviation was necessary. A real blocker stops the task and goes back to the user; it is never resolved by silently choosing an interpretation.

### 5. Stop at the checkpoint

Present the completed task and wait for the user's release. Include:

- The deviation report.
- The task-scoped diff: `git diff HEAD` when the previous task commit is `HEAD`, or `git diff <previous-task-sha>` for later tasks.
- Every verification command and its result.

Do not commit or start the next task until the user releases this checkpoint. Rework a returned task and present the checkpoint again; if the plan is amended, update the plan before continuing.

### 6. Commit released tasks

After release, commit exactly that task using its plan task name in the commit message, then mark its todo complete. The commit is the boundary for the next task's diff.

Run the full test suite once the final task's checks are complete.

### 7. Hand off for final review

After every task is released, committed, and verified, invoke `/code-review` for the complete diff against the project standards and the approved plan/spec.

## Deviation report template

```markdown
## Task <N>: <Task name>

**Deviations:**

- `path/to/file.ts` - explain the difference and why it was necessary.
- Step N changed or skipped - explain the observed result.

**Verification:**

- `pnpm typecheck` -> clean
- `pnpm vitest run path/to/test.ts` -> passed

**Diff:** one-line summary of the files this task touched.
```