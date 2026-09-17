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
- Create one todo per plan task and execute tasks in plan order.
- Follow each step's code and test examples, run its `Expected:` check, and run the task's declared typecheck and focused tests.
- Use `/tdd` at the seams agreed by the plan.
- At the end of every task, present the task-scoped diff, deviation report, and verification output, then wait for the user's release before committing or starting the next task.
- Run the full test suite once after the final task.

Only this phase changes application code. If the plan and implementation disagree about the work, stop and ask for a plan amendment instead of choosing an interpretation.

## Completion

After every task has been released and committed, invoke `/code-review` to review the complete diff against the standards and the approved plan/spec.

Commit the released work to the current branch, following the task boundaries required by `executing-plans`.
