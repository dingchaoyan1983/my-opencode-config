---
name: update-plan
description: Use when a user needs to iteratively revise an existing plan document under .agents/plan/ based on natural-language feedback.
license: MIT
metadata:
  author: dane.ding
  version: "1.0"
---

# Update Plan

Read the specified plan document under `.agents/plan/`, understand its contents, and iteratively revise it based on the user's natural-language feedback.

**Input**: The plan file path or file name, plus optional revision requirements.
- Resolve paths the same way as `run`:
  - Absolute path -> use it directly.
  - Relative path -> resolve it relative to the current project root.
  - File name only, such as `2026-07-24-admin-start-prod.md` -> look under `.agents/plan/` and append `.md` automatically.
  - Empty -> prefer the **most recently generated or referenced plan file in the current conversation**. If no plan file has appeared in the conversation, use `list_dir` to read `.agents/plan/` and `question` tool to let the user choose from its `.md` files. If the directory does not exist or is empty, tell the user to generate a plan with the `plan` skill first, then stop.
- Treat the remaining text after the file name as the initial revision request, if present.

Do not continue until the plan file to modify is identified.

---

## Workflow

### 1. Locate the plan file

Parse the input in this order:

- Absolute path -> use it directly.
- Relative path -> resolve it relative to the current project root.
- File name only, such as `2026-07-24-admin-start-prod.md` or `2026-07-24-admin-start-prod` -> look under `.agents/plan/` and append `.md` automatically.
- Empty -> prefer the **most recently generated or referenced plan file in the current conversation**. If no plan file has appeared in the conversation, use `list_dir` to read `.agents/plan/` and `question` tool to let the user choose a plan file.

Stop and notify the user if locating the file fails.

### 2. Read and understand the plan file

Use `read_file` to read the complete selected plan file.

Output a summary of the current plan in the conversation:
- File name and path.
- Level-one heading.
- Key points from the current-state section.
- Task list (number + description).
- Total line count.

Ask the user to confirm that the current content is understood.

### 3. Obtain revision requirements

- If the input contains text after the file name, use it as the initial revision request.
- Use `question` tool (open-ended) to ask: "What changes would you like to make to this plan?"
- After each revision, ask whether there are more changes, supporting multiple iterations.

### 4. Apply revisions

For each revision request:

1. **Locate the change**: Based on step 2, identify where to modify, such as a Task, code block, or parameter-source list.
2. **Reread the file**: Before editing, use `read_file` again to ensure the operation uses the latest content.
3. **Apply the change**: Use `replace_string_in_file` for an exact string replacement.
4. **Show the change**: Output a summary of the revision, including location and key changes.
5. **Get user confirmation**: Use `question` tool (yes/no) to ask, "Is this change correct? Would you like to continue revising?"
   - Yes -> continue to the next revision or finish.
   - No -> use `replace_string_in_file` to undo the revision by restoring the old content.

### 5. Summarize

After all revisions are complete:

- Output the final plan's summary structure, including its title and Task list.
- List all revisions made during the session.
- Tell the user that they can use the `run` skill to execute the revised plan or commit the changes manually in Git.

---

## Format Constraints

Follow the language requirements defined in the current project's `AGENTS.md` when revising plan documents. Apply those requirements to titles, headings, descriptions, summaries, and revised content. Keep code, file paths, commands, API names, identifiers, and other technical names in their original form.

1. **Modify only plan files under `.agents/plan/`**: This command does not modify other project files.
2. **Wait for confirmation after every revision**: Do not continue to the next revision without user confirmation.
3. **Support undo**: If the user is dissatisfied with a revision, restore the state from before that revision.
4. **Preserve the plan format**: Revised content must follow the plan document's Markdown structure (`# Title` -> `## Current State` -> `## Task N:`).
5. **Do not commit to Git automatically**: The user commits all changes manually.

---

## Behavioral Guidelines

- Reread the plan before every revision to confirm its current content and prevent inconsistency from concurrent edits.
- Use an exact matching `oldString` when editing to avoid unintended changes.
- If the user's requirement is ambiguous, clarify it with `question` tool before editing.
- Do not optimize or refactor unmentioned sections without authorization.
- After revising, recommend that the user use the `run` skill to verify the plan's executability.
