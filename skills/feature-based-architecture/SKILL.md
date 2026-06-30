---
name: feature-based-architecture
description: Use this skill to design, review, or refactor frontend projects using feature-based architecture, clear module boundaries, shared UI conventions, service layers, validation, state management, imports, naming, and scalability rules.
---

# Feature-Based Architecture Skill

Use this skill when designing, reviewing, or refactoring a frontend codebase that should be organized by business capability instead of technical file type.

## When to use this skill

Use this skill for tasks such as:

- Creating a new frontend project structure.
- Reviewing whether an existing project follows feature-based architecture.
- Deciding where to place components, hooks, services, schemas, types, utilities, assets, or styles.
- Defining boundaries between application routes, features, shared UI, services, and utilities.
- Enforcing consistent naming, imports, validation, state management, and dependency direction.

## Core principles

- Use a **feature-based architecture** as the primary organizational strategy.
- Organize code by **business functionality**, not by file type.
- Keep each feature as self-contained as possible.
- Avoid unnecessary coupling between features.
- Reuse only truly generic code through shared modules.
- Prefer composition over inheritance.
- Favor explicitness and consistency over clever abstractions.
- Maintain a predictable folder structure across the entire project.

## Recommended directory structure

```text
src/
├── app/
├── features/
│   ├── auth/
│   ├── users/
│   ├── products/
│   └── ...
├── components/
│   ├── ui/
│   ├── layout/
│   └── shared/
├── providers/
├── hooks/
├── lib/
├── services/
├── schemas/
├── constants/
├── types/
├── config/
├── assets/
└── styles/
```

## Directory responsibilities

### `app/`

The `app` directory is responsible only for routing and page composition.

Allowed content:

- Routes.
- Layouts.
- Route groups.
- Loading pages.
- Error pages.
- Not found pages.

Do not place the following inside `app/`:

- Business logic.
- API logic.
- Data transformation.
- Validation.
- Complex state management.

Pages should compose existing components instead of implementing business logic directly.

### `features/`

Each feature represents one business domain.

Example:

```text
features/
├── auth/
├── dashboard/
├── products/
└── users/
```

Each feature owns everything related to itself.

Recommended internal feature structure:

```text
features/users/
├── components/
├── hooks/
├── services/
├── schemas/
├── types/
├── utils/
└── constants/
```

Rules:

- Features must be independent.
- A feature should expose only its public API.
- Avoid importing internal files from another feature.
- Cross-feature communication should happen through exported interfaces.
- Only expose public modules through an index file when necessary.
- A feature must not expose its internal implementation.

### `components/`

The `components/` directory contains reusable UI that is not owned by a specific feature.

Recommended structure:

```text
components/
├── ui/
├── layout/
└── shared/
```

#### `components/ui/`

Contains generated or primitive UI components, such as shadcn/ui components.

Rules:

- Keep UI primitives generic.
- Avoid feature-specific logic.
- Do not modify generated behavior unless customization is required.

#### `components/layout/`

Contains application layout components.

Examples:

- Sidebar.
- Header.
- Footer.
- Navigation.
- Page containers.

#### `components/shared/`

Contains reusable application components.

Examples:

- DataTable.
- ConfirmDialog.
- EmptyState.
- LoadingSpinner.
- SearchBar.

Rules:

- Shared components must not depend on a specific feature.
- Shared components should receive data and callbacks through props.
- Shared components should not perform feature-specific API calls.

### `services/`

The `services/` directory contains API communication shared across the application.

Responsibilities:

- HTTP requests.
- Request mapping.
- Response mapping.

Rules:

- Never call `fetch` directly inside pages.
- Never duplicate API calls.
- Keep services stateless.
- Keep request and response transformations centralized.

Feature-specific service code should live inside the corresponding feature.

### `hooks/`

The `hooks/` directory contains reusable custom hooks.

Rules:

- Hooks should encapsulate reusable behavior.
- Avoid feature-specific hooks unless they belong inside that feature.
- Generic hooks may live in the root `hooks/` directory.

Examples:

```text
useDebounce()
usePagination()
useTheme()
```

### `providers/`

The `providers/` directory contains application providers.

Examples:

- TanStack Query.
- Theme.
- Authentication.
- Localization.

Rules:

- Providers should be registered from the application root.
- Provider setup should be centralized.
- Avoid scattering provider logic across routes or pages.

### `schemas/`

The `schemas/` directory contains shared Zod schemas.

Responsibilities:

- Validation.
- Parsing.
- Type inference.

Rules:

- Validation must be centralized.
- Avoid inline validation.
- Feature-specific schemas should remain inside the feature.

### `types/`

The `types/` directory contains shared TypeScript types.

Examples:

- DTOs.
- Enums.
- Interfaces.
- Shared models.

Rules:

- Feature-specific types should remain inside the feature.
- Shared types must represent concepts used by multiple features or application-level modules.

### `lib/`

The `lib/` directory contains framework-independent utilities.

Examples:

- Utility functions.
- Date helpers.
- Formatters.
- Generic helpers.

Rules:

- No React code.
- No UI code.
- No business logic.
- Keep utilities generic and reusable.

### `constants/`

The `constants/` directory contains immutable application constants.

Examples:

- Routes.
- Roles.
- Permissions.
- Regex patterns.
- Configuration values.

Feature-specific constants should remain inside the feature.

### `config/`

The `config/` directory contains application configuration.

Examples:

- Environment configuration.
- API configuration.
- Feature flags.

Rules:

- Keep environment access centralized.
- Avoid reading environment variables directly throughout the application.

### `assets/`

The `assets/` directory contains project assets.

Recommended structure:

```text
assets/
├── images/
├── icons/
├── fonts/
└── logos/
```

### `styles/`

The `styles/` directory contains global styling.

Examples:

- Global CSS.
- Theme overrides.
- Custom animations.

Rules:

- Prefer Tailwind utilities over custom CSS whenever possible.
- Keep global styles minimal.
- Avoid placing component-specific styles in global files.

## Feature rules

Each feature may contain:

```text
components/
hooks/
services/
schemas/
types/
utils/
constants/
```

Rules:

- A feature must own its domain-specific UI, hooks, services, schemas, types, utilities, and constants.
- A feature must not expose its internal implementation.
- A feature should expose only the modules that other areas are allowed to consume.
- Cross-feature usage should happen through stable public exports.
- Promote code to shared modules only after it is reused by more than one feature.

## Component rules

- Components should have a single responsibility.
- Keep components as small as practical.
- Separate presentation from business logic.
- Extract repeated UI into reusable components.
- Avoid deeply nested JSX.
- Prefer explicit props over implicit dependencies.

## State management

Follow this order when choosing state management:

1. Local component state.
2. React Context for application-wide concerns.
3. TanStack Query for server state.

Rules:

- Avoid introducing global state libraries unless there is a demonstrated need.
- Keep server state out of manual global stores when TanStack Query can manage it.
- Keep state as close as possible to where it is used.

## API access

- All backend communication belongs inside services.
- Never mix HTTP calls with rendering.
- Keep request and response transformations centralized.
- Keep API-specific details away from UI components.

## Validation

- Use Zod for all runtime validation.
- Use React Hook Form together with Zod for forms.
- Never duplicate validation logic.
- Keep validation schemas close to the domain they validate.

## Imports

Prefer absolute imports.

Example:

```text
@/features/users/components/UserTable
```

Rules:

- Avoid long relative paths.
- Avoid importing internal files from another feature.
- Prefer public exports when consuming feature modules.

## Naming conventions

Directories:

```text
lowercase
```

Files:

```text
kebab-case
```

React components:

```text
PascalCase
```

Hooks:

```text
useSomething
```

Types:

```text
PascalCase
```

Constants:

```text
UPPER_SNAKE_CASE
```

## Scalability guidelines

- Prefer adding a new feature instead of expanding existing unrelated modules.
- Keep shared modules generic.
- Promote code to shared only after it is reused.
- Avoid premature abstractions.
- Keep dependencies flowing inward:

```text
app -> features -> shared components -> utilities
```

Never allow lower-level modules to depend on higher-level ones.

## General rules

- Favor readability over cleverness.
- Keep files focused.
- Remove dead code.
- Keep dependencies minimal.
- Follow consistent naming.
- Maintain clear separation of concerns.
- Organize by business capability first and implementation detail second.
