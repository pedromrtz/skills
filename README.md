# Skills

A collection of reusable AI agent skills for software engineering workflows. Each skill provides specialized instructions and conventions that can be loaded on demand to guide agents through specific tasks — from project architecture to clean code.

## What is a skill?

A skill is a Markdown file (`SKILL.md`) containing structured instructions, conventions, and workflows for an AI agent. Skills are loaded by compatible tools (such as [opencode](https://opencode.ai)) and inject task-specific guidance into the agent's context.

This repository follows the [agent skills specification](https://github.com/anthropics/skills) for portability across platforms.

## Installation

Skills can be installed from this repository using the `skills` CLI or any compatible agent tool that supports remote skill sources.

```bash
# Install a skill from this repository
npx skills add https://github.com/pedromrtz/skills --skill <skill>
```

## Available skills

| Skill | Description |
|---|---|
| [feature-based-architecture](skills/feature-based-architecture/SKILL.md) | Use this skill to design, review, or refactor frontend projects using feature-based architecture, clear module boundaries, shared UI conventions, service layers, validation, state management, imports, naming, and scalability rules. |
| [git-automation](skills/git-automation/SKILL.md) | Provides safe, autonomous Git operations for coding agents. Use when the user asks to commit changes, resolve merge conflicts, amend commits, or perform any Git workflow that involves staging, committing, merging, conflict resolution, or rewriting history. |
| [plan-executor](skills/plan-executor/SKILL.md) | Executes a `PLAN.md` checklist continuously and autonomously until the project is fully implemented, validated, and production-ready. Use when the user wants to build a project, implement a plan, execute a blueprint, or follow a development checklist. |
| [plan-generator](skills/plan-generator/SKILL.md) | Generates comprehensive, implementation-ready development blueprints as exhaustive `PLAN.md` checklists. Use when the user wants to plan a software project, design a system architecture, or create a technical implementation plan. |

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
