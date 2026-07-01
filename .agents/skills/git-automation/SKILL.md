---
name: git-automation
description: Provides safe, autonomous Git operations for coding agents. Use when the user asks to commit changes, resolve merge conflicts, amend commits, or perform any Git workflow that involves staging, committing, merging, or rewriting history. Also trigger when the user says "commit this", "merge this branch", "fix the conflicts", "update the last commit", "amend my commit", "push with force", or describes a Git problem that needs resolution. Do not use for simple read-only queries like "show me git log" or "what branch am I on" — those are trivial. Use when the operation modifies the repository state.
---

# Git Automation

Provides safe, autonomous Git operations for agents. Every operation begins with repository state validation and follows a fail-safe protocol — work is never lost, history is never rewritten without explicit justification, and the user is always informed before disruptive actions.

## Why safety-first

Autonomous Git operations are dangerous. A single mistaken force push or blind conflict resolution can destroy hours of work. This skill enforces a validation-before-action pattern: check the repository state, detect any ongoing operations that could interfere, and only proceed when it's safe. When in doubt, it stops and asks.

## Pre-Operation Validation

Before executing any operation, run these checks:

1. **Verify repository exists**: `git rev-parse --git-dir` — abort if not in a git repo.
2. **Check for detached HEAD**: `git symbolic-ref -q HEAD` returns non-zero → detached. Flag it to the user before proceeding.
3. **Detect ongoing operations**: Check for `.git/MERGE_HEAD` (merge in progress), `.git/rebase-merge` or `.git/rebase-apply` (rebase in progress), `.git/CHERRY_PICK_HEAD` (cherry-pick in progress), `.git/BISECT_START` (bisect in progress). If any exist, report the situation and do not proceed unless the operation is specifically meant to resolve it.
4. **Check for uncommitted changes**: `git status --porcelain` — understand what's modified, staged, untracked, or deleted before acting.

Print a concise summary of the repository state before acting: current branch, ongoing operations if any, and a categorized list of pending changes.

---

## Operation 1: Commit Pending Changes

Triggered when the user wants to save work with a commit.

### Step 1: Inventory changes

Run `git status --porcelain` and categorize every entry:

| Status code | Meaning | Action |
|---|---|---|
| `M ` | Modified in index (staged) | Include in commit |
| ` M` | Modified in working tree (unstaged) | Stage then include |
| `A ` | Added to index | Include in commit |
| `??` | Untracked | Decide per file (see below) |
| `D ` | Deleted in index | Include in commit |
| ` D` | Deleted in working tree | Stage then include |
| `R ` | Renamed in index | Include in commit |
| `UU` | Unmerged (conflict) | Do NOT commit — abort and suggest conflict resolution |

**Untracked files**: Present them to the user grouped by type (source code, config, generated, binary). Only stage source files and config files by default. Skip `node_modules/`, `.env`, build artifacts, and binaries unless the user explicitly includes them.

**Large files**: Warn if any file exceeds 1MB before staging.

### Step 2: Analyze changes for commit type

Read the diff of staged changes (`git diff --cached` or `git diff --staged`) and determine the Conventional Commits type:

| Type | When to use |
|---|---|
| `feat` | New feature or functionality added |
| `fix` | Bug fix |
| `docs` | Documentation only changes |
| `style` | Formatting, whitespace, linting (no logic change) |
| `refactor` | Code restructuring without feature/fix |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `build` | Build system, dependencies, tooling config |
| `ci` | CI/CD pipeline changes |
| `chore` | Maintenance, cleanup, non-functional changes |
| `revert` | Reverting a previous commit |

If changes span multiple categories, choose the one that represents the primary intent. If truly mixed (e.g., a feature + its tests + docs), use the most significant type and note the others in the body.

### Step 3: Determine scope

Derive the scope from the files changed:
- Files in `src/auth/` → scope `auth`
- Files in `src/components/` → scope `ui` or `components`
- Files in `docs/` → no scope (just `docs:`)
- Changes across multiple modules → broader scope or omit scope
- Root-level config files → scope `config` or omit

The scope should be a short noun (1-2 words, lowercase, no spaces). When in doubt, omit the scope rather than forcing one.

### Step 4: Generate commit message

Format:
```
type(scope): imperative description

Optional body explaining what and why (not how).

Optional footer (BREAKING CHANGE, issue references).
```

**Description rules:**
- Use imperative mood: "add" not "added", "fix" not "fixed"
- 50-72 characters maximum for the summary line
- Lowercase first word after the colon
- No period at the end of the summary
- Be specific: "fix race condition in payment processor" not "fix bug"

**Body rules (when needed):**
- Explain what changed and why (the diff already shows how)
- Wrap at 72 characters
- Use bullet points for multiple changes

### Step 5: Stage and commit

```bash
git add <files>
git commit -m "type(scope): description" -m "body line 1" -m "body line 2"
```

Verify the commit was created: `git log -1 --oneline`.

**Do not commit if:**
- No changes are staged (abort with a clear message)
- Merge conflicts exist (abort, suggest conflict resolution)
- The repo is in a detached HEAD state without the user's knowledge

---

## Operation 2: Resolve Merge Conflicts

Triggered when a merge results in conflicts that need resolution.

### Step 1: Confirm we're in a merge state

Run `git status --porcelain`. Files marked `UU` are unmerged (both sides modified). The presence of `.git/MERGE_HEAD` confirms we're mid-merge.

Identify:
- **Current branch**: `git rev-parse --abbrev-ref HEAD`
- **Incoming branch**: Read `.git/MERGE_HEAD` and resolve: `git name-rev <hash>`
- **Merge base**: `git merge-base HEAD MERGE_HEAD`

### Step 2: For each conflicting file, understand intent

For every file with conflicts:

1. **Read the file** with conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) to see both versions.
2. **Determine what changed on each side**:
   ```bash
   git diff <merge-base>..HEAD -- <file>    # our changes
   git diff <merge-base>..MERGE_HEAD -- <file>  # their changes
   ```
3. **Understand the intent**: not just the textual diff, but the purpose behind each change. Did we refactor while they added a feature in the same area? Did both sides fix the same bug differently?

### Step 3: Produce the merged result

For each conflict, decide on a resolution strategy:

| Situation | Resolution |
|---|---|
| Both sides added different code in different locations | Keep both |
| Both sides modified the same lines differently | Analyze intent, merge semantically |
| One side deleted code the other modified | Check if the modification is still needed |
| Our side has newer/better implementation | Keep ours, note theirs in comment if useful |
| Their side has newer/better implementation | Take theirs |
| Unclear which is correct | Stop and ask the user |

**When merging, the goal is a correct program, not just text without markers.** After resolving all conflicts in a file:
- Read the entire file from top to bottom
- Verify imports are correct and all referenced symbols exist
- Check for duplicate code, dead code, or orphaned logic from partial merges
- Run any available linter or type checker on the file

### Step 4: Validate the merge

After resolving all files:
1. **Build check**: If the project has a build command, run it.
2. **Test check**: If tests exist, run them (`npm test`, `pytest`, `cargo test`, etc.).
3. **Lint check**: Run the linter if available.

If any validation fails, fix the issues before staging. Do not stage unresolved files.

### Step 5: Complete the merge

```bash
git add <resolved-files>
git commit -m "merge: resolve conflicts between <branch-a> and <branch-b>"
```

If conflicts still exist (files with unresolved markers), abort with a clear message listing the remaining files.

### When to stop and ask

Stop execution and present the situation to the user when:
- The same lines were modified for completely different reasons with no clear winner
- One side introduces a new dependency or architectural change that conflicts with the other
- The merged code passes syntax checks but the semantic correctness is uncertain
- A file has more than 5 conflict regions (high risk of incorrect merge)
- Tests fail after resolution and the fix isn't obvious

Present: the conflicting file, both versions' intent, the proposed resolution, risks, and ask for confirmation before proceeding.

---

## Operation 3: Update Latest Commit

Triggered when the user wants to add changes to the most recent commit (amend or force-push).

### Step 1: Check push status

```bash
git log -1 --format="%H"  # Get HEAD hash
git branch -r --contains HEAD  # Check if on any remote
```

Alternatively: `git status` will say "Your branch is ahead of 'origin/main' by N commits" if unpushed.

Determine:
- Is the latest commit pushed to any remote? (check all remotes, not just origin)
- Is this branch shared (used by other developers)?

### Step 2: If NOT pushed — amend

```bash
git add <new-changes>
git commit --amend -m "type(scope): updated description" -m "updated body"
```

Rules for amending:
- Preserve the intent of the original commit unless the user explicitly wants to change it
- Update the commit message to reflect the combined changes — re-analyze the full diff
- Ensure the amended message still follows Conventional Commits
- If the original commit was a merge commit, warn the user that amending a merge is complex and may lose history

### Step 3: If pushed — inform and use safe force

If the commit has been pushed, present the situation clearly:

```
The latest commit has already been pushed to origin/main.
Amending it will rewrite published history.

Options:
1. Create a new commit instead (safe, recommended)
2. Amend and force-push with --force-with-lease (rewrites history, may affect collaborators)
```

If the user chooses option 2:
```bash
git add <new-changes>
git commit --amend -m "type(scope): updated description"
git push --force-with-lease
```

**Never use `--force`** — only `--force-with-lease`, which refuses to push if the remote has commits you haven't seen.

### Step 4: Verify

After the operation:
```bash
git log -1 --oneline  # Confirm the updated commit
git status            # Confirm clean state
```

If force-pushed, also verify: `git log --oneline -3` to confirm the remote history is correct.

---

## Edge Cases and Safety Barriers

### Operations that require user confirmation

These actions stop and ask before executing:
- Force-push to any branch
- Amending a merge commit
- Committing during an active rebase
- Deleting or resetting commits (not covered by this skill — but if detected, stop)
- Committing files that match `.gitignore` patterns (accidental `git add -f`)
- Operations on `main`/`master` branch (commits allowed, but force-push requires extra confirmation)

### Recovery from errors

- **Commit fails** (hook rejection): Read the hook output, explain what failed, suggest fixes. Do not bypass hooks with `--no-verify` unless the user explicitly requests it.
- **Merge conflict during resolution**: Never leave the repo in a conflicted state. Either resolve everything or abort the merge cleanly with `git merge --abort`.
- **Push rejected**: Read the rejection reason. If it's a non-fast-forward, explain that the remote has diverged and suggest `git pull --rebase` before retrying.
- **Detached HEAD**: Warn the user. If they confirm they want to proceed, suggest creating a branch first to avoid losing commits.

### What this skill does NOT do

- Interactive rebase (`git rebase -i`)
- Hard reset (`git reset --hard`)
- Squash or fixup operations without explicit user request
- Branch deletion
- `git clean` (removing untracked files)
- Pushing to protected branches without confirmation
- Any operation that permanently deletes commits
