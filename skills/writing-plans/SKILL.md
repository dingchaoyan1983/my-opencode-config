---
name: writing-plans
description: "Create a reviewable implementation plan with concrete code and test examples before touching application code; use when a ticket is picked up or when implement needs a route from requirements to code."
---

# Writing Plans

Write the plan for one ticket: how the work gets built, at step level. The ticket holds what to build and how completion is judged; the plan holds the route through the work, detailed enough that execution is mechanical and reviewable.

The plan is a gate. It ends by putting itself in front of the user and waiting for explicit approval. Application code starts only after that approval.

## Where the plan lives

Read `docs/agents/issue-tracker.md` when it exists. For the local Markdown tracker, save the plan alongside its source spec and tickets at `.scratch/<feature-slug>/plans/<NN>-<slug>.md`. If the configured tracker document defines a different `Plans` location, follow that location; otherwise use the local Markdown convention instead of creating a separate top-level plans directory.

Read alongside the plan: the ticket, the spec it came from, the project's `CONTEXT.md` glossary, and any ADR covering the area. Use the project's domain vocabulary instead of inventing new terms.

## Language

Inspect the active conversation and project instructions before writing. Use the required language for explanatory prose, headings, task descriptions, rationale, and verification notes. Keep code, comments, paths, commands, API names, identifiers, package names, and technical literals unchanged unless project conventions require otherwise.

Before saving, verify that explanatory prose consistently uses the selected language and that code-related literals remain unchanged.

## Process

### 1. Read the sources

Read the ticket, spec, glossary, ADRs, and applicable project instructions in full before planning. Carry binding architectural decisions and acceptance criteria into the plan's `Global Constraints`; do not silently re-decide them in a task.

### 2. Map files and interfaces

Decide which files the change touches and where new boundaries belong before splitting tasks. Fold setup, configuration, and documentation into the task that needs them so each task leaves the project working.

### 3. Split the work into tasks

Create tasks at points where a reviewer could reject one task while accepting the next. Each task must produce a coherent, independently verifiable deliverable. Keep tasks small enough that each step takes roughly two to five minutes.

### 4. Write every task concretely

Every task includes:

- **`Files:`** exact paths to create, modify, or test, with ranges when useful.
- **`Interfaces:`** exact inputs from earlier tasks and outputs for later tasks, including names, parameters, and return types.
- **`Checks:`** the project's typecheck and the focused test files touched by the task. The last task also names the full test suite.
- Checkbox steps with one action each.
- The complete code, test code, or command required by each step.
- An `Expected:` line describing the result that proves the step worked.

Code examples and test examples are mandatory. A step that says what to do without showing how to do it is not ready for review.

Use vertical slices where possible: write one failing behavior test, run it red, add the smallest implementation, then run it green before moving to the next slice.

### 5. Self-review the plan

Read the plan back against its sources and fix every gap:

- Every spec decision and acceptance criterion maps to a task.
- Every task names its files, interfaces, and checks.
- The final task names the full test suite.
- Names, types, and signatures are identical wherever reused.
- Every step contains its actual code, test, or command and an expected result.
- The plan contains no placeholders such as `TBD`, `TODO`, "implement later", "add appropriate error handling", "write tests for the above", or "similar to Task N".
- The selected language is applied consistently to explanatory prose.

### 6. Save, present, and stop

Save the plan, then present:

- The plan path.
- The goal and architecture summary.
- The task list and files each task touches.
- Any assumptions or unresolved questions.

Stop and wait for explicit user approval or requested edits. Do not execute the plan or modify application code in this skill.

## Plan template

```markdown
# <NN>: <Ticket title>

**Goal:** one sentence on what this plan builds.

**Architecture:** the shape of the change and how data flows through it.

**Tech Stack:** the languages, frameworks, and test tools involved.

**Spec:** the source spec path or issue reference.

**Ticket:** the source ticket path or issue reference.

**Global Constraints:** binding rules copied from the spec.

**Checks:** the exact typecheck and test commands used below.

## Task 1: <Task name>

**Files:**

- Create: `path/to/new-file.ts`
- Modify: `path/to/existing.ts:40-72`
- Test: `path/to/existing.test.ts`

**Interfaces:**

- Consumes: `parseTemplate(source: string): TemplateNode[]` from `src/template-syntax.ts`
- Produces: `renderDocument(nodes: TemplateNode[], data: DocData): Buffer`

**Checks:**

- `pnpm typecheck`
- `pnpm vitest run path/to/existing.test.ts`

- [ ] **Step 1: Write the failing behavior test**

```ts
it("renders a greeting for the supplied name", () => {
	expect(formatGreeting("Ada")).toBe("Hello, Ada!");
});
```

Expected: the new case fails for the missing behavior.

- [ ] **Step 2: Run the focused test**

```bash
pnpm vitest run path/to/existing.test.ts
```

Expected: the new case is red for the reason stated above.

- [ ] **Step 3: Add the minimal implementation**

```ts
export function formatGreeting(name: string): string {
	return `Hello, ${name}!`;
}
```

Expected: the implementation satisfies the new behavior without changing unrelated behavior.

- [ ] **Step 4: Run the focused test and typecheck**

```bash
pnpm vitest run path/to/existing.test.ts
pnpm typecheck
```

Expected: both commands pass.
```