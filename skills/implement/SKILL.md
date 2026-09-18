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

- Let `executing-plans` own the complete execution lifecycle: plan review, task todos and their completion states, task checks, `/tdd` seams, deviation tracking, plan-level checks, the single pre-commit `/code-review`, source-ticket resolution, user approval, and the single batch commit.
- Do not repeat any of those lifecycle actions in `implement`, and do not invoke `/code-review` again after `executing-plans` returns.

Only this phase changes application code. If the plan and implementation disagree about the work, stop and ask for a plan amendment instead of choosing an interpretation.

## Completion

The batch is complete when `executing-plans` returns after completing its single pre-commit `/code-review`, user approval, source-ticket resolution, and one batch commit. `implement` performs no post-commit review.
