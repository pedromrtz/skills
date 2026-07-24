---
name: trilium
description: Use this skill whenever the user wants to search, read, create, update, organize, summarize, or restructure notes in their Trilium knowledge base through the Trilium MCP. Trigger strongly for explicit commands like "/trilium", "Trilium", "mi base de conocimiento", "busca en mis notas", "crea una nota", "guarda esto", "organiza esto", "mueve esta informacion", or any task involving the user's personal knowledge base, university notes, UTEC information, tasks, evaluations, resources, contacts, or daily captures. This skill should guide the agent to use the available trilium_* MCP tools instead of treating the request as a local filesystem task.
---

# Trilium Knowledge Base

Use the Trilium MCP as the source of truth for the user's personal knowledge base. The user often calls this with `/trilium` and may say "archivo" when they mean a Trilium note; interpret that as a note unless they explicitly ask for a local file on disk.

## Core behavior

Start by using the Trilium MCP tools, not filesystem tools, when the request is about notes, knowledge, study material, UTEC, personal information, tasks, links, contacts, evaluations, or anything already stored in Trilium.

For every task:

1. Determine whether the user wants to search/read, create, update, move/organize, delete, or build a dashboard/automation.
2. Inspect the relevant Trilium structure before acting unless the target note ID is already known from the current conversation.
3. Preserve the existing organization. Add notes to the most specific existing parent instead of creating duplicate top-level sections.
4. Prefer clear Spanish titles and Markdown content because this knowledge base is currently written mostly in Spanish.
5. Report the resulting note title, parent location, and note ID after creating, moving, or updating anything.

Ask one short clarification before acting only when the target location or meaning is genuinely ambiguous and a wrong write would create clutter or lose information.

## Available MCP capabilities

The Trilium MCP exposes tools for:

- Searching notes: `trilium_search_notes`
- Reading structure: `trilium_get_child_notes`, `trilium_get_subtree`
- Reading notes: `trilium_get_note`, `trilium_get_note_content`
- Creating notes: `trilium_create_note`
- Updating content: `trilium_set_note_content`, `trilium_edit_note_content`
- Renaming, moving, cloning, and deleting notes: `trilium_rename_note`, `trilium_move_note`, `trilium_clone_note`, `trilium_delete_note`
- Managing labels and relations: `trilium_get_attributes`, `trilium_set_attribute`, `trilium_delete_attribute`
- Working with attachments: `trilium_get_attachment`, `trilium_get_attachment_content`
- Finding icons: `trilium_search_icons`
- Loading Trilium-specific guidance: `trilium_load_skill`

Use `trilium_load_skill` when needed:

- `search_syntax` for complex searches with labels, relations, booleans, note properties, or ordering.
- `backend_scripting` for backend automation scripts.
- `frontend_scripting` for UI widgets or frontend scripts.
- `dashboards` for dashboard notes and widgets.

## Current Knowledge Base Map

This map reflects the observed structure and should be treated as a starting point. Re-check the live tree when location matters because the user's Trilium may evolve.

Top level:

- `Calendar` (`7wNVETTBVYpw`): chronological notes by year/month/day.
- `UTEC` (`FZOTpxgN0AkB`): main university knowledge base.
- `Notas ocultas` (`_hidden`): system/internal Trilium area. Avoid using it unless the user explicitly asks for hidden/system notes.

UTEC structure:

- `UTEC / 2026 / Ciclo 02` (`Bqh6hLOxUSGD`): current academic cycle.
- `Ciclo 02 / Materias` (`DHTFD2oH5ahA`): one note per course. Existing course notes include `MAT1-V Matematica I`, `ORTI-V Ingenieria y productividad`, and `STCB-V Seminario Taller de Competencias`.
- `Ciclo 02 / Entregables` (`BK0i2RY3LlxJ`): assignments, activities, tasks, due dates, and delivery tracking. Contains `Plantilla de entregable`.
- `Ciclo 02 / Evaluaciones` (`1Dyv28ZWLuIs`): exams, quizzes, partials, topics, dates, results, and preparation. Contains `Plantilla de evaluacion`.
- `Ciclo 02 / Recursos del ciclo` (`BUhAzun9JxlW`): links, documents, schedule-independent cycle resources, and useful data for the current cycle.
- `Ciclo 02 / Capturas rapidas` (`pfjZzdO8MX9w`): quick temporary notes grouped by date.
- `Ciclo 02 / Como usar este ciclo` (`jKZ7kIj0jL4C`): explains the organization rule for this cycle.
- `UTEC / Conocimiento general` (`z5lKXYQ8w2Y3`): information that does not depend on a year or cycle.
- `Conocimiento general / Contactos institucionales` (`ziCIQ8g27aKK`): institutional contacts.
- `Conocimiento general / Soporte tecnico` (`d82IZ1R960y0`): technical support contacts and platform support.
- `Conocimiento general / Enlaces utiles` (`m9rsgF6ddEq3`): portal, Blackboard, Teams, Outlook, and other links.
- `Conocimiento general / Modalidad no presencial` (`vxtH4wzGyltQ`): notes about asynchronous/virtual study modality.
- `Conocimiento general / Mi informacion UTEC` (`UJyGqsYclLVW`): personal institutional data. Treat as sensitive and only surface what is needed for the user's request.

## Placement Rules

Choose the parent note by meaning:

- Course content, class notes, topics, materials, or professor/course details go under `Ciclo 02 / Materias`, preferably inside the specific course note if it exists.
- Assignments, activities, homework, due dates, submission links, and pending work go under `Ciclo 02 / Entregables`.
- Exams, partials, quizzes, study topics, grades, or exam preparation go under `Ciclo 02 / Evaluaciones`.
- General cycle-specific links, files, procedures, schedules, or data go under `Ciclo 02 / Recursos del ciclo`.
- Fast unclassified notes go under `Ciclo 02 / Capturas rapidas`, ideally inside a dated child note if the date is known.
- University-wide contacts, procedures, platform links, and modality information go under `UTEC / Conocimiento general` or its specific child sections.
- Truly date-based journaling goes under `Calendar`.

If the user asks to capture something quickly and does not specify a destination, use `Ciclo 02 / Capturas rapidas` for UTEC-related material and include enough title context to make it easy to move later.

## Search Workflow

When the user asks to find or summarize information:

1. Search broadly with `trilium_search_notes` using the user's terms and obvious Spanish synonyms.
2. If results are sparse, inspect the relevant subtree and search again by likely section titles.
3. Read the most relevant note contents before answering.
4. Cite note titles and, when useful, note IDs so the user can locate the source.
5. Distinguish between information found in Trilium and your own inference.

For complex search syntax, load `search_syntax` before composing the query.

## Create Or Update Workflow

When creating a note:

1. Identify the correct parent using the placement rules.
2. Check whether an equivalent note already exists under that parent or nearby.
3. If no duplicate exists, create the note with `trilium_create_note`.
4. Use `type: text` for normal notes and Markdown content.
5. Use `type: book` for containers that should hold child notes.
6. Use `type: code`, `mermaid`, `webView`, `search`, `render`, or dashboard-related note types only when the user asks for that kind of artifact.

When updating a `text` note, prefer `trilium_set_note_content` after reading the current content, because rich text notes need whole-content replacement. Use targeted edit tools only for compatible non-rich-text note types where exact replacement is safe.

When adding tasks or evaluaciones, follow the existing templates when possible:

```markdown
*   Materia:
*   Tipo:
*   Fecha limite:
*   Estado:
*   Plataforma:
*   Enlace:

## Instrucciones

## Archivos

## Notas
```

```markdown
*   Materia:
*   Tipo:
*   Fecha:
*   Estado:
*   Plataforma:
*   Enlace:

## Temario

## Indicaciones

## Resultado

## Notas
```

## Organization Workflow

When the user asks to reorganize notes:

1. Inspect the current subtree and relevant note metadata first.
2. Explain the intended moves briefly before making multi-note structural changes.
3. Use `trilium_move_note` when the note should live in a new single location.
4. Use `trilium_clone_note` when the same note should appear in two logical places without duplicating content.
5. Use labels and relations for cross-cutting metadata when hierarchy alone is not enough.

Do not delete notes or remove attributes without explicit confirmation.

## Response Style

Answer in Spanish by default. Be concise and practical:

- For search requests: summarize findings and include sources.
- For create/update requests: state what was created or changed and where.
- For ambiguous requests: ask one direct question with likely options.
- For sensitive personal information: avoid repeating unnecessary private details.

## Examples

Input: `/trilium busca el correo de soporte tecnico`

Action: Search/read `Conocimiento general / Soporte tecnico`, then answer with the relevant contacts and note titles.

Input: `/trilium crea una nota para la tarea de Matematica I que vence el viernes`

Action: Use `Ciclo 02 / Entregables` as parent, follow the entregable template, and include `MAT1-V Matematica I` in the content.

Input: `/trilium guarda esto como recurso del ciclo: enlace a la guia de Blackboard ...`

Action: Create or update a note under `Ciclo 02 / Recursos del ciclo`.

Input: `/trilium organiza esta captura dentro de la materia ORTI-V`

Action: Search for the capture and course note, then move or clone according to the user's intent.
