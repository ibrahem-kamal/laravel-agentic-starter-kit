---
name: checkpoint
description: >
  Save current work-in-progress as a git checkpoint commit. Use when the user says "checkpoint",
  "save progress", "commit what we have", or "/checkpoint". Stages all current changes, drafts
  a descriptive message from the diff, and commits — does not push. Safe to use mid-feature
  before risky refactors or before context switches.
---

# Checkpoint

Save the current working tree as a git commit with a descriptive message generated from the diff. Use checkpoints liberally — they're cheap, recoverable, and prevent loss of work-in-progress.

This is intentionally lighter than `ship-laravel`: no Pint, no Pest, no PR draft. Just a clean save point.

## Steps

### Step 1 — Confirm there's something to commit

```bash
git status --short
```

If clean, report "Nothing to checkpoint — working tree is clean." and stop.

### Step 2 — Refuse to checkpoint on protected branches

```bash
git rev-parse --abbrev-ref HEAD
```

If the branch is `main` or `master`, abort with: "Refusing to checkpoint directly to {branch}. Create a feature branch first."

### Step 3 — Inspect the diff

```bash
git diff --stat
git diff --cached --stat
```

Read the diff (use `git diff` and `git diff --cached`) to understand what changed. Look at file paths and a sample of the changes — not every line.

### Step 4 — Detect commit prefix convention

```bash
git log --oneline -10
```

Mirror the project's commit prefix style (`feat(PC-XXX):`, `feat:`, etc.). For checkpoints, use the prefix `wip:` or `checkpoint:` to make them obvious in history.

If the project uses ticket prefixes (e.g. `feat(PC-042):`), use `wip(PC-042): checkpoint — {summary}`.

### Step 5 — Compose message

One-line summary describing what's been worked on. Look at modified file paths to infer the area:

- Only migrations changed → "schema work — {table-name}"
- Livewire components changed → "{component-name} UI"
- Service classes + tests → "{service-name} logic + tests"
- Mixed → name the largest area + "and other changes"

Format:

```
wip{(TICKET)}: checkpoint — {one-line summary}

{2–4 bullets of what's in progress and what's next, derived from the diff}
```

### Step 6 — Stage and commit

```bash
git add -A
git commit -m "$(cat <<'EOF'
{drafted message}
EOF
)"
```

Use `-A` for checkpoints — the whole point is to capture everything. If the user has files they explicitly don't want committed (e.g. `.env.local` they forgot to gitignore), they should tell you up front.

### Step 7 — Report

```
Checkpoint saved: {short hash}

  {commit subject}

Working tree is clean. Continue working — `git reset --soft HEAD~1` to undo.
```

## Critical rules

- Do not push. Checkpoints stay local until the user explicitly ships.
- Do not skip hooks (`--no-verify`). If a pre-commit hook fails, surface the error and let the user decide.
- Do not amend an existing commit. Checkpoints are always new commits.
- Do not run on the default branch.
