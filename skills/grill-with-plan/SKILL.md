---
name: grill-with-plan
description: "Grill requirements, write a plan, and execute it after approval."
disable-model-invocation: true
---

# Grill with Plan

Run these phases in order.

1. Grill and model the domain.
   - Call the Skill tool for `grilling` and `domain-modeling`, passing the user's request unchanged.
   - Completion: the user has explicitly confirmed the requirements, and glossary or ADR changes are saved.

2. Write the plan.
   - Call the Skill tool for `writing-plans` with the confirmed scope, decisions, constraints, and document paths as the authoritative input.
   - Completion: the plan is saved and the user has explicitly approved it. No application code changes before that approval gate.

3. Execute the plan.
   - Call the Skill tool for `executing-plans` with the approved plan. It owns everything through the batch commit.
   - If the plan and implementation disagree about the work, stop and ask for a plan amendment.
   - Completion: `executing-plans` has returned after its batch commit, or is waiting at its own approval gate.
