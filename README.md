# apple-platform-skills
Defines Apple platform skills use for my personal and professional projects around apple platforms development.

## Architecture

Specifies macro and micro architecture used in my projects?

## Coding style

Specifies coding style to use across my projects.

## UI

Specifies how to create user interface.

## Testing

Describes how to make tests.

## How to use

1. Install node and use the command:

```
npx openskills install https://github.com/Magiic/apple-platform-skills.git
```

Then sync with your `Agents.md`

```
 npx openskills sync
```

To use in cursor with codex extension for example and not use with openskills update AGENTS.md usage with the following:

````
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively.

IMPORTANT:
- Skills are stored as files in this repository.
- You MUST read the skill files directly from the filesystem.
- Do NOT require running any CLI command (no `npx`, no `openskills read`).

How to use skills (portable across tools):
- Open the skill's `SKILL.md` file from its path.
- Apply its rules and conventions to the task.
- If needed, also open any files inside the skill folder (e.g. `references/`).

Usage notes:
- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
- Each skill invocation is stateless
</usage>
```

## How to install in Codex

1. Run the installer:

```bash
~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo Magiic/apple-platform-skills \
  --path apple-app-architecture \
  --path apple-coding-style
  --path apple-ui
  --path apple-swift-testing
```

2. Restart Codex to pick up the new skills.
