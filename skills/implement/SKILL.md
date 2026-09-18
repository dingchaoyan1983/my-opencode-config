---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

# Implement

Implement the work described by the user in the spec or tickets through a gated plan-first workflow. Application code starts only after the user has reviewed and explicitly approved the plan.

## Phase 1: Write the plan

Invoke the `writing-plans` skill with the user's request unchanged.

The plan must:

- Read the ticket, spec, project instructions, domain glossary, and relevant ADRs before planning.
- Map the exact files and interfaces that will change.
- Split the work into reviewable, independently verifiable tasks.
- Include the implementation code, test code, commands, and expected results for every step. A prose-only plan is incomplete.
- Complete its self-review, save the plan, present its path and summary, and stop for explicit user approval.

No application code is changed during this phase.

## Phase 2: Execute the approved plan

After explicit approval, invoke the `executing-plans` skill with the approved plan.

The execution must:

- Read and critically review the approved plan before editing; surface plan concerns before starting.
- Create one todo per plan task, execute tasks in plan order, and mark each todo complete immediately after that task's declared checks pass.
- Follow each step's code and test examples, run its `Expected:` check, and run the task's declared typecheck and focused tests.
- Use `/tdd` at the seams agreed by the plan.
- After all tasks finish, run the plan-level checks and full test suite, then invoke `/code-review` with the batch base SHA before presenting the complete diff, deviation report, verification output, and review report.
- Wait for the user's review and approval of that batch. If review rework changes the batch, return the affected todos to in progress, rerun the affected checks, plan-level checks, full test suite, and `/code-review` before asking for approval again.
- After approval and before committing, mark the source ticket's verified acceptance items complete, change its status to `resolved`, and append the tracker-required resolution. Include that ticket update in the single batch commit; do not create a separate ticket-status commit.

Only this phase changes application code. If the plan and implementation disagree about the work, stop and ask for a plan amendment instead of choosing an interpretation.

## Completion

The batch is complete after every task todo is complete, the source ticket is resolved, the user approves the pre-commit `/code-review` report, and the approved implementation plus ticket update are committed to the current branch as one commit, following the batch boundary required by `executing-plans`.
