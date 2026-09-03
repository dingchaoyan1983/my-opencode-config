---
name: grill-with-plan
description: Use when a user has an unclear plan, design, or implementation request and needs its assumptions, tradeoffs, and execution scope clarified before coding.
license: MIT
metadata:
  author: dane.ding
  version: "1.0"
---

# Grill With Plan

Use this skill to turn an unclear idea, design, or implementation request into a clarified and approved execution plan before any code changes are made. The workflow deliberately separates requirements clarification, plan approval, and plan execution.

## Mandatory Gate

**The user's explicit confirmation is a hard gate that must never be bypassed.** The entire purpose of this skill is to guarantee code changes happen ONLY after the user has seen and approved the plan. Auto-executing the plan right after generating it defeats that purpose and is strictly forbidden.

- You MUST NOT execute the plan or make any code changes until the user has explicitly confirmed the plan and asked to proceed.
- Generating a plan does NOT authorize execution. "The plan is ready" is a stopping point, not a launch point.
- End the planning turn after presenting the plan and asking for confirmation. Do not continue with further tool calls, do not "helpfully" start executing, and do not pre-verify anything that would run the plan.

## Workflow

1. Call the Skill tool with `grill-with-docs`.
2. Follow `grill-with-docs` completely until all important questions, assumptions, tradeoffs, and domain terms are clarified.
3. Do not implement code immediately after the grilling phase, even if the requested implementation seems obvious.
4. Announce that you are using the `writing-plans` skill to create the implementation plan.
5. Call the Skill tool with `writing-plans`, using the clarified requirements and decisions from `grill-with-docs` as its input.
6. Follow `writing-plans` completely. Map the files, interfaces, bite-sized implementation steps, tests, verification commands, and self-review into a plan saved under `docs/superpowers/plans/`.
7. Present the generated plan path and a concise summary to the user.
8. **STOP.** Ask the user to review and explicitly confirm that the plan is correct before execution. End your response here — no further tool calls.
9. **Wait.** Do not interpret acknowledgement, review-only feedback, or an ambiguous "okay" as permission to execute.
10. If the user changes requirements, update the plan with `writing-plans` and ask for confirmation again.
11. Only after explicit confirmation, call the Skill tool with `writing-plans` again to reload and reconcile the confirmed plan before execution. This second call must not silently regenerate, expand, or execute the plan; preserve the approved scope and use the plan's existing Execution Handoff.
12. Ask the user to choose an execution approach if one was not already selected: `subagent-driven-development` for execution in the current session with fresh subagents and reviews, or `executing-plans` for inline/batch execution with checkpoints. If the user gives no preference, use `subagent-driven-development` when subagents are available; otherwise use `executing-plans`.
13. Call the selected execution skill and execute the confirmed plan. `writing-plans` documents and reconciles the plan; it does not implement application code.

## Hard Rules (violating these rules fails the workflow)

- Do not write or modify application code during the grilling phase.
- Do not write or modify application code during the planning phase except for creating or updating the plan document itself.
- **Do not execute the plan automatically after creating it — never.** The post-confirmation `writing-plans` invocation may only happen in response to an explicit user execution command in a later turn, and it must only reload/reconcile the approved plan.
- End your planning turn immediately after presenting the plan and the confirmation prompt. Do not call the post-confirmation `writing-plans` invocation, do not start editing files, and do not call any execution skill.
- Wait for an explicit user confirmation — e.g. "confirm", "execute", "start", "go ahead", "follow this plan", or "run it" — before the post-confirmation `writing-plans` invocation and execution skill. The user must be agreeing to EXECUTE, not merely acknowledging the plan.
- If the user's reply is review-only or ambiguous ("I'll take a look", "wait", "I have an objection", "okay", or "understood"), do NOT execute. Ask again for explicit confirmation.
- If the user changes requirements while reviewing the plan, update the plan first, then ask for confirmation again.
- If the user asks to skip grilling, use `writing-plans` directly instead of this skill, then apply its normal Execution Handoff.

## Output Contract

After the planning invocation of `writing-plans` finishes, respond with:

- The generated plan file path.
- A short summary of the plan.
- A clear prompt asking the user to review and explicitly confirm the plan, e.g. "The plan is ready. Please review and confirm it before I begin execution."
- The two available execution approaches, unless the user already selected one.

Do not paste the full plan content into the chat unless the user asks for it.

## Self-Check (complete each check before ending the turn)

Before ending your turn after presenting the plan, verify ALL of the following:

- [ ] I did NOT invoke the execution phase.
- [ ] I did NOT modify any application code.
- [ ] I ended my response with a clear question asking the user to confirm execution.
- [ ] The user has NOT yet given an explicit execution command for this plan.

If any check fails, stop immediately: undo any premature action and wait for the user.
