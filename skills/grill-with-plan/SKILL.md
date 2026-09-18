---
name: grill-with-plan
description: "Clarify an implementation request with documented requirements, then hand the confirmed requirements to the implementation workflow."
disable-model-invocation: true
---

# Grill with Plan

Run these phases in order.

1. Call the Skill tool with `grill-with-docs` and pass the user's request unchanged.
   - Let it complete its grilling and domain-modeling workflow.
   - Wait for the user to confirm the shared understanding.
   - Completion: the user has explicitly confirmed the requirements, and any glossary or ADR changes from the phase are available to the next phase.

2. Call the Skill tool with `implement` and pass the confirmed requirements unchanged.
   - Carry forward the confirmed scope, decisions, constraints, and document paths as the authoritative implementation input.
   - Let `implement` own its plan, approval, execution, verification, review, ticket-resolution, and commit lifecycle.
   - Completion: `implement` has completed its lifecycle or is waiting at its own explicit approval gate. Preserve that gate.
