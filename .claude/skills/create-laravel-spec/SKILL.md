---
name: create-laravel-spec
description: >
  Create a structured Laravel feature specification with self-contained task files organised
  into parallel execution waves. Use when the user says "create a spec", "plan this feature",
  "write up an implementation plan", "break this into tasks", or "/create-laravel-spec".
  Produces local spec files under specs/{feature}/ tailored to Laravel: migrations, models,
  Livewire or Vue/Inertia components, Pest tests, and authorisation. No GitHub integration.
---

# Create Laravel Feature Specification

Transform a planning conversation into a structured spec folder that enables parallel agent implementation in a Laravel project. The spec breaks a feature into self-contained Laravel-specific task files — each one detailed enough that a coder agent can implement it cold, without reading anything else.

## When to use

- After a planning conversation where requirements and technical decisions have been discussed
- When the user asks to create a spec, plan a feature, or break work into tasks
- When preparing work for parallel agent implementation via `implement-laravel-feature`

## Instructions

### Step 1 — Gather requirements

If the conversation already contains planning context, extract all of it. Review the entire conversation — the spec is the single source of truth; anything not captured here is lost.

If no planning conversation exists, interview the user:
- What does this feature do and why?
- What are the acceptance criteria?
- What technical constraints or decisions have been made?
- What models, routes, components, or external services are involved?

### Step 2 — Detect stack profile

Read `.claude/agentic-profile` if present (`livewire`, `vue`, or `none`). Otherwise sniff `composer.json`:
- `livewire/livewire` → Livewire profile
- `inertiajs/inertia-laravel` → Vue/Inertia profile
- Neither → ask the user before generating UI tasks

This determines which UI task template you reach for.

### Step 3 — Name the feature

Choose a kebab-case name (e.g. `add-user-billing`, `migrate-to-uuid-ids`, `clinic-dashboard-redesign`). This becomes the folder under `specs/`.

### Step 4 — Decompose into tasks

Break the implementation into atomic tasks. Each task should:
- Be completable in a single coding session by one agent
- Have a clear, specific scope — one concern per task
- Produce working, testable code when complete
- Not overlap in files modified with other tasks in the same wave

Typical Laravel task shapes (mix and match — one per task):

- **Migration + model + factory**: create a new table with its Eloquent model and factory
- **Form Request + Controller action**: API endpoint or HTTP action
- **Livewire component** (Livewire profile): one component, its view, and Pest test
- **Vue page + Inertia route** (Vue profile): one page component, the route binding, and Pest test
- **Job + queue config**: a queued Job class with its dispatch site
- **Policy / Gate**: authorisation rule for a model
- **Console command**: an artisan command class
- **Service class**: a single business-logic class

Granularity check: a good task touches 1–5 files. If a task touches more than 7 files, split it.

**Do not create test-only tasks** unless the user explicitly asks. Tests live inside the task that produces the code they cover.

### Step 5 — Build the dependency graph

For each task identify:
- **Depends on**: tasks that must complete first
- **Blocks**: tasks that need this one done

Tasks with no dependencies form Wave 1. Tasks whose dependencies are all in Wave 1 form Wave 2, etc. Within a wave, tasks run in parallel.

**Critical rule**: tasks in the same wave must not modify overlapping files. If two tasks would touch the same file (e.g. both add a route to `routes/web.php`), move one to a later wave. Common collision points to watch:

- `routes/web.php`, `routes/api.php`
- `app/Providers/AppServiceProvider.php`
- `config/*.php` files
- `resources/css/app.css`
- `bootstrap/app.php` (Laravel 12+ middleware registration)
- `.env.example`

### Step 6 — Create the spec folder

Create this structure at `specs/{feature-name}/`:

```
specs/{feature-name}/
├── README.md
├── requirements.md
├── action-required.md
└── tasks/
    ├── task-01-{name}.md
    ├── task-02-{name}.md
    └── ...
```

Read the templates in `references/` before writing each file:
- `references/readme-template.md` — wave table, dependency graph, status checkboxes
- `references/requirements-template.md` — feature context for coder agents
- `references/action-required-template.md` — manual human steps (API keys, env vars, third-party setup)
- `references/task-template.md` — generic task shell (covers all task types)

Task files are numbered with zero-padded two-digit prefixes in topological order: Wave 1 first, then Wave 2, etc.

### Step 7 — Write self-contained task files

Each task file is the **only thing** a coder agent will read. It must contain everything that agent needs. Sections required:

- **Description**: what to build and why in context
- **Dependency context**: in prose, what prior tasks produce that this task needs (filenames, schemas, signatures). The agent must not need to read other task files.
- **Stack profile**: Livewire / Vue / backend-only — tells the coder which UI skill to invoke
- **Files to create**: explicit paths
- **Files to modify**: explicit paths
- **Technical details** — Laravel-specific sub-sections as relevant:
  - **Migration** — column definitions, indexes, FKs, UUIDs vs auto-inc
  - **Model + Factory** — fillable/guarded, casts, relationships, factory states
  - **Authorisation** — Policy method names, Gate definitions, middleware
  - **UI** — for Livewire: component path, view path, Flux components to use, wire: directives; for Vue: page path, Inertia props, Headless UI primitives
  - **Routes** — route names, methods, middleware
  - **Pest test** — file path, what assertions are required (HTTP status, DB state, events fired, jobs dispatched, authorisation)
  - **Artisan commands** — exact `php artisan make:*` commands to scaffold files
- **Acceptance criteria**: specific, verifiable bullets (e.g. "GET /api/users/{id} returns 200 with JSON shape `{id, email}`", "Migration creates `users.uuid` column unique-indexed", "Pest test `it allows owner to delete invoice` passes")

Review each task with fresh eyes: could an agent who has never seen the planning conversation implement this correctly using only this file? If not, add what's missing.

### Step 8 — Extract manual actions

Identify steps that need human action (API key signup, DNS, env vars, third-party dashboards, Stripe webhook config, Cashier setup). Write to `action-required.md` grouped by **Before / During / After implementation**. If none, write "No manual steps required."

### Step 9 — Report

After creating the spec, display:

```
Feature specification created at specs/{feature-name}/

Files created:
  - README.md (dependency graph, {N} waves, {T} tasks)
  - requirements.md
  - action-required.md
  - tasks/ ({T} task files)

Wave breakdown:
  - Wave 1: {count} tasks (parallel) — {brief description}
  - Wave 2: {count} tasks (parallel) — {brief description}
  - ...

Next steps:
  1. Review action-required.md for tasks you need to complete manually
  2. Review requirements.md and the task files
  3. Run /implement-laravel-feature to start parallel implementation
```

## Critical rules

- Every task file must be fully self-contained. A coder agent reading only that file must know exactly what to do.
- Capture ALL technical details from the planning conversation. The spec is the single source of truth.
- Tasks within the same wave must not modify overlapping files.
- Keep tasks atomic — one concern per task. Split if more than 5–7 files.
- Do not create testing tasks; tests live inside the task that produces the code under test.
- For non-auth IDs prefer UUIDs (use `$table->uuid('id')->primary()` + `HasUuids` trait on the model). For Fortify/Auth user IDs follow the existing convention in the project.
- Number task files in topological order (wave 1 first) for easy scanning.
