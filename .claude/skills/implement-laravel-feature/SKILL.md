---
name: implement-laravel-feature
description: >
  Orchestrate parallel implementation of a Laravel feature specification by dispatching coder
  sub-agents wave-by-wave with Pint + Pest review gates between waves. Use when the user says
  "implement this feature", "start implementing", "run the spec", "execute the plan",
  "continue implementing", or "/implement-laravel-feature". Reads a spec folder under
  specs/{feature}/ produced by create-laravel-spec. This skill does NOT write code itself —
  it orchestrates coder sub-agents.
---

# Implement Laravel Feature

Orchestrate the parallel implementation of a Laravel feature specification by dispatching coder agents wave-by-wave. This skill reads a spec folder created by `create-laravel-spec`, identifies the next wave of parallelisable work, spawns coder agents for each task, runs Pint + Pest verification, and gates progress with code review.

The orchestrator **never writes code itself**. Its job is to:

1. Parse the spec and determine what to do next
2. Give each coder agent exactly the context it needs (one task per agent)
3. Verify via lint + tests + code review
4. Manage the fix loop if review fails
5. Track progress in the spec files and commit completed waves

## Prerequisites

A `specs/{feature}/` directory containing:
- `README.md` with wave assignments and task status checkboxes
- `requirements.md` with feature context
- `tasks/task-{nn}-*.md` files (one per task, self-contained)

If absent, suggest the user create one with `/create-laravel-spec` first.

## Orchestration

### Step 1 — Load the spec

1. Read `specs/{feature}/README.md`
2. Read `specs/{feature}/requirements.md`
3. Parse the **Task Status** section's checkboxes:
   - `- [x]` = completed (skip)
   - `- [ ]` = pending (include)
4. Determine **current wave**: the first wave with any incomplete tasks
5. If all tasks across all waves are complete, report "All tasks complete!" and stop

This makes the skill **resumable**: invoked on a partially completed spec, it picks up where it left off.

### Step 2 — Process each wave

For each wave starting from the current one, execute Steps 3–8 below, then advance.

### Step 3 — Prepare wave tasks

1. Read all incomplete task files for this wave
2. **Check for file overlaps**: scan "Files to Create" and "Files to Modify" across all tasks in this wave. If any file appears in more than one task, warn the user:

```
Warning: File overlap detected in Wave {N}:
  - {file-path} is modified by both task-{nn} and task-{mm}

Options:
  1. Proceed anyway (risk of conflicts)
  2. Run these tasks sequentially instead of in parallel
```

Wait for the user's decision.

### Step 4 — Dispatch coder agents

For each task in the wave, spawn an agent via the `Agent` tool with `subagent_type: "general-purpose"` (or a project-specific coder if available — check the agent list). Spawn all agents in a **single message** so they run in parallel.

Read `references/coder-prompt-template.md` and construct each agent's prompt filling in:

- `{requirements}`: full text of `requirements.md`
- `{completed_tasks_summary}`: for each previously completed task, a one-paragraph summary of what was implemented and what files were created/modified
- `{task_content}`: full text of the task file being assigned
- `{stack_profile}`: `livewire`, `vue`, or `backend-only` from `.claude/agentic-profile` or `requirements.md`
- `{relevant_skills}`: list of skills the coder should invoke based on task type:
  - Migration/Model → `laravel-best-practices`
  - Livewire UI → `livewire-development`, `fluxui-development`, `tailwindcss-development`
  - Vue/Inertia UI → `frontend-design`, `tailwindcss-development`
  - Auth-related → `fortify-development`
  - AI/LLM → `prism-ai` (bundled)
  - Tests → `pest-testing`
  - Queue/Job → `configuring-horizon` if Horizon is detected

The coder agents must NOT commit their changes — the orchestrator handles commits after review.

### Step 5 — Collect results

Wait for all agents in the wave to complete. Each agent reports:
- Files created
- Files modified
- A summary of what was implemented

If any agent fails, note the failure and continue with the remaining results. Report failures to the user after the wave.

### Step 6 — Verification gate

Run, in this order, capturing exit codes:

```bash
vendor/bin/pint --dirty --format agent
php artisan test --compact
```

If either fails:
- If Pint reports changes only (no errors): note that Pint reformatted files; that's fine, proceed
- If Pest fails: capture which test(s) failed and feed them into the fix loop (Step 7)
- If `php artisan test` errors before running (e.g. boot failure, missing class): treat as a hard fix-loop trigger

Then spawn a single review agent via `Agent` with `subagent_type: "pr-review-toolkit:code-reviewer"` (if available; otherwise `general-purpose` with a review-style prompt). Read `references/review-prompt-template.md` and fill in:

- `{wave_number}`
- `{requirements}`: full `requirements.md`
- `{task_summaries}`: per-task title + coder agent's completion summary
- `{verification_status}`: Pint and Pest output summary

The review agent verifies:
1. Each task's acceptance criteria
2. Files integrate (use statements resolve, FK references valid, route names referenced consistently)
3. No security issues (mass assignment, unauthorised access, SQL injection via raw queries)
4. Authorisation tests cover cross-tenant access where the feature touches multi-tenant models
5. PHP code quality (Laravel idioms, no N+1 obvious in eager-loaded relationships)

Returns verdict: **PASS** or **FAIL** with specific issues.

### Step 7 — Fix loop

If the review or verification returns **FAIL**:

1. Group issues by the task they relate to (match file paths against each task's "Files to Create/Modify")
2. Spawn one coder agent per affected task with a fix prompt (read `references/fix-prompt-template.md`):
   - `{issues}`: the specific issues
   - `{task_content}`: the original task file for context
   - `{verification_output}`: the failing Pint/Pest output if relevant
3. After fix agents complete, re-run Step 6

**Cap at 3 review cycles per wave.** If the third still fails, stop and report:

```
Wave {N} review failed after 3 cycles. Outstanding issues:
{list of remaining issues}

Options:
  1. Fix manually and re-run /implement-laravel-feature
  2. Proceed to the next wave anyway (risky)
  3. Stop here
```

### Step 8 — Complete the wave

After passing review (or user override):

1. **Update task files**: change Status from `pending` to `complete` in each task file
2. **Update README.md**: change `- [ ]` to `- [x]` for each completed task in the Task Status section
3. **Commit the wave**:

```bash
git add -A
git commit -m "feat({feature}): wave {N} — {brief summary}

{bulleted list of completed tasks}
"
```

Use the project's conventional commit prefix if one is established (`feat(pc-XXX):` style, etc.) — check recent `git log --oneline -10` to detect.

4. **Report wave completion**:

```
Wave {N} of {total} complete.

Tasks completed:
  - task-{nn}-{name}: {one-line summary}
  - task-{mm}-{name}: {one-line summary}

Verification:
  - Pint: clean
  - Pest: {X} tests passing
  - Code review: PASS

Commit: {short hash}

Next: Wave {N+1} has {count} tasks ready.
```

### Step 9 — Final integration review

After all waves are complete:

1. Run `vendor/bin/pint --dirty --format agent` and `php artisan test --compact` one final time
2. Run `npm run build` if the frontend was touched (Livewire views or Vue pages)
3. Spawn a final code-review agent over the full diff (`git diff main...HEAD` or against the branch's base)
4. Run `graphify update .` if `graphify-out/` exists in the project
5. Report final status:

```
Feature "{feature}" implementation complete.

Waves: {N}/{N}
Tasks: {T}

Verification:
  - Pint:   clean
  - Pest:   {X} passing
  - Build:  {ok | skipped — no frontend changes}
  - graphify update: {ok | skipped}
  - Integration review: PASS

Next steps:
  - Review the diff: git diff {base-branch}...HEAD
  - Run /ship-laravel to open a PR when ready
```

## Error handling

- **Coder agent failure**: mark the task as failed, report, continue with the rest. The review step will catch the gap.
- **Pest failure that persists across 3 fix cycles**: stop and ask the user — do not commit broken tests.
- **Missing spec folder**: ask the user to provide the feature name or suggest `/create-laravel-spec`.
- **All tasks already complete**: report and stop.

## Key principles

- **The orchestrator does not write code.** Its job is dispatch, verification, review, progress tracking.
- **Each coder agent gets exactly one task.** This keeps each agent's context focused and manageable.
- **Completed task summaries are brief.** One paragraph per task in subsequent waves' prompts — not full file contents — so the prompt size stays bounded.
- **Pint + Pest are non-negotiable.** Every wave runs both before review. This catches breakage early.
- **Progress lives in the spec files.** README checkboxes and task Status fields are the source of truth. Resumable across sessions.
