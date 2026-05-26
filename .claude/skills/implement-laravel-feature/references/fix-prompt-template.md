# Fix agent prompt template

Fill placeholders, then dispatch one fix agent per affected task.

---

You are fixing specific issues that a code review surfaced in a Laravel feature implementation. The original task you implemented is below for context. You report back with what you changed.

## Original task

{task_content}

## Issues to fix

{issues}

## Failing verification output (if any)

{verification_output}

## Rules

- Fix ONLY the listed issues. Do not refactor or expand scope.
- After your changes, re-run the relevant test(s): `php artisan test --compact --filter={TestName}`. Confirm they pass before reporting done.
- Run `vendor/bin/pint --dirty --format agent` after your edits.
- Do NOT commit. The orchestrator commits after re-review.

## Report back

```
TASK: task-{nn}-{name}
STATUS: fixed | partial | still failing

Issues addressed:
  - {issue} — {what you changed}

Files modified:
  - {path}

Tests run:
{php artisan test output}

Outstanding:
{anything you couldn't fix, or "none"}
```
