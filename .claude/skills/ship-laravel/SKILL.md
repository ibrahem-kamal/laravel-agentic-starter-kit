---
name: ship-laravel
description: >
  Pre-push checklist for a Laravel feature branch. Runs Pint, Pest, npm build (if frontend
  changed), graphify update, then drafts a Conventional-Commit PR title + body. Use when the
  user says "ship it", "open a PR", "create the PR", "I'm done — push it", or "/ship-laravel".
  Does NOT push or open the PR automatically — surfaces the exact commands for the user to run.
---

# Ship Laravel

Run the pre-push gauntlet for a Laravel feature branch and draft the PR. The skill is a checklist runner — it executes the verifications, surfaces results, then composes the PR title and body for the user to paste. It does **not** push to remote or call `gh pr create` itself; that's the user's decision.

## When to use

- Feature implementation is complete and tests pass locally
- User wants to open a PR but hasn't drafted the body
- User says "ship it" / "open a PR" / "ready to push"

## Steps

### Step 1 — Confirm branch state

```bash
git status --short
git rev-parse --abbrev-ref HEAD
git log --oneline @{upstream}..HEAD 2>/dev/null || git log --oneline -20
```

If there are uncommitted changes, ask the user whether to commit them now (suggest `/checkpoint`) or stash. Do not proceed with dirty tree.

If the current branch is `main` / `master`, abort with: "You're on the default branch. Create a feature branch first."

### Step 2 — Pint

```bash
vendor/bin/pint --dirty --format agent
```

If Pint reports changes, the working tree is now dirty. Stage and commit them as a separate `style: pint` commit before proceeding:

```bash
git add -A
git commit -m "style: pint"
```

### Step 3 — Pest

```bash
php artisan test --compact
```

Capture pass count and any failures. **Halt and report if any test fails.** Do not proceed to PR draft with a red suite.

### Step 4 — Frontend build (conditional)

Run only if the diff touched frontend files. Detect via:

```bash
git diff --name-only $(git merge-base HEAD origin/main 2>/dev/null || echo HEAD~1)...HEAD \
  | grep -E '\.(blade\.php|vue|js|ts|css)$|^resources/' | head -1
```

If matched:

```bash
npm run build
```

Report success or failure. If failure, halt.

### Step 5 — graphify update

If `graphify-out/` exists in the project root:

```bash
graphify update .
```

If `graphify` is not on PATH, skip silently — graphify update is best-effort, not blocking.

### Step 6 — Detect commit convention

```bash
git log --oneline -15
```

Look for the prefix style used recently. Common patterns:
- `feat(PC-XXX): ...` (ticket-prefixed Conventional Commits — see PreConsult AI)
- `feat: ...` (plain Conventional Commits)
- `Add ...` (sentence-case freeform)

Use whatever style the project's recent history uses for the PR title.

### Step 7 — Determine base branch

```bash
gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null || echo main
```

### Step 8 — Draft PR title and body

Compose from the branch's commits (`git log {base}...HEAD`) and the diff stat (`git diff --stat {base}...HEAD`).

**Title format:** `{prefix}: {short imperative summary}` — under 70 chars. If the branch name encodes a ticket (e.g. `feature/pc-042-billing`), prefix with the ticket.

**Body template:**

```markdown
## Summary

{1–3 bullets describing what this PR does and why}

## Changes

- {bulleted list grouped by area: backend / frontend / tests / config}

## Tests

- {N} Pest tests, all passing locally
- Pint clean
- {if frontend touched} `npm run build` succeeds

## Verification

- [ ] CI green
- [ ] Manual smoke test on review env
- [ ] Reviewer signoff

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Step 9 — Surface the commands, don't run them

Print the exact commands the user should run to push and open the PR. Do **not** execute these — pushing and opening PRs is a deliberate human action.

```
Ready to ship. Run these:

  git push -u origin {branch}

  gh pr create \
    --base {base} \
    --title "{drafted title}" \
    --body "$(cat <<'EOF'
{drafted body}
EOF
)"
```

### Step 10 — Optional: offer post-PR checks

If the project has a `babysit-prs` workflow, post-merge linting, or specific CI signals to watch, suggest the user run `/loop` to monitor.

## Critical rules

- **Halt on any verification failure.** Don't draft a PR for broken code.
- **Never push or open PRs autonomously.** Always surface the commands for the user.
- **Match the project's existing commit/PR conventions.** Detect from `git log`, don't impose a template that doesn't fit.
- **Don't include AI-attribution lines unless the project already does in recent PRs.** Check `gh pr list --state merged --limit 5` to confirm.
