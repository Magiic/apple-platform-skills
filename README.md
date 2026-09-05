# apple-platform-skills
Defines Apple platform skills use for my personal and professional projects around apple platforms development.

## Architecture

Defines feature-based Swift Packages with separate Interface and Feature implementation modules, app-level composition, typed communication, SOLID boundaries, and compilation dependency guidance. Internal architecture stays flexible: SwiftUI views default to at most 300 lines, with ViewModels introduced for reusable or independently testable presentation logic.

## Coding style

Specifies coding style to use across my projects.

## UI

Specifies how to create user interface.

## Testing

Defines behavioral Swift Testing and XCTest UI-test conventions, with isolated test doubles and deterministic fixtures.

## Specialized skills

| Skill | Use for |
| --- | --- |
| [apple-navigation-deeplinks](apple-navigation-deeplinks/SKILL.md) | Typed routing, Universal Links, startup/authentication readiness, and route tests |
| [apple-app-clips](apple-app-clips/SKILL.md) | Focused App Clip targets, invocation, shared modules, and reliable handoff to the full app |
| [apple-swiftui-previews](apple-swiftui-previews/SKILL.md) | Deterministic preview scenarios, isolated dependencies, and canvas troubleshooting |
| [apple-widgets](apple-widgets/SKILL.md) | WidgetKit extension boundaries, shared data, timelines, app links, and family previews |

All skills support new projects and existing apps independently. Examples are fictional; no reference application is required. Each skill contains its essential rules and links to its own supporting references. Install only the skills useful to your work.

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

```text
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
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo Magiic/apple-platform-skills \
  --path apple-app-architecture \
    apple-coding-style \
    apple-ui \
    apple-swift-testing \
    apple-navigation-deeplinks \
    apple-app-clips \
    apple-swiftui-previews \
    apple-widgets
```

2. Restart Codex to pick up the new skills.
