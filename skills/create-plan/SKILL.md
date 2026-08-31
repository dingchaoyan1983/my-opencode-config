---
name: create-plan
description: Use when a user needs a project task broken down into an actionable implementation plan saved under .agents/plan/.
license: MIT
metadata:
  author: dane.ding
  version: "1.1"
---
# Plan

Generate a task plan document and save it as a `.md` file under `.agents/plan/` at the current project root.

**Input**: The user's requirement description.

- If the user's input is `list` or `ls`, use the `read` tool on `.agents/plan/` (or `bash ls`) to list all `.md` files, and stop.
- If the input is empty, use `question` tool (open-ended, with no predefined options) to ask:

> "What task would you like to plan? Describe what you want to implement or modify."

Do not continue until the requirement is understood.

---

## Workflow

### 1. Read-only exploration (do not write application code)

Use read-only tools such as `glob`, `grep`, and `read` to investigate code, configuration, and documentation related to the requirement. Understand:

- The current project structure and technology stack.
- The relevant existing files, contents, patterns, and conventions.
- Reference files to compare against, such as similar features or modules.
- The source of configuration parameters, including the defining file or constant.

Explore sufficiently before writing the plan to avoid unsupported assumptions.

### 2. Generate the file name

Format: `YYYY-MM-DD-<task-name>.md`

- Date: Use the current date, such as `2026-07-24`.
- `<task-name>`: Create a short kebab-case name based on the requirement, such as `admin-start-prod`, `add-user-auth`, or `fix-booking-pagination`.
- Complete example: `2026-07-24-admin-start-prod.md`

### 3. Generate the plan document

Follow the following Markdown structure exactly. Every section is required and the order is fixed:

Follow the language requirements defined in the current project's `AGENTS.md` when writing the plan document. Apply those requirements to the plan title, headings, descriptions, and summaries. Keep code, file paths, commands, API names, identifiers, and other technical names in their original form.

```markdown
# <Short task title>

## Current State

<Describe the current state, context, reason for the change, relevant files/modules, missing pieces, and why the requirement is not currently satisfied.>

## Task 1: <Brief task description>

<Explain in detail what this task must do, such as creating a file, modifying configuration, or adding a function.>

<If code is involved, provide a complete code example in a fenced code block with a language tag:>

```js
const path = require('path');
// ... complete code
```

<For important parameters/configuration items, list their sources and rationale:>

- `<parameter-name>` -- from <specific location/definition> in <source file>. Example: `port: 3004` comes from `devServer.port` in `module-federation.config.js`.

## Task 2: <Brief task description>

<Use the same structure: detailed explanation + code example + parameter sources>

## Task 3: <Brief task description>

<Continue incrementing the number if there are more tasks>

```

### 4. Save the file

- Ensure that `.agents/plan/` exists under the current project root, creating it if needed. The command depends on the shell opencode runs (`shell` config in `opencode.json`):
  - POSIX shells (bash/zsh, default on macOS/Linux): run `mkdir -p .agents/plan`.
  - PowerShell on Windows: run `New-Item -ItemType Directory -Path .agents/plan -Force` (equals `mkdir -p`: creates all missing parents and does not error when the directory already exists; plain `mkdir` also works in PowerShell as an alias of `New-Item`, but `New-Item ... -Force` is the explicit, version-safe form).
- If a file with the same `.md` name already exists under `.agents/plan/`, use the `question` tool (yes/no) to ask whether to overwrite it. Stop if the user selects "no".
- **Write the plan in multiple small tool calls. Never write the whole document in one `write` call**: a single oversized tool-call payload frequently gets truncated mid-JSON, surfacing as `Invalid input for tool write: JSON parsing failed: Unterminated string`. Follow this sequence:
  1. First use `write` to create the file with the skeleton only: the `#` title, the `## Current State` section with its text, and every `## Task N:` heading with a one-line placeholder describing that task.
  2. Then use `edit` to replace each placeholder with that section's full content, one task at a time.
  3. Keep every single `write`/`edit` call well under roughly 10 KB of content; if a section (e.g. a large code block) is long, split it across several `edit` calls.
- The target file is `.agents/plan/YYYY-MM-DD-<task-name>.md`, using an absolute path based on the current project root.

### 5. Validate the plan format

After saving, use the `read` tool to reread the plan and check that it contains:

- A level-one heading `# `, with the heading on line 1.
- A `## Current State` section.
- At least one `## Task N:` section, with N starting at 1 and incrementing.
- A language tag on every code block, such as \`\`\`js, \`\`\`json, \`\`\`tsx, or \`\`\`bash.

If any of these elements is missing, output a warning and tell the user to edit the plan manually to add it.

### 6. Output a summary

After saving, output a brief summary in the conversation:

- The absolute path to the plan file.
- The number of tasks and a summary of the core changes.
- Tell the user to review/edit the plan and then use the `run` skill after confirmation.

Do not repeat the full plan in the conversation; it is already in the file.

---

## Format Constraints (must be followed exactly)

1. **Level-one heading `#`**: A short, one-sentence summary of the task.
2. **Required `## Current State` section**: Immediately after the title, describe the context and current state.
3. **Use `## Task N: <description>` for tasks**: Start N at 1 and increment it; each task gets one level-two heading.
4. **Tag code examples with a language**: Use \`\`\`js, \`\`\`json, \`\`\`tsx, \`\`\`bash, etc.; do not use untagged code blocks.
5. **Document sources and expected values for important parameters**: Use `- \`parameter-name\` -- from <specific location/definition> in <source file>`. For example:
   - `port: 3004` -- from the `devServer.port` definition in `module-federation.config.js`.
   - `appBuildPublicPath: "/child/admin"` -- from `output.publicPath` in `webpack.config.js`.
6. **Do not make application code changes**: This command only generates a plan document; it does not modify source code, configuration, or Git state.
7. **Base the plan on real exploration**: File paths, function names, and configuration items in the plan must come from the read-only exploration in step 1; do not invent them.

---

## Behavioral Guidelines

- Explore thoroughly in read-only mode to avoid guessing file paths or function signatures.
- Make the plan clear, specific, actionable, and suitable for user review.
- Make code examples complete and copyable; do not omit important parts.
- Preserve parameter-source explanations as a core requirement.
- Keep the file name short and meaningful for easy discovery.
```
