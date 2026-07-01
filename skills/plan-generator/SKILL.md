---
name: plan-generator
description: Generates comprehensive, implementation-ready development blueprints as exhaustive PLAN.md checklists. Use when the user wants to plan a software project, create a technical blueprint, design a system architecture, generate an implementation plan, produce a project execution checklist, or needs a detailed development specification. Also trigger when the user asks "how should I build X", "plan a project for Y", "create a technical plan for", "design a system for", "what's the architecture for", "I need a blueprint for", or describes a software project they want to build and needs a concrete, step-by-step execution plan. Use even if the user doesn't explicitly say "plan" or "blueprint" but is clearly describing a system they need built.
---

# Plan Generator

This skill transforms a project description into an exhaustive, executable implementation checklist (PLAN.md) that serves as the single source of truth for an autonomous AI agent building the system.

## Why this approach

A traditional project plan organized by phases and timelines assumes a human team with handoffs and meetings. An autonomous agent operates continuously — it needs a flat, dependency-ordered checklist where every item is small enough to execute and verify in one session. The checklist format also means the agent always knows exactly what remains, what's done, and what new work emerges during execution.

The breadth of coverage (architecture through operations) prevents the agent from making implicit decisions that later cause rework. By specifying technology choices, error handling patterns, and security requirements upfront, the plan eliminates whole categories of "should I do this?" deliberation.

## Process

### Step 1: Understand the project

Read and analyze the user's project description. Before generating any output, form a mental model of:

- What the system does and who uses it
- The core domain concepts and business logic
- Technical constraints (language runtime, hosting, existing systems)
- Non-functional requirements (scale, performance, security, compliance)
- Integration points and external dependencies

If the description is ambiguous on any of these, make reasonable, production-conservative assumptions rather than asking clarifying questions. Document those assumptions in the plan itself.

### Step 2: Generate PLAN.md

Create a file named `PLAN.md` in the **workspace root directory** (not in a subdirectory, not alongside source files — at the very top of the working project). Overwrite any existing PLAN.md after informing the user.

Write PLAN.md using this exact structure:

```markdown
# [Project Name] — Implementation Plan

> **Single source of truth for autonomous execution.** Every task must be
> completed, verified, and checked off before the project is considered done.
> New tasks discovered during execution must be appended immediately.

## Assumptions and Constraints
- List every assumption made during planning (tech stack, scale, environment)
- List every constraint (must use X, cannot use Y)

## Domain Model (optional — only if the project has significant data modeling)
- Core entities and their relationships
- Key attributes and invariants

## System Architecture
- Component diagram (text-based)
- Service responsibilities and boundaries
- Data flow between components
- External integrations

## Repository and Project Structure
- Monorepo or multi-repo decision
- Directory layout with every top-level folder explained
- Package/module naming conventions

## Technology Stack
- Language + runtime + version
- Framework choices with justification
- Key libraries and their purpose

## Implementation Checklist

A flat, exhaustive list of `- [ ]` and `- [x]` items. Follow these rules:

1. **Every item must be atomic** — one concrete action that can be completed in one session and independently verified.
2. **No nesting beyond 2 levels** — use indentation for grouping only, not for subtask hierarchy that obscures progress.
3. **Group by domain function, not by role** — e.g., "Authentication" as a group containing all tasks (code, config, test, deploy) rather than separate "Backend" and "DevOps" sections each with auth subtasks.
4. **Include verification tasks** — after every implementation group, add a checklist item that verifies the work (e.g., `- [ ] Verify that login returns JWT with correct claims and expiry`).
5. **Cover every aspect** — use the categories below as a checklist to ensure no gaps.

Required coverage categories (each must have at least one item, skip only if genuinely N/A):

| Category | What to include |
|---|---|
| Core domain logic | Models, business rules, invariants, state machines |
| API / endpoints | Route definitions, request/response schemas, error codes |
| Data layer | Schema migrations, queries, transactions, indexing |
| Authentication & authorization | Login, sessions/JWT, RBAC, OAuth if applicable |
| Validation & error handling | Input validation, error types, error responses, retry logic |
| Frontend / UI | Pages, components, state management, routing |
| Infrastructure | Cloud resources, containers, environment provisioning |
| Configuration management | Env vars, feature flags, config files per environment |
| Secrets management | Secret storage, rotation, access control |
| CI/CD | Build steps, test stages, deployment pipeline, rollback |
| Observability | Logging, metrics, tracing, health checks |
| Alerting & monitoring | SLOs, alert rules, dashboards |
| Testing | Unit, integration, E2E, contract, load tests |
| Performance & scalability | Caching, connection pooling, pagination, async processing |
| Resilience & fault tolerance | Retries, circuit breakers, graceful degradation, idempotency |
| Backup & recovery | Backup schedule, restore procedures, RPO/RTO targets |
| Security | HTTPS, CORS, CSP, dependency scanning, input sanitization |
| Documentation | API docs, runbooks, architecture decision records |
| Operations | Deployment runbooks, troubleshooting guides, run scripts |
| Upgrade & migration paths | Schema migration strategy, breaking change policy |
| Automation | Scripts for repetitive tasks, DB seeding, data migration |

### Step 3: Write each checklist item with precision

Bad items are vague and skip verifiable output. Good items specify exactly what to produce and how to verify it.

**Bad:**
```
- [ ] Implement authentication
- [ ] Set up database
- [ ] Add tests
```

**Good:**
```
- [ ] Create `src/middleware/auth.ts` that validates JWT from `Authorization: Bearer <token>` header
- [ ] Run migration `001_create_users` with columns: id (uuid PK), email (unique, not null), password_hash (not null), created_at
- [ ] Write unit test for `calculateTax()` covering: zero income, negative income, bracket boundaries, max bracket
```

### Step 4: Save and present

Write PLAN.md to the workspace root. After saving, present a brief summary to the user highlighting:
- Total task count
- Major component groups
- Key technology decisions made
- Any risks or assumptions worth noting

## Output quality guidelines

- **Concrete over abstract**: Name specific libraries, specific file paths, specific column names.
- **Cover the unhappy path**: For every feature, include error handling, edge cases, and failure modes.
- **Production-ready defaults**: Assume HTTPS everywhere, environment-variable-based config, structured logging, health checks, graceful shutdown — even for small projects.
- **No summaries**: The plan IS the execution guide. It should be long and detailed. A 500-line PLAN.md for a medium project is normal.
- **Self-documenting**: The plan should be readable by another agent with no context beyond the plan itself.
