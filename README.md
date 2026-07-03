# Skills

[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

A collection of reusable AI agent skills for software engineering workflows. Each skill provides structured instructions and conventions that guide AI agents through specific tasks — from project architecture to code explanation.

## What is a skill?

A skill is a Markdown file (`SKILL.md`) containing structured instructions, conventions, and workflows for an AI agent. Skills are loaded by compatible tools such as [opencode](https://opencode.ai) or [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview), and inject task-specific guidance into the agent's context.

This repository follows the [agent skills specification](https://github.com/anthropics/skills) for portability across platforms.

> [!TIP]
> Skills are **not** plugins or extensions. They don't add new capabilities to the agent — they provide specialized instructions that help the agent perform better at specific tasks.

## Installation

Skills can be installed using the `skills` CLI or any compatible agent tool that supports remote skill sources.

```bash
# Install a single skill from this repository
npx skills add https://github.com/pedromrtz/skills --skill <skill-name>
```

## Available skills

| Skill | Description |
|---|---|
| [conventional-commit](skills/conventional-commit/) | Generates standardized commit messages following the Conventional Commits specification, with XML-based structure and validation rules. |
| [feature-based-architecture](skills/feature-based-architecture/SKILL.md) | Design, review, or refactor frontend projects using feature-based architecture with clear module boundaries, shared UI conventions, and scalability rules. |
| [git-automation](skills/git-automation/SKILL.md) | Safe, autonomous Git operations for coding agents — commit changes, resolve merge conflicts, amend commits, and more with safety-first validation. |
| [mentor-for-dummies](skills/mentor-for-dummies/SKILL.md) | Explains any software project in beginner-friendly, plain language for non-programmers. Defines every technical term with analogies and progressive disclosure. |
| [plan-executor](skills/plan-executor/SKILL.md) | Executes a `PLAN.md` checklist continuously and autonomously until the project is fully implemented, validated, and production-ready. |
| [plan-generator](skills/plan-generator/SKILL.md) | Generates comprehensive, implementation-ready development blueprints as exhaustive `PLAN.md` checklists from project descriptions. |

## Usage

Once installed, skills are loaded automatically when the agent detects a matching task. For example, installing the `git-automation` skill will guide the agent when you ask it to commit changes, without any additional configuration.

Skills can also be loaded explicitly in supported tools:

```bash
# Load a skill in opencode
opencode --skill <skill-name>

# Reference a local skill file in Claude Code
claude --skill /path/to/skill/SKILL.md
```

## Repository structure

```
skills/
├── skills/                        # Available skills (one directory per skill)
│   ├── conventional-commit/
│   ├── feature-based-architecture/
│   ├── git-automation/
│   ├── mentor-for-dummies/
│   ├── plan-executor/
│   └── plan-generator/
├── .agents/skills/                # Internal agent tooling (skill-creator)
├── skills-lock.json               # Lockfile tracking installed skill sources and hashes
├── .gitignore
└── README.md
```

## Creating a skill

1. Create a directory under `skills/<skill-name>/`
2. Add a `SKILL.md` file with YAML frontmatter containing `name` and `description` fields
3. Write the skill instructions in Markdown following the [skill authoring guide](https://github.com/anthropics/skills)

**Frontmatter template:**

```yaml
---
name: your-skill-name
description: A clear description of when to use this skill.
---
```
