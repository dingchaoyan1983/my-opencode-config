---
name: grill-with-plan
description: Use when a user has an unclear plan, design, or implementation request and needs its assumptions, tradeoffs, and execution scope clarified before coding.
license: MIT
metadata:
  author: dane.ding
  version: "1.0"
---

# Grill With Plan

Use this skill to turn an unclear idea, design, or implementation request into a clarified and approved execution plan before any code changes are made.

## Mandatory Gate

**The user's explicit confirmation is a hard gate that must never be bypassed.** The entire purpose of this skill is to guarantee code changes happen ONLY after the user has seen and approved the plan. Auto-executing the plan right after generating it defeats that purpose and is strictly forbidden.

- You MUST NOT call the `run` skill, and MUST NOT make any code changes, until the user has explicitly told you to execute the plan.
- Generating a plan does NOT authorize execution. "The plan is ready" is a stopping point, not a launch point.
- End your turn after presenting the plan and asking for confirmation. Do not continue with further tool calls, do not "helpfully" start executing, do not pre-verify anything that would run the plan.

## Workflow

1. Call the Skill tool with `grill-with-docs`.
2. Follow `grill-with-docs` completely until all important questions, assumptions, tradeoffs, and domain terms are clarified.
3. Do not implement code immediately after the grilling phase, even if the requested implementation seems obvious.
4. Call the Skill tool with `create-plan` and use the clarified requirements from the grilling phase as the input.
5. Generate and save a plan file under `.agents/plan/` according to the `create-plan` skill.
6. Present the generated plan path and a concise summary to the user.
7. **STOP.** Ask the user to review and confirm the plan before execution. End your response here — no further tool calls.
8. **Wait.** Only after the user explicitly confirms the plan, call the Skill tool with `run` and execute the confirmed plan.

## Hard Rules (violating these rules fails the workflow)

- Do not write or modify application code during the grilling phase.
- Do not write or modify application code during the planning phase except for creating or updating the plan document itself.
- **Do not run the plan automatically after creating it — never.** The `run` skill may only be invoked in response to an explicit user execution command in a later turn.
- End your turn immediately after presenting the plan and the confirmation prompt. Do not call `run`, do not start editing files, do not call any tool that would start executing the plan.
- Wait for an explicit user confirmation — e.g. "confirm", "execute", "start", "go ahead", "follow this plan", or "run it" — before calling `run`. The user must be agreeing to EXECUTE, not merely acknowledging the plan.
- If the user's reply is review-only or ambiguous ("I'll take a look", "wait", "I have an objection", "okay", or "understood"), do NOT execute. Ask again for explicit confirmation.
- If the user changes requirements while reviewing the plan, update the plan first, then ask for confirmation again.
- If the user asks to skip grilling, use `create-plan` directly instead of this skill.

## Output Contract

After `create-plan` finishes, respond with:

- The generated plan file path.
- A short summary of the plan.
- A clear prompt asking the user whether to execute the plan, e.g. "The plan is ready. Please review and confirm it before I begin execution."

Do not paste the full plan content into the chat unless the user asks for it.

## Self-Check (complete each check before ending the turn)

Before ending your turn after presenting the plan, verify ALL of the following:

- [ ] I did NOT call the `run` skill.
- [ ] I did NOT modify any application code.
- [ ] I ended my response with a clear question asking the user to confirm execution.
- [ ] The user has NOT yet given an explicit execution command for this plan.

If any check fails, stop immediately: undo any premature action and wait for the user.
