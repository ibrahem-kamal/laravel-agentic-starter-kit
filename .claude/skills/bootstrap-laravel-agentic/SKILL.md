---
name: bootstrap-laravel-agentic
description: >
  Install the Laravel Agentic Starter Kit into a Laravel project. Detects whether the project
  uses Livewire/Flux or Vue/Inertia (or neither), prompts to install Laravel Boost, offers to
  install graphify if missing and registers its skill, copies the bundled skills into
  .claude/skills/, and writes CLAUDE.agentic.md and DESIGN.md without overwriting an existing
  CLAUDE.md. Use this skill the first time the
  bundle is installed in a project, or to upgrade an existing install. Trigger phrases: "install
  the agentic starter", "bootstrap the kit", "set up agentic coding here", "/bootstrap-laravel-agentic".
---

# Bootstrap Laravel Agentic Starter Kit

You are installing a portable Claude Code skill bundle into a Laravel project. The bundle is **pure agent tooling** — no PHP code is shipped. Your job is to detect the project's stack, wire Laravel Boost and graphify, copy the bundled skills into `.claude/skills/`, and write the rules and design files without clobbering anything the user has set up.

## Critical rules

- **Never overwrite an existing `CLAUDE.md`.** Write `CLAUDE.agentic.md` instead and tell the user to add `@CLAUDE.agentic.md` to their own CLAUDE.md.
- **Never overwrite an existing `DESIGN.md`** unless the user explicitly asks to replace it.
- **Never overwrite an existing skill folder** inside `.claude/skills/` unless the user passes `--force` (verbally or in the trigger message). Skip and report which ones were skipped.
- **Always prompt before installing Laravel Boost.** Boost is a dev dependency that adds MCP tools and modifies composer.json — get explicit consent.
- **Always check for graphify** and offer to install it (`uv tool install graphifyy` / pipx / pip) if it's not on PATH. Don't try to build the graph yourself — graph construction is an explicit user action via `/graphify .`. Do not abort the bootstrap if the user declines graphify or the install fails.
- Do not edit any Laravel application files (models, controllers, migrations, routes, blade, vue, livewire components, env files). The bundle is agent tooling only.

## Steps

### Step 1 — Verify Laravel project

Read `composer.json` from the current working directory.

- If the file does not exist: ask the user "No `composer.json` found in `{cwd}`. Scaffold a fresh Laravel project here?". If yes, run `composer create-project laravel/laravel . --no-interaction` and re-read `composer.json`. If no, abort with a clear message.
- If the file exists but `laravel/framework` is not in `require`: ask whether to proceed anyway (e.g. the user might be in a monorepo subfolder). Default to abort.

### Step 2 — Detect frontend profile

Read `composer.json` and `package.json` (if present). Determine the profile:

- `livewire/livewire` in composer.json → **Livewire profile**
- `inertiajs/inertia-laravel` in composer.json → **Vue/Inertia profile**
- Both present → ask the user which is primary (treat the other as escape hatch)
- Neither present → use AskUserQuestion with two options:
  1. **Livewire 4 + Flux** — run `composer require livewire/livewire livewire/flux`
  2. **Vue 3 + Inertia** — run `composer require inertiajs/inertia-laravel` then `npm install @inertiajs/vue3 vue @vitejs/plugin-vue`
  3. **Skip — agent tooling only, no frontend dependency** — proceed without installing either

Record the chosen profile for the report at Step 8. Write it to `.claude/agentic-profile` (single line: `livewire`, `vue`, or `none`) so other skills can read it later.

### Step 3 — Prompt to install Laravel Boost

Show the user this exact prompt via AskUserQuestion:

> **Install Laravel Boost?**
> Boost adds MCP tools (`database-query`, `search-docs`, `browser-logs`, `tinker`, etc.) that the bundled skills rely on. Highly recommended.
> Docs: <https://laravel.com/docs/13.x/boost>

Options: **Yes, install now** / **Skip — I'll install later**.

If yes: run `composer require laravel/boost --dev` then `php artisan boost:install --no-interaction`. Report any errors but continue the bootstrap.

### Step 4 — Install and register graphify

graphify is a knowledge-graph layer that this kit's other skills assume is available. Always make sure it's wired up.

1. If `graphify-out/` already exists in the project, skip — graphify is already initialised here.
2. Otherwise check `command -v graphify`:
   - **If present**: ensure the graphify skill is registered with Claude Code by running `graphify install --platform claude` (idempotent — safe to re-run). Then tell the user: "graphify is installed. Run `/graphify .` once to build the initial knowledge graph for this project." Do **not** try to build the graph yourself from the bootstrap — graph building can take minutes on a large project and the user should kick it off explicitly.
   - **If absent**: offer to install it. Use AskUserQuestion:

     > **Install graphify?**
     > graphify turns this project into a queryable knowledge graph. The kit's skills query the graph instead of grepping raw files for many tasks. Highly recommended.
     > Package: `graphifyy` on PyPI · Docs: <https://pypi.org/project/graphifyy/>

     Options: **Yes, install via uv** / **Yes, install via pipx** / **Yes, install via pip** / **Skip — I'll install later**.

     Install commands (pick by user choice):
     - `uv tool install graphifyy` (recommended — fastest)
     - `pipx install graphifyy`
     - `pip install graphifyy`

     After the install command, run `graphify install --platform claude` to register the graphify skill, then tell the user to run `/graphify .` to build the initial graph.

     If the user picks "Skip", print this line and continue (do not abort): `graphify not installed. Install later with 'uv tool install graphifyy && graphify install --platform claude', then run '/graphify .' to build the project's knowledge graph.`

If any install command fails (uv/pipx/pip not on PATH, network error), surface the error and continue the bootstrap — graphify is recommended, not required.

### Step 5 — Copy bundled skills

The bundle ships these skills in its own `.claude/skills/`:

- `create-laravel-spec`
- `implement-laravel-feature`
- `prism-ai`
- `ship-laravel`
- `checkpoint`

(Do **not** copy `bootstrap-laravel-agentic` itself — it's the installer.)

For each skill:
1. Check if `<target>/.claude/skills/<skill>/` already exists.
2. If it exists and `--force` was not given: skip and add to the "Skipped" list in the final report.
3. Otherwise: copy the entire skill folder (including `references/`) recursively.

If you are running via `npx skills add`, the bundle files are available at the path skills.sh extracts to (usually a temp dir whose path is passed to the skill). If running manually (e.g. cloned repo), the bundle root is the directory containing this SKILL.md's grandparent (`../../../`).

Use the Bash tool with `cp -R` for the copy. Do not use Write per-file — preserves any references/ subfolders automatically.

### Step 6 — Write CLAUDE.agentic.md

The bundle ships `CLAUDE.agentic.md` at its root. Copy it to `<target>/CLAUDE.agentic.md`.

If `<target>/CLAUDE.agentic.md` already exists: ask the user "An existing CLAUDE.agentic.md is present. Overwrite with the latest version from the bundle?". Default to no.

**Never touch an existing `CLAUDE.md`.** After writing, print the line the user needs to add to their CLAUDE.md:

```
@CLAUDE.agentic.md
```

If no `CLAUDE.md` exists in the project, create a minimal one containing only that import line.

### Step 7 — Write DESIGN.md

The bundle ships `DESIGN.md` at its root. Copy to `<target>/DESIGN.md` only if no DESIGN.md exists. If it exists, skip and add to the "Skipped" list.

### Step 8 — Final report

Print a structured report. Use this exact template, filling in the bracketed values:

```
Laravel Agentic Starter Kit — bootstrap complete.

Project:        {project-name from composer.json}
Frontend:       {Livewire | Vue/Inertia | None}
Laravel Boost:  {installed now | already present | skipped}
graphify:       {already present — skill registered | installed via {uv|pipx|pip} now | skipped — install hint printed}

Skills installed:
  - create-laravel-spec
  - implement-laravel-feature
  - prism-ai
  - ship-laravel
  - checkpoint

Skills skipped (already present, --force to overwrite):
  {list or "none"}

Files written:
  - CLAUDE.agentic.md
  - DESIGN.md

Next steps:
  1. Add this line to your project's CLAUDE.md (created automatically if it didn't exist):

         @CLAUDE.agentic.md

  2. {if graphify was installed or already present, but graphify-out/ is missing}
     Run /graphify . to build the project's knowledge graph (one-time, ~minutes).

  3. Run /create-laravel-spec to plan your first feature.
```

## When the user has overrides

If the user's trigger message includes flags or preferences (e.g. "install but skip Boost", "force overwrite skills", "use Vue", "don't run graphify"), honour them at the relevant step and skip the corresponding prompt. Echo back what you'll do before starting so the user can correct you.
