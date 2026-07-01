---
name: plan-executor
description: Executes a PLAN.md checklist continuously and autonomously until the project is fully implemented, validated, and production-ready. Use when the user wants to build a project, implement a plan, execute a blueprint, follow a development checklist, or turn a technical specification into working software. Also trigger when the user says "execute the plan", "build the project", "implement this", "start working on", "run through the checklist", "follow PLAN.md", or describes wanting to build something and a PLAN.md already exists in the workspace. If no PLAN.md exists, suggest creating one with the plan-generator skill first. Do not use for simple, single-file changes or quick bug fixes — this is for structured, multi-step project execution driven by a formal checklist.
---

# Plan Executor

This skill drives autonomous, continuous execution of a PLAN.md checklist. It treats the plan as the single source of truth and works through it item by item until the project is done.

## Why this approach

Autonomous execution fails in predictable ways: the agent gets distracted by sub-problems, leaves work half-done, assumes things work without verification, or drifts from the original specification. A checklist-based execution protocol prevents all of these. Every task is explicit. Every completion is verified. The loop is mechanical enough that the agent can't skip steps, but judgment is still applied in *how* each checklist item is implemented.

## Precondition: PLAN.md must exist

Before beginning any work, check whether a file named `PLAN.md` exists in the **workspace root directory** (the top-level of the current working project).

If PLAN.md does NOT exist:
- Tell the user: "No PLAN.md found in the workspace root. I recommend running the **plan-generator** skill first to create one. Would you like me to load that skill and generate a comprehensive implementation plan?"
- Do not proceed with execution until a PLAN.md is available.

If PLAN.md exists, proceed to the execution loop.

## The Execution Loop

This is the core protocol. Repeat it until every checklist item in PLAN.md is marked `- [x]`.

### Step 1: Read PLAN.md

Before starting any task — even if you just read it moments ago — re-read PLAN.md. Plans may have been updated by a previous iteration of the loop. 

While reading, identify:
- The next unchecked item (`- [ ]`) 
- Any dependencies that item has on prior unchecked items (items earlier in the list that are not yet `[x]`)
- Any context from surrounding items that informs how this item should be executed

If all items are `[x]`, the project is complete. Perform a final verification pass (see "Project Completion" below).

### Step 2: Select the next task

Pick the **first** unchecked item in the checklist. If earlier unchecked items exist, they take priority — work through the list in order unless a dependency chain makes a later item blocking.

If the selected item has subtasks (indented `- [ ]` items beneath it), select the first unchecked subtask instead. Complete subtasks before completing their parent.

### Step 3: Execute the task completely

Execute the selected checklist item in full. "In full" means:

- All code, configuration, documentation, and tests that the item describes must be produced
- No stubs, placeholders, TODOs, FIXMEs, mock implementations, or "implement later" notes are acceptable — except where the checklist item itself calls for a stub as an intermediate step
- If the item is ambiguous, interpret it in the way that best serves the overall project objective
- If the item requires a decision (e.g., choosing between two equivalent libraries), make the decision and document it inline

Use all available tools to execute: file operations, terminal commands, MCP tools, skills, web searches, and any other capabilities at your disposal.

### Step 4: Verify the work

Before marking the item as done, verify that it actually works. The verification method depends on the item:

| Item type | Verification approach |
|---|---|
| Code implementation | Run the code, check output, run related tests |
| Configuration | Validate syntax, test that config is loaded correctly |
| Test suite | Run the tests, confirm all pass |
| Infrastructure | Check that resources exist and are configured correctly |
| Documentation | Read it back, verify it matches the implementation |
| Integration | End-to-end test across components |
| Security | Run relevant scanners, check config manually |

If verification fails, fix the issue and re-verify. Do not mark an item as done with known failures.

### Step 5: Mark as complete

Edit PLAN.md and change the corresponding `- [ ]` to `- [x]`. Save the file immediately.

If the item had subtasks and they are all now `[x]`, also mark the parent as `[x]`.

### Step 6: Check for newly discovered work

During execution, you may discover that:
- A dependency was not anticipated in the plan
- An edge case requires additional handling
- A configuration step was missing
- A test revealed a gap that needs filling

When this happens, add new `- [ ]` items to PLAN.md **before** continuing. Place them in the appropriate section, near related items. Only add items that are **strictly necessary** to complete the original project objective. Do not add:

- Optional enhancements
- "Nice to have" features
- Future backlog items
- Speculative optimizations
- Tasks that would be nice but aren't required

After adding, return to Step 1 and continue the loop. The new items become part of the execution stream.

### Step 7: Repeat

Go back to Step 1. Continue until PLAN.md contains zero `- [ ]` items.

## Project Completion

When all items are `[x]`, perform a final verification:

1. **Re-read the original project requirements** (at the top of PLAN.md or in the initial user request) and confirm every requirement has been addressed.
2. **Run the full test suite** one final time — every test must pass.
3. **Verify the project builds and runs** from a clean state (e.g., `npm install && npm test && npm start`).
4. **Check for any unresolved issues** — warnings, skipped tests, missing error handlers, hardcoded values, missing environment variables, undocumented endpoints.
5. **Read through all changed/new files** looking for leftover TODOs, stubs, or incomplete logic.

If the final verification reveals any gaps, add them as new checklist items, execute them, and re-verify.

When everything passes, tell the user: "Project implementation is complete. All PLAN.md items are verified and marked done."

## Handling Errors and Blockers

If a task cannot be completed due to an external blocker (missing API key, unavailable service, permission issue):

1. Document the blocker clearly in the PLAN.md item description
2. Add a follow-up task to resolve the blocker
3. Skip to the next independent item
4. Return to the blocked item once the blocker is resolved

Do not stop execution entirely because of a single blocked item — work on other items while waiting.

If you encounter a fundamental problem (the plan is wrong, the architecture won't work, a key dependency doesn't exist), flag it to the user clearly and explain the options. Do not silently work around architectural flaws.
