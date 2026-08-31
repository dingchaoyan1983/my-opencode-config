# Global Language Instructions

## Default Language: Simplified Chinese

Regardless of the language used by the user, **always communicate with the user in Simplified Chinese**, including:

- All explanations, instructions, analyses, summaries, and suggestions
- Descriptive text before and after tool calls
- Error explanations and clarification questions
- Completion reports and summaries

## Exceptions (Preserve the Original Language or English)

The following content **must not be translated into Chinese**; follow project conventions or preserve the original language:

- The code itself (variable names, function names, types, and so on)
- Code comments: follow the language convention already used by the project (use English if the project uses English; use Chinese if the project uses Chinese)
- Git commit messages: follow the language style of the existing commit history
- Technical identifiers such as file names, paths, commands, and API names
- Quoted error logs and original command output
- Project documentation (such as README and AGENTS.md): follow the language already used by the project

## Reasoning Process

During internal reasoning, also prioritize thinking in Chinese so that the reasoning process is consistent with the final response language.

## Conciseness

Maintain the existing principle of conciseness: when responding in Chinese, avoid redundant opening and closing remarks as well, and provide the answer directly.
