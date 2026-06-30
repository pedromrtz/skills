# Skills

A collection of reusable AI agent skills for software engineering workflows. Each skill provides specialized instructions and conventions that can be loaded on demand to guide agents through specific tasks — from project architecture to clean code.

## What is a skill?

A skill is a Markdown file (`SKILL.md`) containing structured instructions, conventions, and workflows for an AI agent. Skills are loaded by compatible tools (such as [opencode](https://opencode.ai)) and inject task-specific guidance into the agent's context.

This repository follows the [agent skills specification](https://github.com/anthropics/skills) for portability across platforms.

## Installation

Skills can be installed from this repository using the `skills` CLI or any compatible agent tool that supports remote skill sources.

```bash
# Example using opencode
skills install pmejia/skills/<skill-name>
```

## Available skills

| Skill | Description |
|---|---|
| [feature-based-architecture](skills/feature-based-architecture/SKILL.md) | Design, review, or refactor frontend projects using feature-based architecture with clear module boundaries, shared UI conventions, service layers, validation, state management, and scalability rules. |

## Repository structure

```
skills/
├── skills/              # Available skills (one subdirectory per skill)
│   └── <skill-name>/
│       └── SKILL.md     # Main skill definition
├── .agents/             # Internal agent tooling (skill-creator)
├── skills-lock.json     # Lockfile tracking installed skill sources and hashes
├── .gitignore
└── README.md
```

## Creating a new skill

1. Create a directory under `skills/<skill-name>/`
2. Add a `SKILL.md` file with frontmatter metadata (`name`, `description`)
3. Write the skill instructions in Markdown following the [skill authoring guide](https://github.com/anthropics/skills)

**Frontmatter template:**

```yaml
---
name: your-skill-name
description: A clear description of when to use this skill.
---
```

## License

MIT
