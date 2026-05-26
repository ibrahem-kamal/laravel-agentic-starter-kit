# {Feature Name}

{One-paragraph summary of the feature: what it does, why it matters, who asked for it.}

## Status

`pending` | `in_progress` | `complete`

## Dependency Graph

```
Wave 1: [task-01] [task-02] [task-03]
            ↓        ↓        ↓
Wave 2: [task-04 — depends on 01,02]  [task-05 — depends on 03]
            ↓
Wave 3: [task-06 — depends on 04,05]
```

## Wave Table

| Wave | Tasks | Can run in parallel | Description |
|------|-------|---------------------|-------------|
| 1    | task-01, task-02, task-03 | Yes | {what this wave establishes — e.g. "schema + base models"} |
| 2    | task-04, task-05 | Yes | {e.g. "business logic and policies"} |
| 3    | task-06 | n/a (single task) | {e.g. "UI wiring and final integration"} |

## Task Status

Wave 1:
- [ ] task-01-{name}
- [ ] task-02-{name}
- [ ] task-03-{name}

Wave 2:
- [ ] task-04-{name}
- [ ] task-05-{name}

Wave 3:
- [ ] task-06-{name}

## Files

- [requirements.md](./requirements.md) — feature context shared by all coder agents
- [action-required.md](./action-required.md) — manual steps for the human
- [tasks/](./tasks/) — self-contained task files
