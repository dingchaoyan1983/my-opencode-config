---
description: Clarify an unclear implementation request and produce an approved execution plan before coding.
agent: plan
---

Use the `grill-with-plan` skill to handle this request.

Pass the user's input to the skill unchanged:

$ARGUMENTS

Follow the skill's workflow and hard rules exactly. Complete the grilling and planning phases, present the generated plan path and summary, ask the user for explicit execution confirmation, and stop. Never call the `run` skill or modify application code before that confirmation.
